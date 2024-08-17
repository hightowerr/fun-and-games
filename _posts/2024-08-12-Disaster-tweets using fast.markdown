---
layout: posts
title:  "A Step-by-Step Journey into Sentiment Analysis: Exploring, Engineering, and Predicting with Real Disaster Data"
header:
  image: /assets/images/rubaitul-azad-Fty8u3JZ2T8-unsplashed.jpg
  og_image: /assets/images/rubaitul-azad-Fty8u3JZ2T8-unsplashed.jpg
date:   2024-08-14 09:00:00 +0100
categories: Sentiment analysis, Fast.ai, Transformers
author: Olayinka Ola
---

In this project, I'll leverage a pre-trained English language model to classify tweets as either real disaster reports or not. The process will involve data exploration, feature engineering, and model application using best practices for train-test-validation splits. By following this guide, you'll implement a straightforward pipeline achieving a submission score of approximately 0.81121.

[Link to the notebook](https://www.kaggle.com/code/madcontender/disaster-tweets-eda-model-fast-ai)

**Exploring the Data: Understanding Sentiment Distribution and Tweet Characteristics**

When working with any dataset, the first step is to explore and understand the data. Here, we are dealing with tweets, classified as either related to real disasters or not. A quick look at the distribution of sentiment labels reveals a class imbalance: there are significantly more non-disaster tweets than real disaster tweets. This imbalance can affect the performance of our model, potentially making it biased towards the non-disaster tweets.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/class distribution of sentiment label.png" alt="class distribution of sentiment label">

Next, I looked at the length of the tweets. It turns out that tweets about real disasters tend to use more characters compared to non-disaster tweets.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Distribution of number of characters in tweets.png" alt="Distribution of number of characters in tweets">

Both categories typically use between 15 to 20 words, with similar stop words appearing in the top five for both classes.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Distribution of number of No. Words in Tweets.png" alt="Distribution of number of No: Words in Tweets">

However, there is a difference in the usage of punctuation. Real disaster tweets show a wider spread in punctuation usage, with a slightly left-skewed distribution, indicating that these tweets might have more varied emotional content.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Number of Punctuations in tweets.png" alt="Number of Punctuations in tweets">

Moving on to the unique words in each tweet, I found that both classes exhibit characteristics of a normal distribution. Interestingly, the distribution of unique words in real disaster tweets is more normalized compared to non-disaster tweets, suggesting a more consistent use of vocabulary when discussing real disasters.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Distribution of number of unique words.png" alt="Distribution of number of unique words">

Punctuation usage, particularly of URLs, affects the data distribution significantly. When looking beyond this, punctuation marks like question marks and exclamation marks appear more frequently in non-disaster tweets, indicating a different tone or level of uncertainty in these tweets. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Punctuations in tweets.png" alt="Punctuations in tweets">

On the other hand, real disaster tweets are often marked by more emotive language, as reflected in a word cloud generated during the analysis. This visualization vividly shows the difference in word choice between the two categories, with disaster-themed words standing out in real disaster tweets.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Common words in tweet text.png" alt="Common words in tweet text">

To better understand the nature of the text, I used the Flesch Reading Ease Test, which helps gauge how easy or difficult a text is to read. The mean score for our dataset was around 57.4, indicating that the tweets are moderately difficult to read. Interestingly, non-disaster tweets were easier to read than real disaster tweets. This could be due to the more complex or technical language used when discussing real disasters, or simply the heightened emotional tone.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Flesch Reading Ease.png" alt="Flesch Reading Ease">

Next, I analyzed the polarity and subjectivity of the tweets. Polarity measures how positive or negative the text is, while subjectivity measures how much of the text expresses personal opinions. The results showed that the polarity of real disaster tweets tends to be close to zero, indicating a more neutral tone, though with plenty of outliers that reflect more intense emotions. Non-disaster tweets had slightly more pronounced polarity, suggesting a more defined positive or negative tone. However, both datasets had numerous outliers, emphasizing the varied nature of tweets.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Tweet type vs Polarity of text.png" alt="Tweet type vs Polarity of text">

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Tweet type vs subjectivity of text.png" alt="Tweet type vs subjectivity of text">

Feature Engineering: Creating a Comprehensive Feature

Now that we have a good understanding of the data, it’s time to engineer a new feature. This new feature combines the contents of all existing features, including the tweet text itself, into a single feature. The idea is to retain the context of the content even when certain information, like keywords or location, is missing. This step helps ensure that our model has as much relevant information as possible during training.


| 	id | keyword | location | text | target | input |
|---------|----------------------|---------------------|
| 1 | NaN | NaN | Our Deeds are the Reason of this #earthquake May ALLAH Forgive us all | 1 | keyword: nan; location: nan; text: Our Deeds are the Reason of this #earthquake May ALLAH Forgive us all |

With our data prepped and ready, it was time to choose a model. I opted for Google BERT, specifically a pre-trained uncased model, meaning it doesn’t differentiate between uppercase and lowercase letters.

```python
model_nm = 'google-bert/bert-base-uncased'
```

The first step in the modelling process was tokenization, followed by splitting the data into a training set (80%) and a test set (20%).

```python
from transformers import AutoModelForSequenceClassification,AutoTokenizer
tokz = AutoTokenizer.from_pretrained(model_nm)
```

For training, I set up an F1 score metric to validate performance, adjusted my learning rate, batch size, and epoch settings through some trial and error, and then began training the model. 

```python
def compute_metrics(pred):
    labels = pred.label_ids
    preds = pred.predictions.argmax(-1)

    f1 = f1_score(labels, preds, average='weighted')

    return {'f1': f1}

```

After preparing the test set similarly to the training set, I applied my newly trained model to make predictions on the test set

```python
eval_df['input'] = 'keyword: ' + eval_df.keyword.fillna('nan') + '; location: ' + eval_df.location.fillna('nan') + '; text: ' + eval_df.text.fillna('nan')

eval_ds = Dataset.from_pandas(eval_df).map(tok_func, batched=True)
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/eval_df.png" alt="test set">

The final model yielded a score of 0.81121.

This step-by-step guide walks you through the process of exploring and preparing data, engineering features, and building a model to classify sentiment in tweets.

Disaster Tweets EDA + Model fast.ai: [Notebook Link](https://www.kaggle.com/code/madcontender/disaster-tweets-eda-model-fast-ai)