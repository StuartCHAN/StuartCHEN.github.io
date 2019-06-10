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


![architecture and pipeline](https://pic3.zhimg.com/80/v2-a9408da7b2103f4018708efdc3e1f6be_hd.jpg)

# Pipeline 

## 1. Question Generation from Natural Language

![QuestionsGeneration](https://pic2.zhimg.com/80/v2-cbcf157a6a472066848f3623789a9565_hd.jpg)

We will use attention mechanism to generate questions from natural language contextual passage. By detecting entities in the passage, it will automatically get questions about the entities.

### 1.1 Input from Natural Language

We use the natural language passage from Wikipedia as input :
```
    Audrey Hepburn was a British actress and humanitarian. Recognised as a movie star and fashion icon, Hepburn was active during Hollywood's Golden Age. She was ranked by the American Film Institute as the third-greatest female screen legend in Golden Age Hollywood, and was inducted into the International Best Dressed List Hall of Fame. She rose to star in Roman Holiday in 1953.
```
![](https://pic1.zhimg.com/80/v2-6a37b3b2f4db3949137d90642df08ff4_hd.png)

### 1.2 Attention Layer with Embedding

The passage goes through attention layer with embedding:

```python
# T stands for time_steps, timing length
def embedding_attention_seq2seq(encoder_inputs, # [T, batch_size]
                             	Decoder_inputs, # [T, batch_size]
                             	Cell,
                             	Num_encoder_symbols,
                             	Num_decoder_symbols,
                             	Embeddding_size,
                             	Num_heads=1, # attention of the number of taps
                             	Output_projection=None, #decoder's projection matrix
                             	Feed_previous=False,
                             	Dtype=None,
                             	Scope=None,
                             	Initial_state_attention=False):

```
```python
encoder_cell = rnn_cell.EmbeddingWrapper(
        cell, embedding_classes=num_encoder_symbols,
        embeddding_size=embedding_size)
encoder_outputs, encoder_state = rnn.rnn(
        encoder_cell, encoder_inputs, dtype=dtype) 

top_states = [array_ops.reshape(e, [-1, 1, cell.output_size]) \
                  for e in encoder_outputs] 
attention_states = array_ops.concat(1, top_states) 

```
```python
def embedding_attention_decoder(decoder_inputs,
                                initial_state,
                                attention_states,
                                cell,
                                num_symbols,
                                embeddding_size,
                                num_heads=1,
                                output_size=None,
                                output_projection=None,
                                feed_previous=False,
                                update_embedding_for_previous=True,
                                dtype=None,
                                scope=None,
                                initial_state_attention=False):
# core code
    embedding = variable_scope.get_variable("embedding",
                                            [num_symbols, embedding_size])
    loop_function = _extract_argmax_and_embed(
        embedding, output_projection,
        update_embedding_for_previous) if feed_previous else None
    emb_inp = [
        embeddding_ops.embedding_lookup(embedding, i) for i in decoder_inputs]
    # T * [batch_size, embedding_size]
    return attention_decoder(
        emb_inp,
        initial_state,
        attention_states,
        cell,
        output_size=output_size,
        num_heads=num_heads,
        loop_function=loop_function,
        initial_state_attention=initial_state_attention)

```
### 1.3 Output The Question
Then we get the output question:
```
    "Which movie does Audrey Hepburn star ?"
```

## 2. From Natural Language Questions to Templates

![](https://pic2.zhimg.com/80/v2-a23047e0556d185a03e86f145659d625_hd.jpg)

With the help of semantic parsing, the model extracts the templates from the questions.

### 2.1 Entity Annotation
With the help of DBpedia's application Spotlight, the model is able to detect the entities in the question:
```bash
\examples\dbpedia>python annotation.py "Which movie does Audrey Hepburn star ?"
```
```json
{'Audrey Hepburn': {'@URI': 'http://dbpedia.org/resource/Audrey_Hepburn', 'Ref': 'Audrey_Hepburn', 'Schema': 'Person', 'DBpedia': ['Person', 'Agent']}}
the template is : ('Which movie does <A> star ?', ['Person'])
```
### 2.2 Template Generation With Semantic Parsing
We deploy the method of semantic parser to get the core structure of a question for query generation:
```python 
question = "Which movie does Audrey Hepburn star ?"
parser = semantic_parser(question)
regex = parser.match_regex()
```

```bash
regex
>>>Question(Pos("DT")) + nouns(Pos("NN") | Pos("NNS") | Pos("NNP") | Pos("NNPS"))
```
given thes above, the query template is generated:
```json
'query': IsMovie() + HasName(name)
```
then, we can get the primitive query template:
```
SELECT DISTINCT ?a WHERE {
   ?a rdf:type dbpedia-owl:Film.
   ?x0 dbpprop:starring ?b.
   ?a foaf:name ?x2.
   ?b rdf:type <A>
   ?b rdfs:label "Audrey
   ?b rdfs:label "Audrey}
```

## 3. Construction of the Templates Bank
The Templates Bank is a base where stores the existed templates. 
The basic idea is to use the DBpedia entities embedding vectors to encode the question templates in order to match the existent templates.

![TemplatesMatching](https://pic1.zhimg.com/80/v2-4a5f6b5e4f26dfb80083e9a3e3c337ec_hd.jpg)


## 4. Comparison of Newly Generated Templates and the Model Matching
The templates generated in the previous step shall be compared against the tempalets in the Templates Bank as to match the similar one. Then, if matched, it will refer to the similar trained model that contains the matched template; if not matched, it turns to train the model on the new templates, and store the new results into the Templates Bank.