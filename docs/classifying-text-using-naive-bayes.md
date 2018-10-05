## Classifying Text using Naïve Bayes

### In this Lab

**Objective:** To train a ***NaiveBayesClassifier*** in NLTK

**Successful outcome:** You will be able to test whether a comment is positive or not based on a trained ***NaiveBayesClassifier*** that you create using NLTK

----
### Steps

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Open the Existing Notebook</h2>

1\.  Navigate to the labs folder from your IPython Notebook/Jupyter dashboard page

2\.  Open the Notebook named `NLP_Classification`. You should see a set of comments labeled `train_samples`:

![classifying-text-using-naive-bayes-1](https://user-images.githubusercontent.com/21102559/40942567-a776f140-681c-11e8-8b60-d2bae3515c2e.png)

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Import NLTK</h2>

1\.  Add the following two import statements into a new cell:

![classifying-text-using-naive-bayes-2](https://user-images.githubusercontent.com/21102559/40942568-a78daed0-681c-11e8-8c1a-04dcc74ce17c.png)

2\.  Run the cell.

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Define a Function</h2>

1\.  In a new cell, define the following function named `create_dictionary` that takes in a sentence, tokenizes it into words, and creates a dictionary containing each word and assigning it the meaning "True":

![classifying-text-using-naive-bayes-3](https://user-images.githubusercontent.com/21102559/40942569-a7aad2a8-681c-11e8-9854-3510f89aa999.png)

2\.  Run the cell. There will not be any output. 

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"/>
<h2>4. Separate the Comments</h2>

<!-- -->

1\.  In a new cell, create two new arrays:
```
positive_comments = [] 
negative_comments = []
```

2\.  Wrie a `for` loop that iterates through the training data and separates the positive and negative comments:
```
for comments, label in train_samples.items(): 
     if label == 'pos' :
          positive_comments.append(comments.lower()) 
     else :
          negative_comments.append(comments.lower())
```

3\.  Add two `print` statements to display both arrays and run the cell.

4\.  The output should look like:

![classifying-text-using-naive-bayes-4](https://user-images.githubusercontent.com/21102559/40942570-a7c60848-681c-11e8-9fd5-9e9dbcd79ff1.png)

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Define the Features</h2>

1\.  NLP classifiers use a list of features for training. The features need to be in a dictionary, which is why you added the `create_dictionary` function. Add the following line of code into a new cell:

![classifying-text-using-naive-bayes-5](https://user-images.githubusercontent.com/21102559/40942571-a7e3c932-681c-11e8-84c0-156c235d7a18.png)

2\.  Run the cell. The output should look like:

![classifying-text-using-naive-bayes-6](https://user-images.githubusercontent.com/21102559/40942572-a810f3d0-681c-11e8-87bf-b4c46ef7d93a.png)


> Notice each negative comment has been broken down into a dictionary of words followed by the string '**neg**'. For example, the "not a good day." comment becomes:
```
({'not': True, 
     'a': True,
     'good': True, 
     'day': True, 
     '.': True}, 
     'neg')
```

3\.  In a new cell, do the same thing for the positive comments:

![classifying-text-using-naive-bayes-7](https://user-images.githubusercontent.com/21102559/40942573-a82bec76-681c-11e8-8b43-f7c5f236d2ac.png)


d.  Run the cell:

![classifying-text-using-naive-bayes-8](https://user-images.githubusercontent.com/21102559/40942574-a846ff84-681c-11e8-8b7e-04e9caaa2861.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Combine the Training Data</h2>

1\.  Define a new dataset named `training_features` that is the sum of the `positive_features` and `negative_features`:

![classifying-text-using-naive-bayes-9](https://user-images.githubusercontent.com/21102559/40942575-a8625cf2-681c-11e8-91d5-f4ff1af1b4d5.png)

2\. Print the `training_features`. The results should look like:

![classifying-text-using-naive-bayes-10](https://user-images.githubusercontent.com/21102559/40942576-a8ab8f76-681c-11e8-920b-e52dc88132e9.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. Train the Data</h2>

1\.  In a new cell, create a new `NaiveBayesClassifier` based on your training data, then invoke the `show_most_informative_features()` function and run the cell:

![classifying-text-using-naive-bayes-11](https://user-images.githubusercontent.com/21102559/40942577-a8cafdde-681c-11e8-99a7-abe48bcd66e2.png)

The output should look like:

![classifying-text-using-naive-bayes-12](https://user-images.githubusercontent.com/21102559/40942578-a8e97f02-681c-11e8-9b51-4009bb3c4b50.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>8. Classify New Comments</h2>

1\.  Now that your naive Bayes classifier has been training, use it to classify new comments. For example, try the following comment:

![classifying-text-using-naive-bayes-13](https://user-images.githubusercontent.com/21102559/40942579-a9004782-681c-11e8-96c7-d121b944a7a4.png)

> Notice this comment was classified as positive.

2\.  Try a negative comment:

![classifying-text-using-naive-bayes-14](https://user-images.githubusercontent.com/21102559/40942581-a91b4762-681c-11e8-9bb4-7d110f2a6bdc.png)

3\.  Try a comment with words that were not used in the training data:

![classifying-text-using-naive-bayes-15](https://user-images.githubusercontent.com/21102559/40942583-a94a237a-681c-11e8-913d-8c1581c8158b.png)

4\.  Try entering comments with words like "brownies", "coffee", "rainbows" and "Hadoop", since those are words that appeared in your training set.


### Result

You have just used NLTK and the naive Bayes algorithm to classify comments based on a training set of positive and negative comments.

You are finished! 

