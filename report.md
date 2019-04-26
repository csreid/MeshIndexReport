Author: Cameron Reid
Title: MeSH Index Classification

[TITLE]

# Data Exploration

The provided dataset was a bag-of-words transformation of 303,076 medical abstracts, which are to be labelled with any number of 126 class labels.

## Class Labels

As shown in Figure [#summary-class], there is wide variance in the frequency of class labels; ~95% of the training data is labelled with class 11, while only a small fraction of a percent (275 total) are labelled with class 22. This presents a challenge; sparsely-represented classes only carry a small amount of the information about the class.

~ Figure { #summary-class; caption: "Summary of class label distribution" }

![classfreqs]

| Mininum Frequency | Mean Frequency | Maximum Frequency |
|:-----------------:|:--------------:|:-----------------:|
|        0.0009     |    0.1078      |        0.9499     |

~

## Features

Figure [#summary-feats] shows a summary of the distribution of the features. Many of the features are present in several of the class labels, but the distribution has a long tail, with many features being nonzero in only a small portion (or none) of the samples.

~ Figure { #summary-feats; caption: "Summary of feature distribution" }

![featfreqs]

| Mininum Frequency | Mean Frequency | Maximum Frequency |
|:-----------------:|:--------------:|:-----------------:|
|       0.0000      |    0.0060      |        0.2301     |

~

To uncover feature importances, I trained a 100-tree random forest to a depth of 3 with information gain as the splitting criteria. The results of that are shown in Figure [#feature-importances].

Comparing this to the distribution in Figure [#summary-feats], it appears that very little discrimintating information is carried in the sparsely respresented features. However, the random forest was not trained enough to sufficiently distinguish some of the more rare class labels, so it is possible that those features are helpful in those cases.

~ Figure { #feature-importances; caption: "Feature importances" }

![featimport]

~

## Baseline Classifier

To set a baseline, a linear SVM with C=1 was trained for each class label. Before training, the data set was transformed via tf-idf and scaled to unit norm. This scaling and transformation step causes the SVM training to converge more quickly.

Several of the labels are very easy to classify, achieving F1 scores from this baseline near 1. On the other hand, several labels appear to be very difficult to classify and the baseline model achieves very low F1 scores [^sampling].

~ Figure { #baselinef1; caption: "Summary of baseline F1 scores across classes. The mean value is indicated by a red line." }

![baselinef1]

| Mininum F1        | Mean F1        | Maximum F1        |
|:-----------------:|:--------------:|:-----------------:|
|       0.1316      |    0.6006      |        0.9824     |

~

# Data Augmentation

Because some of the labels are very sparsely represented, it was useful to generate new samples that follow the same distribution as the members of those classes. To achieve this, I modeled instances belonging to a given class as a multinomial distribution whose mean vector is the class mean vector and whose number of trials is a normal random variable with $\mu$ equal to the mean of the number of words in the vector (i.e., $\frac{1}{|\bold{x}|}\underset{x \in \bold{x}}{\Sigma}x$), and then sampled from this distribution. To prevent doing unnecessary computation, I only do the augmentation on classes with support below some threshold. Because of this, the impact is moderate; however, for classes where it applies, it can substantially improve results, as shown in Figure [#f1delta].

~ Figure { #f1delta; caption: "F1 score, along with gain from augmentation. In this case, I generated 2000 samples for classes with fewer than 1000 instances." }

![deltaf1]

~

# Heirarchical Classification

Looking a bit into the data, it becomes clear that some of the class labels are total subsets of other class labels -- for example, every instance which is labelled with class 50 is also labelled with class 0. Some classes are identical to others; class 6 and class 58 share all of their member instances.

Once the superset/subset relationships are identified, I train a model to identify the superset, and then train the subset models using only data from the superset. Then, during prediction, I check for membership in the superset before attempting prediction with any of the subset models.

This approach has two benefits. First, there are substantial performance benefits because during training, most of the models can be trained on only a subset of the data without losing generality. Second, data which is noisy in the full dataset may be more easily separable.

# Multilayer Model

Beyond strict superset/subset relationships, there are softer relationships between classes, and some classes are consistently poorly modeled. In order to capture this information, I train another model for each class which takes the predictions generated by the first layer and predicts the class. In this way, I capture both the weaknesses of the first layer and the inter-model correlation that isn't captured by the subset analysis mentioned above.

# Results

[^sampling]: To validate methods without submitting to Kaggle, I do cross validation on 25% of the dataset. Sampling is done globally (i.e., across all classes), but stratified along the rarest class. F1 scores for very rare classes appear to be highly dependent on sampling; during the competition, I've seen F1 scores for the rarest class ranging from 0.1 to 0.3 with the same methods.

[classfreqs]: ./classfreqs.png { width: 100% }
[featfreqs]: ./featfreqs.png { width: 100% }
[featimport]: ./featimportances.png { width: 100% }
[baselinef1]: ./baselinef1.png { width: 100% }
[deltaf1]: ./f1delta.png { width: 100% }
