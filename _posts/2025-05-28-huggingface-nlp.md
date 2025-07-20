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


## Models beyond text

Transformers are not limited to text. They can also be applied to other modalities like speech, audio, images, and video.

### Speech and Audio

For speech and audio data/tasks, a recent (2022) innovation regarding encoder-decoder structure called [**Whisper**][whisper] changed the effectiveness of the LLM models
in speech recognition related tasks, such as generating transcription from audio input.

Example use of code of Whisper is as follows:

```python
from transformers import pipeline

# this model is a fine-tuned model ready for use in speech recognition
transcriber = pipeline(
    task="automatic-speech-recognition", model="openai/whisper-base.en"
)
transcriber("https://huggingface.co/datasets/Narsil/asr_dummy/resolve/main/mlk.flac")
# Output: {'text': ' I have a dream that one day this nation will rise up and live out the true meaning of its creed.'}
```

### Computer Vision

Nowadays (2025), there are two main approaches for computer vision tasks:

1. Split an image into a *sequence of patches* and process them in parallel with a **Transformer**, see [ViT][vit].
2. Using [CNN][cnn] (Convolutional Neural Network), but a 'modern' one: [ConvNeXT][convnext].

ViT and ConvNeXT are commonly used for *image classification*.


[bart]: https://huggingface.co/docs/transformers/model_doc/bart
[whisper]: https://huggingface.co/papers/2212.04356
[vit]: https://huggingface.co/docs/transformers/model_doc/vit
[cnn]: https://cs50.harvard.edu/ai/notes/5/
[convnext]: https://huggingface.co/docs/transformers/model_doc/convnext
