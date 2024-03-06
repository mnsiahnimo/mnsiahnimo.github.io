---
layout: post
title: Text Summarization with Machine Learning
image: "/posts/text_summary.png"
tags: [NLP, Text Mining, Unstructured Data, Machine Learning, Python]
---


Text summarization entails compressing a text into a concise form while retaining essential details and its core meaning. As manual summarization is time-consuming, automated solutions are increasingly sought after, driving academic interest in Natural Language Processing (NLP) for text summarization using Machine Learning techniques.

Machine Learning holds significant relevance in text summarization, playing a crucial role in diverse Natural Language Processing tasks like text classification, question answering, legal document synthesis, news aggregation, and headline generation. The aim of this project is to produce a concise and coherent summary that captures the key points of the original document, which offered the company the ability to efficiently process vast amounts of text data, extract actionable insights, and make informed decisions in a timely manner.  This enhanced efficiency in information processing, improved communication, better resource allocation, and ultimately, greater profitability and success in the marketplace.


# Table of contents

- [00. Project Overview](#overview-main)
    - [Context](#overview-context)
    - [Actions](#overview-actions)
    - [Results](#overview-results)
    - [Growth/Next Steps](#overview-growth)
- [01. Data Overview](#data-overview)
- [02. Tokenize Text](#tokenize-text)
- [03. Word Embeddings](#word-embeddings)
- [04. Summarize Text](#summarize-text)

___

# Project Overview  <a name="overview-main"></a>

### Context <a name="overview-context"></a>

Our client is looking to generate  a summaries from their text based, unstructured database -  to  extract actionable insights, and make informed decisions in a timely manner, which will improve their efficieny and ultimately speed and accuracy which generates much value above their competitors in the marketplace.

In this article, I'll demonstrate how to perform text summarization using Machine Learning and Python, employing the extractive approach. Specifically, I'll leverage the TextRank algorithm, an unsupervised machine learning technique, for generating summaries from text.

As a proof-of-concept they would like us to build a minimum viable product for generation of summaries which could be improved upon for accuarcy and tailored relevant insights. 

The sample data contains unstructured data in text.

<br>
<br>
### Actions <a name="overview-actions"></a>

To begin, we'll gather and explore the required data to understand its content. Then, we'll tokenize the sequences by creating a list. Following that, we'll utilize the GloVe method for word representation, an unsupervised learning algorithm developed by Stanford University. This method involves training on aggregated global word-word co-occurrence statistics from a corpus, resulting in vector representations that reveal intriguing linear substructures within the word vector space.

Next, we'll compute similarities between the sentences using the cosine similarity approach. These similarities will then be converted into a graph, where nodes represent sentences and edges represent similarity scores between them. Finally, we'll generate the summary.

<br>
<br>

### Results <a name="overview-results"></a>
A minimum viable product thea will effectively generate summaties from text was built using the sample data provided by our client. This will be a pioneering effort to have a model that will thrust efficieny with providing actionable insights. 

<br>
<br>
### Growth/Next Steps <a name="overview-growth"></a>

We will test several other  models and methods  like the Abstraction approach which aims to produce a summary by interpreting the text using advanced natural language techniques to generate a new, shorter text – parts of which may not appear in the original document, which conveys the most information.This will build upon this parsimouous yet robust model and will make our client more efficient in generating insights

<br>
<br>

___

# Data Overview  <a name="data-overview"></a>

In the code below, we:

* Import the required python packages & libraries
* Import the data from the database
* Tokenize the sequences by creating a list. 
* Utilize the Glove method for word representation.
* Summarize Text
<br>

<br>
#### Import Packages 

```python
#import required Python packages

import pandas as pd
import numpy as np
import nltk
nltk.download('punkt')
import re
from nltk.corpus import stopwords
```
<br>
<br>
### Import data

```python
df = pd.read_csv("text_doc.csv")
df.head()
```

```python
df['article_text'][1]
```
<br>
<br>
"BASEL, Switzerland (AP), Roger Federer advanced to the 14th Swiss Indoors final of his career by beating seventh-seeded Daniil Medvedev 6-1, 6-4 on Saturday. Seeking a ninth title at his hometown event, and a 99th overall, Federer will play 93th-ranked Marius Copil on Sunday. Federer dominated the 20th-ranked Medvedev and had his first match-point chance to break serve again at 5-1. He then dropped his serve to love, and let another match point slip in Medvedev's next service game by netting a backhand. He clinched on his fourth chance when Medvedev netted from the baseline. Copil upset expectations of a Federer final against Alexander Zverev in a 6-3, 6-7 (6), 6-4 win over the fifth-ranked German in the earlier semifinal. The Romanian aims for a first title after arriving at Basel without a career win over a top-10 opponent. Copil has two after also beating No. 6 Marin Cilic in the second round. Copil fired 26 aces past Zverev and never dropped serve, clinching after 2 1/2 hours with a forehand volley winner to break Zverev for the second time in the semifinal. He came through two rounds of qualifying last weekend to reach the Basel main draw, including beating Zverev's older brother, Mischa. Federer had an easier time than in his only previous match against Medvedev, a three-setter at Shanghai two weeks ago."
<br>

<br>
# Tokenize text <a name="tokenize-text"></a>

```python
from nltk.tokenize import sent_tokenize
sentences = []
for s in df['article_text']:
    sentences.append(sent_tokenize(s))

sentences = [y for x in sentences for y in x]

```
<br>

<br>
# Word embeddings <a name="word-embeddings"></a>

```python
word_embeddings = {}
f = open('glove.6B.100d.txt', encoding='utf-8')
for line in f:
    values = line.split()
    word = values[0]
    coefs = np.asarray(values[1:], dtype='float32')
    word_embeddings[word] = coefs
f.close()

clean_sentences = pd.Series(sentences).str.replace("[^a-zA-Z]", " ")
clean_sentences = [s.lower() for s in clean_sentences]
stop_words = stopwords.words('english')
def remove_stopwords(sen):
    sen_new = " ".join([i for i in sen if i not in stop_words])
    return sen_new
clean_sentences = [remove_stopwords(r.split()) for r in clean_sentences]

```
<br>
<br>
##### Create vectors for sentences

```python

sentence_vectors = []
for i in clean_sentences:
  if len(i) != 0:
    v = sum([word_embeddings.get(w, np.zeros((100,))) for w in i.split()])/(len(i.split())+0.001)
  else:
    v = np.zeros((100,))
  sentence_vectors.append(v)

```
<br>
<br>
#### Find similarities between sentences using the cosine

```python

simi_mat = np.zeros([len(sentences), len(sentences)])
from sklearn.metrics.pairwise import cosine_similarity
for i in range(len(sentences)):
  for j in range(len(sentences)):
    if i != j:
      simi_mat[i][j] = cosine_similarity(sentence_vectors[i].reshape(1,100), sentence_vectors[j].reshape(1,100))[0,0]

```
<br>
<br>
##### Convert similar matrix into graph

Convert the simi_mat similarity matrix into the graph, the nodes in this graph will represent the sentences and the edges will represent the similarity scores between the sentences

```python

import networkx as nx

nx_graph = nx.from_numpy_array(sim_mat)
scores = nx.pagerank(nx_graph)

```
<br>
<br>
# Summarize Text <a name="summarize-text"></a>

There is no right or wrong number of components to use - this is something that we need to decide based upon the scenario we're working in.  We know we want to reduce the number of features, but we need to trade this off with the amount of information we lose.

In the following code, we extract this information from the prior step where we fit the PCA object to our training data.  We extract the variance for each component, and we do the same again, but for the *cumulative* variance.  Will will assess & plot both of these in the next step.

```python
ranked_sentences = sorted(((scores[i],s) for i,s in enumerate(sentences)), reverse=True)
for i in range(5):
  print("ARTICLE:")
  print(df['article_text'][i])
  print('\n')
  print("SUMMARY:")
  print(ranked_sentences[i][1])
  print('\n')

```
<br>
ARTICLE:
Maria Sharapova has basically no friends as tennis players on the WTA Tour. The Russian player has no problems in openly speaking about it and in a recent interview she said: 'I don't really hide any feelings too much. I think everyone knows this is my job here. When I'm on the courts or when I'm on the court playing, I'm a competitor and I want to beat every single person whether they're in the locker room or across the net.So I'm not the one to strike up a conversation about the weather and know that in the next few minutes I have to go and try to win a tennis match. I'm a pretty competitive girl. I say my hellos, but I'm not sending any players flowers as well. Uhm, I'm not really friendly or close to many players. I have not a lot of friends away from the courts.' When she said she is not really close to a lot of players, is that something strategic that she is doing? Is it different on the men's tour than the women's tour? 'No, not at all. I think just because you're in the same sport doesn't mean that you have to be friends with everyone just because you're categorized, you're a tennis player, so you're going to get along with tennis players. I think every person has different interests. I have friends that have completely different jobs and interests, and I've met them in very different parts of my life. I think everyone just thinks because we're tennis players we should be the greatest of friends. But ultimately tennis is just a very small part of what we do. There are so many other things that we're interested in, that we do.'

SUMMARY:
When I'm on the courts or when I'm on the court playing, I'm a competitor and I want to beat every single person whether they're in the locker room or across the net.So I'm not the one to strike up a conversation about the weather and know that in the next few minutes I have to go and try to win a tennis match.


