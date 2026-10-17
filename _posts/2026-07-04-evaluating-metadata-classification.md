---
layout: post
title: Evaluating metadata classification for a retrieval pipeline
date: 2026-07-04 10:00:00 +0000
description: "Retrieval was tuned, so I measured the next stage: how reliably a local model labels each document. Across input windows, more than half the document types stayed wrong, and the chunking setting turned out to be quietly controlling what the classifier could see."
image: /assets/images/classification/eval-loop.png
---


In the previous post, I evaluated how FindFiles splits documents into chunks before retrieving them. The next question was how reliably it identifies those documents. A retrieved passage can mention wages, withholding, and a Social Security number without telling the system whether the file is a W-2, a W-4, or instructions for filling one out. For a task-based checklist, that distinction matters.

FindFiles classifies each PDF locally with `llama3.2` through Ollama. It asks for the document type, date, issuer, subject, a short summary, and a confidence score, then stores that metadata alongside every chunk from the file. Document text stays on the machine. The separate task-reasoning call receives the user's task description.

The metadata gives the retrieval pipeline another way to judge a candidate: does this file have the expected document type, and does its year match the task? That only helps if the metadata is reliable. I built a separate evaluation to measure it. On the baseline, the model identified 20 of 53 documents correctly after label normalization. Its average reported confidence was about 88%.

## Metadata becomes part of the decision

Classification happens once during indexing, and its output is copied to every chunk from the document. A wrong label therefore follows the file through retrieval. Even if a chunk contains the right evidence, the scoring step can penalize it because the stored document type disagrees with the checklist.

That makes classification worth measuring separately. An end-to-end result can tell me that a document was missed, but it leaves several possible causes: extraction lost the text, retrieval failed to find it, or metadata pushed a useful candidate down the ranking. A field-level evaluation lets me inspect one of those stages directly.

I focused on three fields: document type, year, and issuer. The classifier also produces a subject and summary, but I didn't have suitable ground truth for scoring those. A fluent summary wouldn't establish that the other fields were correct anyway.

## Evaluation setup

I used the same 53 hand-labeled public US government PDFs as the chunking evaluation. The manifest records the expected document type, date, and issuer independently of the classifier's predictions.

For each configuration, the harness extracts the PDF text, selects the text to send to the model, records the raw response, and scores the fields against the manifest. It preserves the raw document-type label as well as the version produced by the application's normalization rules. Keeping both turned out to matter.

I also recorded input length, latency, missing answers, and failed calls. Before a run, the harness checks that Ollama can actually perform an inference call. The application can fall back to empty metadata when classification fails, but an unavailable model shouldn't quietly become a benchmark showing 0% accuracy.

The results below are from one pass per configuration over the 53 documents. There were no failed calls in these runs. They are exploratory measurements; repeated runs and a separate evaluation set are still needed before treating small differences as settled.

## What counts as a correct label?

The first complication was deciding how to compare answers. `Form W-2` and `W-2` should count as the same label. But `1099-INT` and `1099-DIV` identify different forms, even though the application currently reduces both to `1099`.

I measured document-type accuracy two ways:

- **Strict accuracy:** ignore formatting differences such as capitalization, whitespace, and a leading "Form," while preserving subtype distinctions.
- **Canonical accuracy:** apply the application's label normalization to both the prediction and the expected answer before comparing them.

On the baseline, strict accuracy was 24.5%, or 13 of 53 documents. Canonical accuracy was 37.7%, or 20 of 53.

The model had produced the same answers in both cases. The seven additional matches came from what the scorer accepted. Some normalization handles useful synonyms; some removes distinctions the task may still need. Reporting only the canonical score would hide that difference.

Issuer names have a similar issue. `IRS` and `Internal Revenue Service` refer to the same organization, so I scored issuer names both directly and with an explicit alias mapping. Baseline issuer accuracy went from 22.6% to 52.8% when aliases were accepted. That is a substantial difference in how the answers are represented, and another reason one overall "metadata accuracy" number would be hard to interpret.

