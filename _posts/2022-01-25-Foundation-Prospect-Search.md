---
layout: post
title: Text Summarization with Machine Learning
image: "/posts/pca-title-img.png"
tags: [NLP, Text Mining, Unstructured Data, Machine Learning, Python]
---

NLP can be used to analyze the language used in the mission statements, funding guidelines, and past grant awards of potential funding organizations. This helps in identifying foundations that align closely with your project goals and objectives. Applying NLP techniques in this context allows for a systematic and data-driven approach to identifying potential funding opportunities. By targeting organizations with missions closely aligned with the project goals, you increase the likelihood of securing funding and support for addressing Hispanic health disparities. This approach not only streamlines the grant writing process but also ensures that resources are directed towards initiatives with the greatest potential for impact in underserved communities.


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
The project aimed to identify potential funding organizations whose missions closely align with a grant proposal focused on addressing Hispanic health disparities. By leveraging Natural Language Processing (NLP) techniques, the goal was to streamline the process of prospect research and enhance the chances of securing funding for initiatives aimed at improving healthcare access and outcomes for Hispanic communities.

Hispanic populations in the United States face significant health disparities, including higher rates of chronic diseases, lower access to healthcare, and disparities in healthcare quality. Addressing these disparities requires targeted interventions and initiatives, often supported by funding from philanthropic organizations and government agencies. However, identifying suitable funding opportunities that align with project goals can be challenging and time-consuming.

<br>
<br>

### Actions <a name="overview-actions"></a>

Actions Taken as a Data Scientist:

#### Data Collection
Mission statements and funding guidelines were gathered from potential funding organizations focusing on health equity, minority health, and community outreach.
Preprocessing: Text data was preprocessed to remove noise, such as stopwords, punctuation, and special characters. The text was also converted to lowercase for consistency.

##### Feature Extraction
NLP techniques were used to extract key features from the mission statements, including word frequencies and topics related to health equity and minority health.
Similarity Analysis: Cosine similarity was calculated between the grant proposal's mission statement and the mission statements of potential funders to quantify alignment.

#### Ranking and Selection
Potential funding organizations were ranked based on their similarity scores with the grant proposal's mission statement. The top-ranked organizations were considered for further validation and selection.

<br>
<br>

### Results <a name="overview-results"></a>
The project successfully identified potential funding organizations that closely aligned with the grant proposal's goals of addressing Hispanic health disparities. By leveraging NLP techniques, the process of prospect research was streamlined, enabling more efficient identification of relevant funding opportunities. The top-ranked organizations provided promising avenues for securing funding and support for initiatives aimed at improving healthcare access and outcomes for Hispanic communities.

<br>
<br>
### Growth/Next Steps <a name="overview-growth"></a>

#### Validation and Selection
The next steps involve validating the identified funding opportunities by reviewing the mission statements and funding guidelines of the top-ranked organizations. This will help ensure alignment with project goals and objectives before pursuing funding applications.

#### Tailored Proposals
Future iterations of the project may involve tailoring grant proposals to better align with the specific priorities and focus areas of potential funding organizations. This could increase the likelihood of securing funding and support for targeted initiatives.

#### Continuous Monitoring
It's essential to continuously monitor the landscape of funding opportunities and adjust the prospect research strategy accordingly. This includes staying updated on new funding opportunities, changes in funding priorities, and emerging trends in health equity and minority health.

#### Collaboration and Networking
Building relationships with potential funding organizations and stakeholders within the Hispanic health community can facilitate collaboration and partnership opportunities. Engaging with key stakeholders can also provide valuable insights and support for project implementation and sustainability

<br>
<br>

___

# Data Overview  <a name="data-overview"></a>

Sample Data:
Let's create a list of sample mission statements from potential funding organizations related to health equity and minority health:

<br>
<br>

```python

# import required Python packages
import pandas as pd
import numpy as np
import nltk

nltk.download('punkt')
import re
from nltk.corpus import stopwords

```

<br>

```python
# Sample data
potential_funders = [
    {'name': 'Foundation A', 'mission': 'To improve healthcare access and outcomes for underserved minority populations.'},
    {'name': 'Foundation B', 'mission': 'Empowering Hispanic communities through culturally competent healthcare initiatives is at the core of our mission.'},
    {'name': 'Foundation C', 'mission': 'We are committed to addressing health disparities among Hispanic populations and promoting health equity for all.'},
    {'name': 'Foundation D', 'mission': 'Promoting health equity and reducing disparities in healthcare access and outcomes for minority communities is our primary objective.'},
]

```
<br>

#### Grant Proposal Topic:
The grant proposal focuses on addressing Hispanic health disparities. Let's define the project's mission statement:


```python
grant_topic = "To improve healthcare access and outcomes for Hispanic communities by addressing social determinants of health and promoting culturally competent care."
```

#### Preprocessing
```python

import re

def preprocess_text(text):
    # Convert text to lowercase
    text = text.lower()
    # Remove special characters, punctuation, and digits
    text = re.sub(r'[^a-zA-Z\s]', '', text)
    # Tokenize text
    tokens = text.split()
    # Remove stopwords (not using lemmatization here)
    stop_words = set(["the", "and", "to", "in", "of", "on", "for", "is", "are", "that", "this", "with", "it", "at", "as", "by"])
    tokens = [token for token in tokens if token not in stop_words]
    # Join tokens back into a string
    preprocessed_text = ' '.join(tokens)
    return preprocessed_text

# Preprocess potential funders' mission statements
preprocessed_mission_statements = [preprocess_text(funder['mission']) for funder in potential_funders]

```
#### Feature Extraction
```python
# Grant proposal topic
grant_topic = "Our project aims to improve healthcare access and outcomes for Hispanic communities by addressing social determinants of health and promoting culturally competent care."

# Preprocess grant proposal topic
preprocessed_grant_topic = preprocess_text(grant_topic)
```


#### Similarity Analysis
```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

# Combine grant proposal topic with potential funders' mission statements
all_texts = [preprocessed_grant_topic] + preprocessed_mission_statements

# Create TF-IDF vectorizer
tfidf_vectorizer = TfidfVectorizer()

# Fit TF-IDF vectorizer and transform texts into vectors
tfidf_matrix = tfidf_vectorizer.fit_transform(all_texts)

# Calculate cosine similarity between grant proposal topic and potential funders' mission statements
similarity_scores = cosine_similarity(tfidf_matrix[0:1], tfidf_matrix[1:])

```


#### Ranking and Selection
```python

import numpy as np

# Extract similarity scores for each potential funder
similarity_scores = similarity_scores[0]

# Rank potential funders based on similarity scores
ranked_funders_indices = np.argsort(similarity_scores)[::-1]
ranked_funders = [potential_funders[i] for i in ranked_funders_indices]

# Print the top-ranked potential funders
for idx, funder in enumerate(ranked_funders, start=1):
    print(f"Top {idx} Funder: {funder['name']} - Similarity Score: {similarity_scores[ranked_funders_indices[idx - 1]]:.2f}")

```

<br>
Top 1 Funder: Foundation D - Similarity Score: 0.32
Top 2 Funder: Foundation B - Similarity Score: 0.32
Top 3 Funder: Foundation A - Similarity Score: 0.27
Top 4 Funder: Foundation C - Similarity Score: 0.24
<br>
