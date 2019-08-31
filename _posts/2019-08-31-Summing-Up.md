---
layout:     post
title:      Summing-Up
subtitle:   Outline & Evaluation 
date:       2019-08-31
author:     Stuart Chen
header-img: img/post-bg-desk.jpg
typo a-root-url: ..
catalog: true
tags:
    - GSoC2019
    - NLP
    - DBpedia
---

# Summing Up
-------------------------------------------------
<br>

## Project Outline

### Week 11&12
* Wiki Extraction & Sentences Filtering


### Week 9&10
* Univerasal Sentence Encoder for Template Matching & Templates Bank

### Week 7&8
* Analyses for Previous Works


### Week 6
* Vector Similarity Calculation


### Week 5
* Issues Analyses 
* Vectors Embedding


### Week 4
* Pre-evaluation Preparation 
* Exploring Potential Issues for Current NSpM


### Week 3
* Improving the Generation Component 
* Thinking About The Evaluations


### Week 2

* Entity Recognition 
* Template Generation


### Week 1

* qald evaluation against GERBIL interface building the semantic tree structue for the natural lang.

<br>

## Evaluation of the Project

In the initial period, we wanted to use DBpedia embedding to do the SQuAD machine reading comprehension tasks with reinforcement learning, but gradually we realize the performance of the neural SPARQL machine is highly dependent on the training data which indicate the crucial necessity of automating the templates generation from long contextual passages. The Wikipedia is a wonderful source of plenty of such articles relevant to DBpedia RDF triples, so we decided to evolve an intelligent neural SPARQL machine with automated templates generation, comparison, and accumulation to try to approach a never-ending-learning intelligent agent. 

Of course, during the coding, we have countered so many difficulties, like doing the benchmark evaluations and some tough impediments, but as now I think about these problems, I think they gave me a totally thorough growth. I got to learn more and more about the newest products in the industry and get more adequate with the international coding standards which open my door to a bigger world. For example, in the part of calculating the vector similarity to match existing templates, we first used word mover distance with GLoVe vectors via gensim, but we found that was too heavy and too slow, then we used spaCy and found it much speedier. And soon after this, we found the Universal Sentence Encoder is even better in this task, which is a huge evolution in our development. 

Another thing that I still remember is the paraphrasing of the predicates, we used to think load all the phrases in RAM and do the matching. I still remember that file was so huge even more than 17.6 GB. Then I found the wordnet from nltk can accomplish this paraphrasing task without such a huge cost, which is a smart solution.

### Future Works

We hope to keep on the work on making the question generation even better and including ASK queries, queries that require filter (how many, how much, etc.) and complex queries as well. Because we believe this can make the neural SPARQL machines get even better and better performance.

* To read more, please refer to the [repository summary](https://github.com/StuartCHAN/neural-qa#summary).

<br>
