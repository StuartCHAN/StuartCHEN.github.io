---
layout:     post
title:      Let-us-make-the-templates
subtitle:   This is a report about the training of the NSpM model.
date:       2019-05-26
author:     Stuart Chen
header-img: img/ivana-cajina-343416-unsplash.jpg
catalog: true
tags:
    - GSoC2019
    - NLP
    - DBpedia
---


# Introduction

What's the template? If you ask me to describe it, I will imagine it as the bone of the question structure. If we have a question, like

        What is the elephant lifesapn?

if we mark the word "elephant" as a variable placeholder "Species", and mak the lifespan as the property "ageRange", then what can we get?

        dbo:Species;;;what is <A> lifespan;select ?a where { <A> dbo:ageRange ?a };select distinct(?a) where { ?a dbo:ageRange [] }

Yeah, here is the immitative syntactical silhouette of a question or sentence.

![alt text](https://cdn-images-1.medium.com/max/1200/1*QMgA0UViZMW7LUjRhil9yQ.jpeg "DBpedia RDF")
![](https://images2.minutemediacdn.com/image/upload/c_fill,g_auto,h_1248,w_2220/f_auto,q_auto,w_1100/v1555924214/shape/mentalfloss/451244723_0.png)

# The templates are so important!

 It is the base that we get our training data in the NSpM model.

## OK, we go to the experiment now.

The templates about the [ontology of species](http://mappings.dbpedia.org/server/ontology/classes/Species) in DBpedia can be found [here](https://docs.google.com/spreadsheets/d/1o7mpc7TuJOBnMb4CmtC2FE1wsVrYUVMQR7fy0EOSSqo/edit?usp=sharing), and these are the [generated training data](https://drive.google.com/drive/folders/1J7olhKwObf4yMVaiO2vATixI2QnuZVY1?usp=sharing).


## Experiments Records

I used the Species data that had been generated in previous step, and get the records like this:

 | Dataset | Examples per template |Training size |Average examples per instance|
 | ------ | ------ | ------ |------ |
 | dbo:Species  | 39 | 12000 | 307.69|

 
The experimentation for dbo:Species was also conducted on a virtual cloud server with the GPU of GTX1080, 480GB SSD, and E5-2690 v3 24-core CPU of 64G, with basical setting for both comparative groups were Ubuntu 16 system with TensorFlow 1.3.0. The testing for the NMT model at was mainly displayed at three different times, i.e. at the 2,000th, 1,000th, and 12,000th iteration.

| Dataset | BLEU 2k steps|BLEU 10k steps|BLEU 12k steps|
 | ------ | ------ |------ |------ |
 | dbo:Species  | 60.2 (May 25 07:24:53 2019) |84.3 (May 25 08:39:16 2019) | 89.2 (May 25 08:54:56 2019)|

 

## Analyses

However, I saught a problem, you see the dev bleu and the test bleu don't match 
![alt text](https://dbpedia.slack.com/files/UJA85N9G9/FJNK43M3M/image.png "a screenshot when the 2,000th step")
What's the problem?
The training has been interrupted a few times when the previous training steps displays the bleu dev is approaching 60 while the external bleu test records it still as 0. I have restarted the training several times, but it goes on to be like that. Maybe it was because the number of global steps was still below 2k steps? Here will show a piece of the record:

 

    External evaluation, global step 1000
    decoding to output /data/DBPEDIA/neural-qa/data/annotations_Species/output/output_dev.
    2019-05-25 07:20:16.906704: W tensorflow/core/framework/op_kernel.cc:1192] Out of range: End of sequence
            [[Node: IteratorGetNext = IteratorGetNext[output_shapes=[[?,?], [?]], output_types=[DT_INT32, DT_INT32], _device="/job:localhost/replica:0/task:0/cpu:0"](Iterator)]]
    2019-05-25 07:20:16.906708: W tensorflow/core/framework/op_kernel.cc:1192] Out of range: End of sequence
            [[Node: IteratorGetNext = IteratorGetNext[output_shapes=[[?,?], [?]], output_types=[DT_INT32, DT_INT32], _device="/job:localhost/replica:0/task:0/cpu:0"](Iterator)]]
    done, num sentences 1200, time 1s, Sat May 25 07:20:16 2019.
    bleu dev: 59.3
    saving hparams to /data/DBPEDIA/neural-qa/data/annotations_Species/output/hparams
    External evaluation, global step 1000
    decoding to output /data/DBPEDIA/neural-qa/data/annotations_Species/output/output_test.
    2019-05-25 07:20:18.993003: W tensorflow/core/framework/op_kernel.cc:1192] Out of range: End of sequence
            [[Node: IteratorGetNext = IteratorGetNext[output_shapes=[[?,?], [?]], output_types=[DT_INT32, DT_INT32], _device="/job:localhost/replica:0/task:0/cpu:0"](Iterator)]]
    2019-05-25 07:20:18.993082: W tensorflow/core/framework/op_kernel.cc:1192] Out of range: End of sequence
            [[Node: IteratorGetNext = IteratorGetNext[output_shapes=[[?,?], [?]], output_types=[DT_INT32, DT_INT32], _device="/job:localhost/replica:0/task:0/cpu:0"](Iterator)]]
    done, num sentences 0, time 0s, Sat May 25 07:20:18 2019.
    bleu test: 0.0
    saving hparams to /data/DBPEDIA/neural-qa/data/annotations_Species/output/hparams

 

## How to fix the ZeroDivisionError?
We found that the log shows 0 sentences in test set, as a result this turns out to be zero.
So, we should change the argument values while spliting the data_.* files into train_., dev_., and test_.* .
It should trace back to the [NSpM/split_in_train_dev_test.py](https://github.com/AKSW/NSpM/blob/master/split_in_train_dev_test.py) file, and I think it was due to percentage that it set:

        TRAINING_PERCENTAGE = 90
        TEST_PERCENTAGE = 0
        DEV_PERCENTAGE = 10

Here we can see the developper tried to build the datasets based on the 10-fold cross-validation method. However, it is neglect about he situation where the number of total training templates might not be numerous enough.

How to better the spliting?

If the test set contains zero sentence, it is possible to try selecting 10 percent from the training data and 10 percent from the testing data to manually build one after generating and a check. The the ratio would be

        TRAINING_PERCENTAGE : TEST_PERCENTAGE : DEV_PERCENTAGE = 90 : 10 : 10


with

        TEST _fromTRAINING : TEST_fromDEV = 9 : 1
        TEST _fromTRAINING + TEST_fromDEV = TEST_PERCENTAGE

**We can see the training results [here](https://drive.google.com/drive/folders/1f2cs0Pz4-OmXUQ0nkr3RnOpBi7oE4NWB?usp=sharing).
