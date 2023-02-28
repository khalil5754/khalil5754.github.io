# Using Python to Build an XGBoost Prediction Model With an AWS Sagemaker Deployment: Penguin Weight Prediction

Okay, now that we've gotten the _what_ and _why_ out of the way, let's focus on the fun one: the _how_. 

## The problem

The first and most important thing to keep in mind when approaching a problem you'd like to solve with machine learning is how you're going to frame the problem. Some models are better equipped for certain problems. Is it really something you need an ML model for, or can you use a pre-made model? For example, if you just need something to read documents and fill in some boxes, there are a multitude of publically available models already trained and re-trained thousands of times on millions of data points. If you're looking for anomaly detection, you may need to use something like kNN, or the _nearest neighbour_ algorithm. If you have a set of categorical values this will need its own accommodations compared to a set of numerical values. In our example, let's look at a combination of both categorical and numerical values, and try and build a simple XGBoost model around it.

After you frame your problem, what are your dependent and independent variables? Are they all relevant? Which variable do you want to predict? These are all important questions to answer before you start working on your model.


## The Penguin Problem

I love penguins, so naturally when I found a dataset built into seaborn called _penguins.csv_ I opened it. Fast. I initially just planned to build a visualization around it, but after seeing the dataset I realized it could be a perfect candidate to test an XGBoost model on. 

First and foremost, installing XGBoost is as simple as:
```pip install XGBoost```

I'd recommend installing in a venv (virtual environment), but if that's not something you're interested in, a simple pip install will suffice. We'll also e using the XGBoost API itself, as opposed to the Scikit API for XGBoost. This is just for simplicity and the two are easily interchangeable.
Let's load the dataset in, import some typical Python libraries, and take a peek at the dataset.

```
import seaborn as sns

import pandas as pd

import numpy as np

import matplotlib.pyplot as plt

penguins = sns.load_dataset("penguins")
penguins.head()

```
Note, we used penguins.head() because this is a 343 row dataset. While usually, you should be looking more deeply into a dataset and conducting some exploratory analysis to validate and ensure it's ready to be worked with, we're lucky enough to be using a built-in dataset that comes baked into Seaborn. Here is what the dataset looks like:

<img width="674" alt="image" src="https://user-images.githubusercontent.com/44441178/221719927-928db570-defb-4b7d-bab4-bb5355cd268d.png">

Okay, we can work with this. First thing's first: XGBoost is not going to like the null values, so let me take this opportunity to show you how to drop rows with null values.

```
penguins.dropna(inplace = True)
penguins
```
<img width="689" alt="Screenshot 2023-02-27 at 7 29 51 PM" src="https://user-images.githubusercontent.com/44441178/221720874-32f84b44-748c-417d-8387-021276fab46b.png">

Much better. Next, let's frame our problem. We want to predict a penguin's weight given its body characteristics, its species, as well as the island it's found on. To do this, split our dataset into our prediction variable and our inputs. Note from the last blog post: variable _x_ is typically the input variable, any _y_ is the prediction variable.

```
from sklearn.model_selection import train_test_split

x, y = penguins.drop('body_mass_g', axis = 1), penguins[['body_mass_g']]
```

Our dataset has some numerical values, and some categorical (string) values. Typically, we would have to deal with this ourselves, but XGBoost has a built-in capability for this. All we need to do is use the cast function .astype() in conjunction with a for loop to set these string values as the category type. 

```
labs = x.select_dtypes(exclude=np.number).columns.tolist()


for value in labs:
    x[value] = x[value].astype('category')
    x.dtypes
```
<img width="429" alt="image" src="https://user-images.githubusercontent.com/44441178/221723277-c82ea9ef-56fe-455b-9c1f-a94976300d86.png">

Looking good so far! 

Alright, now let's do one vital step: first, we'll assign testing and training jobs to the data. Then, we'll use XGBoost's built-in function for dataset storing: DMatrix.

```
import xgboost as xgb

#split the data into training and testing sets
x_train, x_test, y_train, y_test = train_test_split(x, y, random_state=1)


#Create regression matrices

dtrain_reg = xgb.DMatrix(x_train, y_train, enable_categorical = True)

dtest_reg = xgb.DMatrix(x_test, y_test, enable_categorical = True)
```

The enable_categorical parameter is set to True because this allows for Python to automatically encode columns. 

Next, we'll choose a value for the _objective_ parameter, which tells XGBoost what problem we are trying to solve and which training loss formula _L_ we want to use. In doing so, we also set the parameters and the number of times we want XGBoost to cycle the model using the n variable. It's important that we find the perfect number for this, because if it does runs the model too many times it will inevitably end up overfitting - which we really want to avoid. To find this perfect amount don't throw a dart at a board or roll a set of dice. Instead, simply tell XGBoost you want it to stop running once the model stops improving for _x_ number of rounds. This is done using early_stopping_rounds and is among one of the many customizable options in your model.
Another important factor you can customize is the verbose_evals number, which essentially tells XGBoost: Okay, I don't want to see how accurate you are _every_ round (we are doing 10000 rounds, which certainly adds up). The verbose_eval line will set the interval at which XGBoost will update you on it's accurazy, in our example, let's do every 50 rounds.

```
#Define our hyperparameters using a dictionary - we use RMSE as our objective
#If you have a GPU, set tree_method to GPU_hist to enable gpu acceleration
parameters = {"objective": "reg:squarederror", "tree_method": "hist"}
evals = [(dtest_reg, "validation"), (dtrain_reg, "train")]
n = 10000

XGB_model = xgb.train(
    params=parameters,
    dtrain=dtrain_reg,
    num_boost_round=n,
    evals = evals,
    verbose_eval = 50, 

    early_stopping_rounds=20
 ```

This is the boosting round now completed (see how simple XGBoost is?). The model has trained on the dataset we fed it, and has found all the patterns possible. The most important part comes next: measuring the accuracy of its results using unseen data. 



