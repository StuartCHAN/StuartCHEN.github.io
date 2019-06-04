---
layout:     post
title:      Architecture
subtitle:   a glimpse of the silhouette of the architecture
date:       2019-06-02
author:     Stuart Chen
header-img: img/apple-blur-branch-257840.jpg
typo a-root-url: ..
catalog: true
tags:
    - GSoC2019
    - NLP
    - DBpedia
---

# The Pipeline of Architecture

The model architecture is like:
![architecture and pipeline](img/Atten_NSPM00.png)

# Pipeline 

## 1. Question Generation from Natural Language
We will use attention mechanism to generate questions from natural language contextual passage. By detecting entities in the passage, it will automatically get questions about the entities.

## 2. From Natural Language Questions to Templates
With the help of semantic parsing, the model extracts the templates from the questions.

## 3. Construction of the Templates Bank
The Templates Bank is a base where stores the existed templates. 

## 4. Comparison of Newly Generated Templates and the Model Matching
The templates generated in the previous step shall be compared against the tempalets in the Templates Bank as to match the similar one. Then, if matched, it will refer to the similar trained model that contains the matched template; if not matched, it turns to train the model on the new templates, and store the new results into the Templates Bank.