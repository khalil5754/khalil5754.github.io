###### Why XGBoost is The Best Machine Learning Tool 

# What is XGBoost?
If, like me, you intend to become a Kaggle grandmaster then you've come to the right place. XGBoost is the undisputed king of the vast majority of Data Science and Machine Learning problems - winning 16 of the 29 published Kaggle challenges in 2015, less than a year after the infamous model was released. Of these 16, 8 teams=used *only* XGBoost to train their  
model, while the other teams used XGBoost with neural nets in ensembles. XGBoost stands for Xtreme Gradient Boosting, a type of gradient boosted decision tree system, and an example of ensemble learning. In case you missed my previous post about decision tree boosting, boosting is a machine learning algorithm that creates a loop wherein a sequence of models is trained, and each model's results including their residuals (errors) are used to train previous models. The final result is thus the combination of all previous models' predictions. 

## What is ensemble learning?

This is a machine learning technique that essentially takes multiple individual (often weak or below-average) models, and aggregates them all into one model to reduce inaccuracy and offer a more powerful singular model. It allows for enhanced model accuracy by reducing variance and bias, but also increases scalability by an impressive amount - especially in the case of XGBoost. Ensemble learning is widely accepted as one of the best machine learning techniques, being used in everything from high-level recommendation algorithms to natural language processing (NLP).
The biggest drawback with ensemble learning is that it can require more resources from your computer due to the inherent complexity involved in combining multiple different models. This means a skilled machine learning engineer must know how to balance their models and achieve efficiency.

## Okay, so if ensemble learning is not specific to XGBoost, what makes XGB special?

Great question. First of all, XGBoost scales to *billions* of examples. It uses a novel tree learning algorithm for optimization and for handling sparse data. This algorithm, in tandem with the exceptionally advanced parallel and distribute computing techniques pre-built into XGBoost, allows for incredibly robust machine learning models capable on running on a simple laptop. 
As opposed to getting predictions by adding up predictions from each component model, we can multiply the predictions from each model by a small number before adding them in. This means each tree we add to the ensemble impacts the model less and less, thus reducing the overfit problem that often comes from ensemble learning. This small number is referred to as the "learning rate". Overfitting is a common problem in machine learning and one which many beginners don't thoroughly know how to avoid. It happens when a model - usually optimized for variance - is giving predictions that are accurate for the training dataset, but not or new data. The ultimate goal is to reduce the residuals (errors) and 
XGBoost combines a small learning rate with a large number of estimators to avoid overfit.

Here's the basic idea of how XGBoost works:

1. Initializing just one decision tree as the beginning of an ensemble. 
2. Trains the singular decision tree on the input data by fitting it to the target variable. 
3. Calculates the errors based on the previous model.
4. Builds a model to predict these errors.
5. Adds the error prediction model to the ensemble of models
6. Calculates the errors basked on the previous models.
7. Rinse, Repeat, Rejoice!

This way, a set of weak models can be used to predict a target variable with staggering accuracy. An important part that has been glossed over is the need for parameters. This is the undetermined part which we need to learn from data. Parameters are usually denoted with the coefficient θ or _theta_. Training a model is a simply finding the best parameters θ that best fit the training data. We'll go over this in more detail in our coding example below.

XGBoost also needs a training loss function _L_, which is a function that compares the target and predicted output values. This gives us an idea of how accurately the model is performing. Usually, the formula for _L_ is simply the equation for the mean square error: 

![image](https://user-images.githubusercontent.com/44441178/221717080-76606744-2087-4c91-b080-6e55f629b470.png)

The root mean squared error (RMSE) is also common for the training loss function. Now that that's out of the way, let's get to work.

Implementing XGBoost is as simple as a pip install and an xgboost import. From there, you can simply tune your hyperparameters by using a randomized grid search to find the optimal hyperparameters given the number of cross-folds you want, and the number of permutations you want per cross-fold. 

I will go over this in more detail in a future post but for now, after just a quick intro - logging off.

- KhalilTheStatsGuy

