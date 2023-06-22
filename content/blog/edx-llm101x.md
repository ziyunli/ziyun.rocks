---
title: "Notes on Databricks LLM101x"
summary: "Large Language Models: Application through Production"
date: 2023-06-13
tags: ["notes", "course", "llm"]
draft: true
---

I signed up [Large Language Models: Application through Production](https://learning.edx.org/course/course-v1:Databricks+LLM101x+2T2023/home) from Databricks on edX to learn more about applications of large language models. This page contains my notes on the course.

<!--more-->

# 0. Introduction

This module goes through some key concepts and terminology.

Language models: probabilistic models that assign probabilities to word sequences.
Large: 10~50M to many billions of parameters. Made possible by transformer architecture since ~2017.

Token: basic building block of language models. Word, subword, character, etc.
Sentence: sequence of tokens.
Vocabulary: complete list of tokens.

Tokenization:
1. Words:
   1. Intuitive
   2. Big vocabulary
   3. Complications such as misspelling, out-of-vocabulary (OOV) words, etc.
2. Characters:
   1. Small vocabulary
   2. No OOV
   3. Long sequences
   4. No word-level semantics
3. Subwords:
  1. popular: byte pair encoding (BPE)

Word embeddings:
1. By frequency -> sparsity issue
2. word/token -> embedding function -> word embedding/vector

# 1. Application

This module is in fact an introduction of huggingface transformers. The code examples are very straightforward to understand if you already know Python.
In fact the whole lab takes around 10 minutes to complete if it's not waiting to download all the data sets and models along the way.

On the high level, a HF pipeline could have these steps:
input -> prompt constructions -> tokenizer (encoding) -> model -> tokenizer (decoding) -> output

Some parameters to tweak:

tokenizer:
- `max_length`: max length of input sequence

model:
- `do_sample`: whether to use sampling
  - `top_k`: top k tokens to sample from
  - `top_p`: cumulative probability of top tokens to sample from
  - `termperture`: temperature of sampling
- `num_beams`: number of beams for beam search
- `max_length`: max length of output sequence
- `min_length`: min length of output sequence