## The model wasn't receiving the text I thought it was

The classifier's prompt said "first 3,000 characters," and the function sliced its input to that length. But the indexing code passed it only the first retrieval chunk:

```python
classification = classify_document(texts[0], file_path)
```

In the evaluated baseline, the splitter used a 1,000-character budget. The actual classifier input averaged about 899 characters. Taking the first 3,000 characters inside the classifier couldn't recover text the caller had already left out.

This connected the classification experiment back to chunking in a way I hadn't accounted for. Retrieval chunk size was also controlling how much of the document the classifier could see. Changing it for better search would change classification input too.

I tested that input window directly: the first split chunk, the first 1,000 characters of extracted text, the first 3,000, the first 6,000, and a longer input capped at 24,000 characters. The model, prompt template, schema, and sampling settings stayed fixed. The prompt template retained its original 3,000-character wording, so that wording was inaccurate for some experimental inputs as well.

| Classifier input | Strict accuracy | Canonical accuracy | Mean latency |
|---|---:|---:|---:|
| First retrieval chunk | 24.5% | 37.7% | 3.5 s |
| First 1,000 characters | 26.4% | 41.5% | 4.1 s |
| First 3,000 characters | 26.4% | 45.3% | 6.2 s |
| First 6,000 characters | 30.2% | 35.8% | 7.8 s |
| Extracted text, capped at 24,000 characters | 24.5% | 30.2% | 15.2 s |

The 3,000-character window had the highest canonical accuracy: 24 correct documents, four more than the baseline. Its strict score improved by only one document. The 6,000-character window had the highest strict score. There wasn't a single winner across both definitions of correctness.

The longest input was slower and had lower accuracy on both measures than the 3,000-character window. It also returned no document-type label for 11 of the 53 files, compared with one in the baseline.

More text didn't consistently help in this experiment. These forms can contain instructions and references to other forms, so extra text may introduce competing clues. That's a possible explanation, not something this sweep isolates. The measured result is narrower: increasing the input budget alone did not make this classifier reliable.

## Confidence didn't separate the mistakes

The classifier reports its own confidence in the document-type label. That seems useful for deciding when to trust metadata, so I checked whether the confident predictions were actually more dependable.

In the baseline, average confidence was 0.879. Of the 52 predictions at or above a 0.6 confidence threshold, only 20 were canonically correct, 38.5% accuracy. Almost every prediction passed the threshold, including most of the mistakes.

A confidence gate at that level therefore did little to protect the pipeline. The number was another model output that needed evaluation before it could be used as evidence of correctness.

## Dates evaluation

Document year was weaker still. The baseline matched the labeled year for 10 of 53 documents and left the date empty for 28. Of the 16 it did return a full date for, nine had the wrong year an answer can look precise and still fail the coarser check. My labels are year-level, so I didn't score months and days at all.

Before generalizing any of this, I need to decide which date I'm asking for. A blank public form carries a revision date, a filled-in statement carries a tax year or a period that matters to the task. The prompt currently treats every visible year as interchangeable.

## What I would change next

Classification should build its own `text_to_classify` from the extracted text instead of inheriting the first retrieval chunk, which means changing both the caller and the 3,000-character cap inside the classifier. The first 3,000 characters are worth another run, but 45.3% canonical accuracy isn't dependable enough to build on.

Before touching the prompt or the model, I'd repeat the promising configurations and read the individual errors. I also need a corpus of completed forms, statements, and scans; 53 blank public forms are good for finding problems, not for establishing performance.

And this only measures the fields. Whether better metadata produces a better found / uncertain / missing checklist is a separate comparison, with retrieval and scoring held fixed.

The chunking experiment gave me a better retrieval configuration. This one showed me that the same setting was also deciding how much of a document the classifier could see, and that widening it still left more than half the document types wrong. Before giving those labels more influence over a search result, I need better evidence that they deserve it.
