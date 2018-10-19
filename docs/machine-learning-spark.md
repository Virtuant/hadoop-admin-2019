## Machine Learning with Apache Spark

The hands-on portion for this lab is a Zeppelin notebook that has all the steps necessary to ingest and explore data, train, test, visualize, and save a model. We will cover a basic Linear Regression model that will allow us perform simple predictions on a sample data. This model can be further expanded and modified to fit your needs. Most importantly, by the end of this lab, you will understand how to create an end-to-end pipeline for setting up and training simple models in Spark.

Machine Learning models can be applied to accomplish a variety of tasks from Classification, to Collaborative Filtering, Clustering, and Regression.

We start with Linear Regression as this is one of the most common approaches for estimating unknown parameters after training on a known dataset. In this lab, we are training on a 2D dataset, so our Linear Regression model can be intuitively thought as curve fitting. With more parameters, or features, we can make interesting predictions, for example, what should be a price listing range for a house with three bedrooms, two baths, 20 years old, and in a specific zip code area. 

Using Linear Regression for pricing houses given a set of input parameters works surprisingly well provided a large enough sample dataset. To cover edge cases, however, other Machine Learning methods might have to be used such as Random Forests or Gradient Boosted Trees.

There are five major steps in this lab that we will cover:

- First, the dataset will consist of pairs of inputs and expected outputs. You may think of this as (x, y) pairs, with x being the input and y being the output. We will have approximately twenty points in our small training dataset.
- Second, we will construct a Linear Regression pipeline by specifying the input and output columns, and the model we want to use (i.e. Linear Regression). Once the pipeline is created, we will use the fit function to fit the data to a model (i.e. train the model).
- Third, we will summarize the model results focusing on:
    - Root Mean Square Error (RMSE) as another measure of differences between predicted by model and observed values (taking in account all the differences or residuals) with lower value indicating a better fit.
    - R2 or R Squared, also called coefficient of determination or goodness of fit, with the higher values indicating better fit (on a 0 to 1 scale).
    - Residuals that indicate difference between the observed value and the estimated value of the quantify of interest. With these snapshot measures, a Data Scientist can quickly understand model fitness and compare different trained models to choose the best one.
- Fourth, with the trained model we will make predictions and see how individual predictions compare to original (expected) values.
- Fifth, we will graph the model as a straight line overlaid over the original training data points. This will give you a quick visual snapshot on the fitness of the model.

### What is a Model?

A model is a mathematical formula with a number of parameters that need to be learned from the data. Fitting a model to the data is a process known as model training. Take, for instance one feature/variable linear regression, where a goal is to fit a line (described by the well know equation y = ax + b) to a set of distributed data points.

For example, assume that once model training is complete we get a model equation y = 2x + 5. Then for a set of inputs [1, 0, 7, 2, …] we would get a set of outputs [7, 5, 19, 9, …]. That’s it!

In this notebook you will get a chance to learn a step-by-step process of training a one variable linear regression model with Spark.

### Why Linear Regression?

We’re introducing Machine Learning with Linear Regression because it’s one of the more basic and commonly used predictive analytics method. It’s also easy to explain and grasp intuitively as you’ll make your way through the examples.

>Note that we will not cover the details of how the underlying Linear Regression algorithm works. We will merely focus on applying the algorithm and generating a model.

To summarize, we will be:

1. Setting up a two dimensional dataset
2. Creating a Linear Regression model pipeline
3. Summarizing the model training
4. Predicting output using the model
5. By the end of the notebook you will be able to visualize the linear fitting prediction that your model generated:

![linreg-match-4](https://user-images.githubusercontent.com/558905/47195499-310fb400-d32a-11e8-8f34-dd7d31c09d98.jpg)

### Import a Zeppelin Notebook

Great! now you are familiar with how to train a Linear Regression Machine Learning model and you are ready to Import the Introduction to Machine Learning using Linear Regression notebook into your Zeppelin environment.

To import the notebook, go to the Zeppelin home screen:

1. Click Import note
2. Select Add from URL
3. Copy and paste this [URL](https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/intro-to-machine-learning-with-apache-spark-and-apache-zeppelin/assets/Introduction%20to%20Machine%20Learning%20with%20Apache%20Spark.json) into the Note URL.

4. Click on Import Note - once your notebook is imported, you can open it from the Zeppelin home screen by:

5. Clicking Clicking on the Introduction to Machine Learning using Linear Regression

Once the Introduction to Machine Learning using Linear Regression notebook is up, follow all the directions within the notebook to complete the lab.

### Results

We hope that you’ve been able to successfully run your first lab introducing basic, yet very common, Machine Learning application. Through this lab we learned how to generate and explore data, train, test, visualize, and save a Linear Regression Machine Learning model.
