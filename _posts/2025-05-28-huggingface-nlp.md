---
layout: post
title:  NLP Study Notes
date:   2025-05-28
categories: [technology, ai, nlp, llm, python , notes]
tags: [technology, artificial intelligence, natural language processing, python, notes]
description: Notes from learning Hugging Face NLP course
---

## Goals of the Course

This course is not focusing on how to build LLM or the theory behind it. It is only focusing on understanding LLM,
and in particular Transformer models and how to use them properly and its various applications.


## Important Notes

- Knowing which Transformer architecture is suited for what NLP task is key to choosing the right model.

1. encoder-only models => tasks requiring *bidirectional* context
2. decoder-only models => tasks requiring uni-directional context (from left to right), like text generation tasks according to texts already seen.
3. encoder-decoder models => good at sequence-to-sequence tasks such as translation, summarization and question answering

## Specific Models

### Text Classification

- Pretrained BERT
- Encoder-only architecture
- Objectives: <u>masked language modeling</u> and <u>next-sentence prediction</u>.


### Token Classification

- Also pretrained BERT
- But add a token classification head on top of the base BERT model.
- Also encoder-only architecture (because it's BERT!)
- Objectives: assigning a label to each token in a sequence, like Named Entity Recognition (NER)


### Question Answering

- Objectives: finding the answer to a question within a given context or passage.
- Model: BERT, but add a span classification head on top of the base BERT model.


### Summarization

- Objectives: condensing a longer text into a shorter version while preserving its key information and meaning.
- Architecture: encoder-decoder models. Encoder-decoder models like BART and T5 are designed for the sequence-to-sequence pattern of a summarization task
- Models: [BART][bart] or T5


### Translation

- Objectives: converting text from one language to another while preserving its meaning
- Architecture: encoder-decoder models. Translation is another example of a sequence-to-sequence task, which means you can use an encoder-decoder model like BART or T5 to do it.
- Models: BART, mBART (for multiple languages) or T5


[bart]: https://huggingface.co/docs/transformers/model_doc/bart
