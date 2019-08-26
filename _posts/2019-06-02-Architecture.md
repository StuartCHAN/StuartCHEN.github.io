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


![architecture and pipeline](https://res.cloudinary.com/stuarteec/image/upload/v1566790535/Atten_NSPM00.v02_y762nv.png)

# Pipeline 

## 1. Question Generation from Natural Language

![QuestionsGeneration](https://res.cloudinary.com/stuarteec/image/upload/v1566791144/Atten_NSPM00.v02_2_ep2wsi.png)

To generate the templates from the text extracted from Wikipedia, we need first to get the sentences containing RDFs.

### 1.1 Input from Natural Language

We use the natural language passage from Wikipedia as input :
```
    Audrey Hepburn was a British actress and humanitarian. Recognised as a movie star and fashion icon, Hepburn was active during Hollywood's Golden Age. She was ranked by the American Film Institute as the third-greatest female screen legend in Golden Age Hollywood, and was inducted into the International Best Dressed List Hall of Fame. She rose to star in Roman Holiday in 1953.
```
![](https://res.cloudinary.com/stuarteec/image/upload/v1563699161/v2-6a37b3b2f4db3949137d90642df08ff4_hd_uu57j6.png)

### 1.2 Pre-processing & RDF filtering with paraphrases

The passage goes through the `pre-processing` to remove the redundant characters and punctuations.

The cleaned passage will go into the part of neural coreferrences resolutions by spaCy and neuralcoref to confirm the subject's words of the topic, like: 

`Barack Obama`--> `he` or `him` ;

then, we use the wordnet via nltk to paraphrase the verbal forms of the predicate word, 

`dbo:parent` --lemmatize--> `parent` --paraphrase--> [ `father`, `mother`, ... ]

which can facilate the pinpointing of the RDF and filter out those sentences that can match the `< coreference(subject), paraphrase(predicate), object >` triple.

### 1.3 Output The Question

Then we get the output question, for example:

```bash
    " The <father> of <Barack> is <Obama Sr>. "
```

after which, we use the DBpedia-Spotlight API to detect the category of the entity, whether to classify it by using which interogative word to ask the question, e.g.

`Obama Sr` --spotlight--> `dbo:Person`

then we can pick the `who` from the list of interogative words [`who`, `what`, `where`, `which`] for questioning.

The output question will be converted to like:

```bash
    " who is the father of <dbr:Barack_Obama> ? "
```
which will be: `dbo:Person;;; who is the father of <A> ;` with the annotation of Spotlight.

## 2. Matching the new template question with existing templateset to see whether there's already a similar template for this question

![](https://res.cloudinary.com/stuarteec/image/upload/v1566791144/Atten_NSPM00.v02_4_f7jrnt.png)

With the help of the product of Universal Sentence Encoder by importing TensorFlow-hub, the model calculate the vector similarity between the existing templates and these new questions.

If the similarity score can pass the treshold, the system automatically fetch the matched item's template query to concatenate into the new question;

else if the treshold doesn't pass that there is no similar question for this new question, the system will also generate the query based on the annotation triple and the regex to buid a template query for it.


## 4. Transformer with Entities Annoatated

![Attention is all we need](https://res.cloudinary.com/stuarteec/image/upload/v1566816472/atten_figure1_mrubms.png "Attention Is All You Need .Figure 1")

For the prominent performance in neural machine translation task of Transformer with attention mechanism, it can be one state-of-the-art neural model to do the natural language to SPARQL task.

First, we have a look at the training data, which consist of two parts, namely, `data.en` the source data where there're the natural language questions with entities annoated, and `data.sparql` the target data where there're the correspondent SPARQL queries.

```text
In data.en:
    what is the total population of dbr_Barranca_de_Otates?
    who painted the dbr_Jekyll_+_Hyde?
    what is the total population of dbr_Lemithou?
    when did dbr_Haunting_of_Cassie_Palmer creator die?
    ...
```

```text
In data.sparql:
    select var_uri where brack_open dbr_Barranca_de_Otates dbo_populationTotal var_uri sep_dot brack_close
    select distinct var_uri where brack_open dbr_Jekyll_+_Hyde dbp_artist var_uri sep_dot brack_close
    select var_uri where brack_open dbr_Lemithou dbo_populationTotal var_uri sep_dot brack_close
    select distinct var_date where brack_open dbr_Haunting_of_Cassie_Palmer dbo_creator var_x sep_dot var_x dbo_deathDate var_date sep_dot brack_close
    ...
```

