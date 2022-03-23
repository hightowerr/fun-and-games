---
layout: posts
title:  "Book recommendation using Affinity analysis"
date:   2021-12-05 13:00:00 +0100
categories: Data_Science
author: Olayinka Ola
---
A product recommender system is a system with the goal of predicting and compiling a list of items that a customer is likely to purchase. Recommender systems are widely deployed in multiple fields like book recommendations, music preferences and so on. Studies have shown that personalised product recommendations improve conversion rates and customer retention rates.

### The book recommendation problem

In this post, I will use affinity analysis to determine which books are typically reviewed together.

Affinity analysis is the study of what goes with what. It helps identify customer patterns when items/experiences are used in similar ways.

Link to my [Github Repo][Github Repo]

## The code
### Import of the packages is needed
```python
import numpy as np
import pandas as pd
from zipfile import ZipFile, Path
from collections import defaultdict
from sklearn.model_selection import GroupShuffleSplit
from operator import itemgetter
import sys
```

## The data

The book dataset used in this post can be found [here](http://www2.informatik.uni-freiburg.de/~cziegler/BX/). With the archive.zip file successfully downloaded I can open the zipfile without extracting its contents.

The Book-Crossing dataset comprises 3 tables.

- **BX-Users**

    Contains the users. Note that user IDs (*`User-ID`*) have been anonymized and map to integers.

- **BX-Books**

    Books are identified by their respective ISBN. Invalid ISBNs have already been removed from the dataset.

- **BX-Book-Ratings**

    Contains the book rating information. Ratings (*`Book-Rating`*) are either explicit, expressed on a scale from 1-10 (higher values denoting higher appreciation), or implicit, expressed by 0.

## Importing the data

```python
# specifying the zip file name
file_name = "Archive.zip"

# opening the zip file in READ mode
with ZipFile(file_name, 'r') as zip:
    # printing all the contents of the zip file
    zip.printdir()

    # extracting all the files
    print('Extracting all the files now...')
    zip.extractall()
    print('Done!')
```

Happy with this output I pass in the specific file name to open the ratings.csv. Pandas is then used to store the ratings data as a new data frame.

```python
# pass in the specific file name
# to the open method
with ZipFile("Archive.zip") as myzip:
    data = myzip.open("Ratings.csv")

#Now, we can read in the data
df = pd.read_csv(data, sep=';')
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Read_data.png" alt="read_data">

## Preparing the data

The format of the data given here is compact. The user ID indicates a unique user the second column is the book reviewed and the third column gives the review ranking out of 10.

For example user 276725 left no reviews on any of the five books shown in the sample.

Observations where no reviews have been given are of no value to my model and so I am going to exclude any rows where no review has been left.

```python
df2 = df[(df['Rating'] != 0)]
df = df2.copy()
```

For my model I create a new feature where a positive rating - signified as greater than 4 -  has been provided.

```python
# positive ratings
df['Positive'] = df["Rating"] > 4
```

### Create Train and Test data
To avoid overfitting / under fitting  - issues where the model learns the data too well or too little - I create seperate training and test datasets.  The most common split ratio is 80/20. 80% of the data goes into the training set and 20% of the data goes into the testing set. Both dataframes are written into new labelled csvs.

```python
# Create Train and test data
train_inds, test_inds = next(GroupShuffleSplit(test_size=.20, n_splits=2, random_state = 100).split(df, groups=df['User-ID']))
train = df.iloc[train_inds]
test = df.iloc[test_inds]

# Writes to a CSV file
train.to_csv('book_train.csv')
test.to_csv('book_test.csv')
```

The newly minted training dataset is imported and a data set of only the positive reviews are returned.

```python
top_ratings_train = pd.read_csv('book_train.csv')
# Dataset of only favourable reviews
fav_rating = top_ratings_train[top_ratings_train["Positive"]]; fav_rating
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/positive rating.png" alt="positive rating">

The following code uses frozensets to store which books were reviewed by each user. A pandas dataframe is created showing how frequently each book has been given a positive review.

```python
# the books which each user has given a favorable review
postive_reviews_by_users = dict((k, frozenset(v.values))
                               for k, v in fav_rating.groupby("User-ID")["ISBN"]);

# create a DataFrame that tells us how frequently each book has been given a favorable review
num_positive_by_book = top_ratings[["ISBN", "Positive"]].groupby('ISBN').sum(); num_positive_by_book
# The top five book list
num_positive_by_book.sort_values("Positive", ascending = False)[:5]
```

Displayed are the top five most reviewed books.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/num_positive_by_book.png" alt="Number of positive reviews by book">

## The Apriori Algorthim.

The Apriori method is a tool to find frequent itemsets. A frequent itemset is a set of items that meet the threshold for minimum support. The main principle behind it as explained in ****book by Michael Steinbach, Pang-Ning Tan, and Vipin Kumar, **Introduction to Data Mining**

"If an itemset is frequent, then all of its subsets must also be frequent."

This video by [CSE GURUS](https://www.youtube.com/channel/UCnaDKXKHRc2_V6shuoZMpiA) offers a great explanation of how the apriori algorithm works.

 [Link to video](https://www.youtube.com/watch?v=h_l3b2CIQ_o)

 Here are the broad strokes of the Apriori Algorithm steps followed below:

1. First we initialise a dictionary to store discovered frequent item sets. We then have to define a minimal support this is the threshold manually set for an item set to be considered frequent.
2. Create candidate itemsets using supersets of existing frequent itemsets
3. Test candidate itemsets to see if they are frequent, discarding those that are not. If there are no new frequent itemsets from this step, go to step 5.
4. Loop to find all the newly discovered frequent itemsets. We break out the preceeding loop if no new frequent itemsets are found. We need itemsets with at least two items so those with an itemset length of one can be deleted.
5. Return all discovered frequent itemsets.

**Step 1**

```python
frequent_itemsets = {}
min_support = 50 # minimum support needed to be considered frequent
frequent_itemsets[1] = dict((frozenset((ISBN,)), row["Positive"])
                  for ISBN, row in num_positive_by_book.iterrows()
                  if row["Positive"] > min_support)
print(frequent_itemsets)
```

**Step 2 & 3**

```python
def find_frequent_itemsets(postive_reviews_by_users, k_1_itemsets, min_support):
    counts = defaultdict(int)
    # iterate over all of the users and their reviews
    for user, reviews in postive_reviews_by_users.items():
        for itemset in k_1_itemsets:
            if itemset.issubset(reviews):
                for other_reviewed_book in reviews - itemset:
                    current_superset = itemset | frozenset((other_reviewed_book,))
                    counts[current_superset] += 1
    return dict([(itemset, frequency) for itemset, frequency in counts.items() if frequency >= min_support])
```

**Step 4**

```python
for k in range(2, 20):
    cur_frequent_itemsets = find_frequent_itemsets(postive_reviews_by_users, frequent_itemsets[k-1], min_support)
    frequent_itemsets[k] = cur_frequent_itemsets
    # break out the preceding loop if we didn't find any new frequent itemsets
    if len (cur_frequent_itemsets) == 0:
        print("Didn't find any frequent itemset length {}".format(k))
        sys.stdout.flush()
        break
    else:
        print("I found {} frequent itemsets of length {}".format(len(cur_frequent_itemsets), k))
        sys.stdout.flush()
# these are itemsets of length one, we need at least two items - Let's delete them     
del frequent_itemsets[1]
```

**Step 5**

```python
frequent_itemsets[5].items()
```

## Mining association rules

Now we have a list of frequent item sets, but this by iteself is not very useful. The next step is to build an association rule. Association rules help find patterns and relationships among items in frequent itemsets. This technique is crucial to actionable cross-selling because it allows us to formulate non-deterministic premise and conclusion statements such as ' if a user recommends book_x and book_y, they will also recommend book_z'

Click here for a good video on [premises and conclusion statements](https://www.youtube.com/watch?v=07mehbgE5jc)

In the code below, we iterate over each frequent itemset.  We then iterate over every book, using it as the conclusion. The remaining books in the itemset are the premise. The premise and conclusion are added to the candidate rules list.

```python
candidate_rules = []
for itemset_length, itemset_counts in frequent_itemsets.items():
    for itemset in itemset_counts.keys():
        for conclusion in itemset:
            premise = itemset - set((conclusion,))
            candidate_rules.append((premise, conclusion))
    print(candidate_rules[:5])
```

Next we compute the confidence of each of these rules. First we create dictionaries to store how frequently we see the premise leading to the conclusion this would represent a correct example of the rule. we also store how many times it doesn’t and this is an incorrect rule example.

Each user and their positive reviews are iterated over. We then test if each user gave all the books in the premise a positive review if they did then we check if the conclusion book was also given a positive review. If it was then all is correct if it wasn’t the rule was incorrect.

Once we looped through all the users and their reviews we compute the confidence for each rule by dividing the correct rule count by the sum of correct and incorrect rule counts.

```python
correct_counts = defaultdict(int)
incorrect_counts = defaultdict(int)

for user, reviews in postive_reviews_by_users.items():
    for candidate_rule in candidate_rules:
        premise, conclusion = candidate_rule
        if premise.issubset(reviews):
            if conclusion in reviews:
                correct_counts[candidate_rule] += 1
            else:
                incorrect_counts[candidate_rule] += 1

rule_confidence = {candidate_rule: correct_counts[candidate_rule]
                   /float(correct_counts[candidate_rule] + incorrect_counts[candidate_rule])
                  for candidate_rule in candidate_rules}
```

## Get book names

Until  this point we have been working with the ISBN which isn’t very useful when it comes to reporting. The function below will return a Books title from each ISBN.

```python
def get_book_name(book_id):
    title_object =  df_books[df_books['ISBN'] == book_id] ["Title"]
    title = title_object.values[0]
    return title
```

If you remember from the beginning of this post the dataset being used comprises three tables this includes a Books table. From this table I can retrieve the book title

```python
# pass in the specific file name
# to the open method
with ZipFile("Archive.zip") as myzip:
    data = myzip.open("Books.csv")

#Now, we can read in the data
df_books = pd.read_csv(data, sep=';')
```

now we can run the following for loop which will print out the top 10 rules with the premise and conclusion using The book name rather than the ISBN.

```python
sorted_confidence = sorted(rule_confidence.items(), key=itemgetter(1), reverse=True)
for index in range(10):
    print("Rule #{0}".format(index + 1))
    (premise, conclusion) = sorted_confidence[index][0]
    premise_names = ", ".join(get_book_name(idx) for idx in premise)
    conclusion_name = get_book_name(conclusion)
    print("Rule: if a person recommend {0} they will also recommend {1}".format(premise_names, conclusion_name))
    print(" - Confidence:{0:.3f}".format(rule_confidence[(premise, conclusion)]))
    print("")
```

```
Rule #1
Rule: if a person recommend Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Chamber of Secrets (Book 2)
 - Confidence:0.970

Rule #2
Rule: if a person recommend Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Prisoner of Azkaban (Book 3), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Chamber of Secrets (Book 2)
 - Confidence:0.968

Rule #3
Rule: if a person recommend Harry Potter and the Goblet of Fire (Book 4), Harry Potter and the Chamber of Secrets (Book 2), Harry Potter and the Order of the Phoenix (Book 5) they will also recommend Harry Potter and the Prisoner of Azkaban (Book 3)
 - Confidence:0.964

Rule #4
Rule: if a person recommend Harry Potter and the Goblet of Fire (Book 4), Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Chamber of Secrets (Book 2)
 - Confidence:0.958

Rule #5
Rule: if a person recommend Harry Potter and the Goblet of Fire (Book 4), Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Prisoner of Azkaban (Book 3)
 - Confidence:0.958
```

To evaluate the performance of the Association rules we will use test dataset to see how the model performs on unseen data

## Test Dataset

First, we import the test data set. As with the training set we get the positive reviews for each user and then count the correct and incorrect instances as we did on the training set. We then compute the confidence of each association rule for comparison.

```python
test_dataset = pd.read_csv('book_test.csv')

# Dataset of only favourable reviews
test_positive = test_dataset[test_dataset["Positive"]]
test_positive_by_users = dict((k, frozenset(v.values))
                                  for k, v in test_positive.groupby("User-ID")["ISBN"])

correct_counts = defaultdict(int)
incorrect_counts = defaultdict(int)
for user, reviews in test_positive_by_users.items():
    for candidate_rule in candidate_rules:
        premise, conclusion = candidate_rule
        if premise.issubset(reviews):
            if conclusion in reviews:
                correct_counts[candidate_rule] += 1
            else:
                incorrect_counts[candidate_rule] += 1

test_confidence = {candidate_rule: correct_counts[candidate_rule]
                   / float(correct_counts[candidate_rule] + incorrect_counts[candidate_rule])
                  for candidate_rule in rule_confidence}
```

Finally, we print out 10 of the association and can see which rules are most applicable in new unseen data

```python
for index in range(10):
    print("Rule #{0}".format(index + 1))
    (premise, conclusion) = sorted_confidence[index][0]
    premise_names = ", ".join(get_book_name(idx) for idx in premise)
    conclusion_name = get_book_name(conclusion)
    print("Rule: if a person recommends {0} they will also recommend {1}".format(premise_names, conclusion_name))
    print(" - Train Confidence:{0:.3f}".format(rule_confidence.get((premise, conclusion), -1)))
    print(" - Test Confidence:{0:.3f}".format(test_confidence.get((premise, conclusion), -1)))
    print("")
```

```
Rule #1
Rule: if a person recommends Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Chamber of Secrets (Book 2)
 - Train Confidence:0.970
 - Test Confidence:1.000

Rule #2
Rule: if a person recommends Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Prisoner of Azkaban (Book 3), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Chamber of Secrets (Book 2)
 - Train Confidence:0.968
 - Test Confidence:1.000

Rule #3
Rule: if a person recommends Harry Potter and the Goblet of Fire (Book 4), Harry Potter and the Chamber of Secrets (Book 2), Harry Potter and the Order of the Phoenix (Book 5) they will also recommend Harry Potter and the Prisoner of Azkaban (Book 3)
 - Train Confidence:0.964
 - Test Confidence:0.857

Rule #4
Rule: if a person recommends Harry Potter and the Goblet of Fire (Book 4), Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Chamber of Secrets (Book 2)
 - Train Confidence:0.958
 - Test Confidence:1.000

Rule #5
Rule: if a person recommends Harry Potter and the Goblet of Fire (Book 4), Harry Potter and the Order of the Phoenix (Book 5), Harry Potter and the Sorcerer's Stone (Book 1) they will also recommend Harry Potter and the Prisoner of Azkaban (Book 3)
 - Train Confidence:0.958
 - Test Confidence:1.000
```

Rule #3 shows a 96% confidence level in the training data, but it has only ~85% level in the test data. Many of the other rules show a perfect confidence in test data which makes them great candidates for recommendations.

Summary

In this post, we used affinity analysis to recommend books for review based on booked reviewed together in the past. This was done in two stages. First we found frequent item-sets using the Apriori algorithm. We then created association rules from those item-sets. Association rules were found using training data and then applied to the test data.

### References

**Python Real-World Data Science** by Dusty Phillips, Fabrizio Romano, Phuong Vo.T.H, Martin Czygan, Robert Layton, Sebastian Raschka

[Github Repo]: https://github.com/hightowerr/marketing/blob/master/Affinity%20Analysis/Book%20recommendation.ipynb
