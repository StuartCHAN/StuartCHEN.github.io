---
layout:     post
title:      Think About The NSpM
subtitle:   This is a report about the training of the NSpM model.
date:       2019-05-17
author:     Stuart Chen
header-img: img/post-bg-debug.jpg
catalog: true
tags:
    - GSoC2019
    - NLP
    - DBpedia
---


# Introduction

The NSpM model, Neural SPARQL Machines, are a type of LSTM-based Machine Translation Approaches for Question Answering based on external knowledge base in DBpedia and via SPARQL query.

I feel touched by this model mostly because I have always the passion for the neural memory structure, the memorization mechanism and the reasoning processing, with which I saw a lucid hint from the NSpM.

Now, let's talk about it.

Here is the code：
https://github.com/AKSW/NSpM

![alt text](http://www.liberai.org/img/seq2seq-webexport-160px.png "Neural SPARQL Machines")
![](http://www.liberai.org/img/flag-sparql-160px.png)

# The Previous Training

I am an art fan, so I prefered the data with the topic 'LC_QuAD_v6_art'. My lucky number is 608, then I randomly picked 608 temple from the file, and generated 98458 lines in the training data file. 
After running
```bash
sh train.sh data/LC_QuAD_v6_art 120000
```
different cases were found:

## Trial No.1
At first, I choosed a laptop with Ubuntu 16, TensorFlow and Python 2.7 to do the computation, without GPU. It turned out that the training seemed abnormal while there were many conflicts in the codes, which kept printing that the modiles had something unmatched. Then I borrowed some functons with a same arithmetical meaning to replace the bugs. But the results were not so reasonable as expected, and my laptop's Linux OS just collapesd.

## Trial No.2
So it went to the next. That time I figured out that it might be due to the version of TensorFlow had not been satisfied. It had been deployed on a higher version than 1.3.0. So this time I was more fucuesd on the selection of the TensorFlow version. And considering about the hardware, the second trial was experimented on an Cloud ECS computing platform of [Alibaba Cloud](https://cn.aliyun.com/?accounttraceid=a3b99d73-db56-4cd2-ae2f-aa707c1e0a9e). But there were still glitches. When I open the data holding folder, I found that the corresponding pars of natural language setences and the queries, repectively from the .en and .sparql files, were not well matched like the provided pretrained sample. And the final inferences did not feel so logic to the questions. It was strange-looking. 

## Conclusion
We must be cautious about the following points:\
1, the different versions of TensorFlow can lead to the gap between the functions in the module:

```bash
tf.contrib
```
may be not existing any more in higher versions, which leads to the missing of its sub-modules.

2, If it is run on the online computing cloud, must pay attention to the internet connection interface of virtual server, for it could be unable to use the SPARQL or any relevant functions to the DBpedia. 




# Recent Experiments

Recently, the experiment have been into two trials:

## Experiment 1 
The experiment was still using the previously mentioned data from the topic 'LC_QuAD_v6_art'. This trial was on a Lenovo computer without GPU, which had TensorFlow 1.3.0, Python 2.7, and tensorboard. The procedure was as the official guidance above. I checked into the files, where every thing seemed okay, and the inference after the training looked sensible.

However, I did not notice the absence of the BLEU records. After rethinking about the experiment, I hypothesized that the early stop policy should be blamed, because I had set the stop, at about 12 00, less than the steps officially proposed.

Another suspicion about the training is that, the originally provideded template contains a long number value at the last column which seems unexplainable.


## Experiment 2

Ok, here is my fatal fight with this model.

Firstly, it must have something to say with the template selection. I have konwn that any even subtle edition of the file can lead to a huge divergence. So, this time, I just try to be a good boy and use the very official provided template, checking all the required details to reproduce the experiment.

Secondly, I admited that, at first, I had thought about(and physically, I really did the job) rewriting the cods, incuding overriding the lost function and the evaluation functions. But, anyway, I rewrote them back and do the suggested traditional way.

Finally, I put it on a high performance GPU [online cloud](https://www.jikecloud.net/list.html), sacrificing all the wages of my last week's part-times to pay for the cloud device. I was excited facing the battle with the model.



And, nice, look at what it brought us:
* [Experiment 1 notes](https://docs.google.com/document/d/1S49o0qlKtHYMHDekGryPbUO2CcVXHQeZ1m72Zdm4yVw/edit?usp=sharing)
* [Experiment 2 notes](https://docs.google.com/document/d/1fkbylG4wK9waybCMiM8MKtrUFCUaartHKhShajt2UD0/edit?usp=sharing)

## Experiments Results

I deployed the 'LC_QuAD_v6_art' dataset and the 'monument_300' dataset as parallel comparative groups to conduct the experiment. Here the evaluation of the accuracy of the neural model was based on the BLEU test, which is a modified precision metric for testing the machine translation output against the reference translation standard.

 | Dataset | Examples per template |Training size |Average examples per instance|
 | ------ | ------ | ------ |------ |
 | dbo:LC_QuAD_v6_art | 608 | 1200 |161.94 |
 | dbo:Monument  | 300 | 12000 |13.35 * |

 * *Because of the pretraining of the dbo:Monument, the figure shares a great similarity this [report](https://arxiv.org/html/1708.07624).*

The experimentation for dbo:Monument was conducted on a virtual cloud server with the GPU of GTX1080, 480GB SSD, and E5-2690 v3 24-core CPU of 64G. The experimentation for dbo:LC_QuAD_v6_art was conducted on another virtual cloud server with the GTX1080, 2TB SSD, and E5-2658 v3 48-core CPU of 64G, and . The basical setting for both comparative groups were Ubuntu 16 system with TensorFlow 1.3.0. The testing for the NMT model at was mainly displayed at three different times, i.e. at the 1,200th, 3,600th, and 12,000th iteration.

| Dataset | BLEU 3k steps|BLEU 9k steps|BLEU 12k steps|
 | ------ | ------ |------ |------ |
 | dbo:LC_QuAD_v6_art | 0 |0 |0 |
 | dbo:Monument  | 67.0 (May 16 14:23:21 2019) |79.6 (May 16 15:55:34 2019) | 80.2 (May 16 17:00:32 2019)|

 

## Final Analyses

In the different parts of the experiments, we can see the huge gap between the training results, which tells a lot of information.
* The model may have a strong dependency on the generated data, which requires the high quality of the source template.
* And, before all the training, make sure the environment is fit for every details of the prerequisites, e.g, the version of packages and the dependencies of them.

# References
[1] Tommaso Soru, Edgard Marx, André Valdestilhas, Diego Esteves, Diego Moussallem, Gustavo Publio. (2018). Neural Machine Translation for Query Construction and Composition. 

[2] Papineni K., Roukos S., Ward T., Zhu W. (2002). BLEU: a method for automatic evaluation of machine translation. Proceedings of the 40th annual meeting on association for computational linguistics.