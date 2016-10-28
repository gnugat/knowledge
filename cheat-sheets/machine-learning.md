# Machine Learning

A highly opinionated ML cheat sheet.

## Classification

### Bag of Words Model

Assuming the occurence of each words can be used as a feature for training a classifier.

#### How it works

1. fitting: "learn" vocabulary by extracting each words from all sentences
2. transforming: extract vocabulary word count for each sentence
3. optionally keep only the most x used words

#### Reference

[Bag of Words for beginners](https://www.kaggle.com/c/word2vec-nlp-tutorial/details/part-1-for-beginners-bag-of-words)
by [Kaggle](https://www.kaggle.com/)

### Naive Bayes Classifier

Assuming that a statement probability of being of a certain type depends if this type is more likely
and if the statement contains more words of this type.

#### How it works

1. "learning" types:
   * count the total number of documents
   * count how many times a word from a statement occurs for a given type
2. "guessing" types:
   * calculate type probability: count of documents of this type / total count of documents
   * calculate statement type probability: for each words in the statements
     * calculate word type probability: count occurence of this word for this type / total number of words
     * multiply all word type probabilities
   * calculate likelihood: multiply type probability by statement type probability
   * the statement is likely being of the type that has the highest likelihood

#### Reference

[Machine Learning: Naive Bayes](https://stovepipe.systems/post/machine-learning-naive-bayes),
by [Yannick de Lange](https://twitter.com/yannickl88)

### Random Forest Classifier

#### How it works

1. "training":
   * take a subset (~66%) of the data
   * for each node:
     - take randomly _m_ items from the subset
     - pick the item that provides the best split and use it to do a binary split on that node
     - for the next node, do the same with another _m_ random items from the subset
2. "running":
   * run input down all the trees
   * take the average, or weighted average, or voting majority of all the results

> _m_ can be either (your choice):
>
> * 1 (random splitter selection)
> * total number of items (breiman’s bagger)
> * something in between, e.g. ½√m, √m, and 2√m (random forest)

#### Reference

[A Gentle Introduction to Random Forests](https://citizennet.com/blog/2012/11/10/random-forests-ensembles-and-performance-metrics/),
by [Dan Benyamin](https://twitter.com/dbenyamin)
