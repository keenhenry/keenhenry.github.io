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
2. decoder-only models => tasks requiring uni-directional context (from left to right), like generating tasks according to texts already seen.
3. encoder-decoder models => good at sequence-to-sequence tasks such as translation, summarization and question answering