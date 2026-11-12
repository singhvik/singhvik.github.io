---
layout: post
title: "What Is an LLM? A Plain-English Explanation"
date: 2026-11-12
tags: [ai, llm, basics]
---

You've probably used one by now. ChatGPT, Claude, Gemini and Copilot are all built on the same idea: the **LLM**. But what is it, and why does it seem so smart?

## The short answer

LLM stands for **Large Language Model**. It's a computer program trained on an enormous amount of text so that it can understand and produce human language.

Let's unpack the three words.

- **Large**: it learned from a huge amount of text (books, websites, articles, code) and has billions of internal settings, called parameters, that store what it learned.
- **Language**: it works with words and text. Newer models can handle images and audio too, but language is the core.
- **Model**: it's a mathematical system, not a person or a database of answers. It captures patterns.

## What does it actually do?

At its heart, an LLM does one thing: **it predicts the next word.**

Think of your phone's autocomplete. If you type "See you at", it might suggest "the". An LLM does the same thing, except it has read a huge part of the internet and can look at everything you've written so far, not just the last word.

Ask it "What is the capital of France?" and it predicts that the most likely continuation is "The capital of France is Paris." It adds one word at a time, each one based on everything before it. That's why replies appear word by word on your screen.

It sounds too simple to produce essays, code and poems, but to predict text well across all those subjects, the model has to pick up grammar, facts, reasoning patterns and writing styles along the way.

## How does it learn?

Training happens in roughly two stages:

1. **Pre-training**: the model reads massive amounts of text and practices predicting the next word, over and over, adjusting its internal settings each time it's wrong. This is where it learns language and general knowledge.
2. **Fine-tuning**: the model is then trained further to be helpful, safe and good at following instructions. This uses example conversations and feedback on its answers, from people and increasingly from other AI systems. This is what turns a raw text predictor into something that feels like an assistant.

## One thing to keep in mind

An LLM generates answers from patterns it learned; it doesn't look facts up in a database. That means it can be wrong while sounding completely confident, which is called a *hallucination*. It's the most important thing to understand about these tools.
