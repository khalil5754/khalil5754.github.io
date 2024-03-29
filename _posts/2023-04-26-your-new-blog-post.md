# Using Python to Build an XGBoost Prediction Model: Penguin Weight Prediction


## The problem with problems

The first and most important thing to keep in mind when approaching a problem you'd like to solve with machine learning is how you're going to frame the problem. Some models are better equipped for certain problems. Is it really something you need an ML model for, or can you use a pre-made model? For example, if you just need something to read documents and fill in some boxes, there are a multitude of publically available models already trained and re-trained thousands of times on millions of data points. If you're looking for anomaly detection, you may need to use something like kNN, or the _nearest neighbour_ algorithm. If you have a set of categorical values this will need its own accommodations compared to a set of numerical values. Tuning your hyperparameters (parameters external to your model) is a direct result of how you frame your problem and will have a pronounced impact on the results of the model. In our example, let's look at a combination of both categorical and numerical values, and try and build a simple XGBoost model around it.

After you frame your problem, what are your dependent and independent variables? Are they all relevant? Which variable do you want to predict? These are all important questions to answer before you start working on your model.


## The Penguin Problem

I love penguins, so naturally when I found a dataset built into seaborn called _penguins.csv_ I opened it. Fast. I initially just planned to build a visualization around it, but after seeing the dataset I realized it could be a perfect candidate to test an XGBoost model on. 

First and foremost, installing XGBoost is as simple as:

```pip install XGBoost```

I'd recommend installing in a venv (virtual environment), but if that's not something you're interested in, a simple pip install will suffice. We'll also be using the XGBoost API itself, as opposed to the Scikit API for XGBoost. This is just for simplicity and the two are easily interchangeable.
Let's load the dataset in, import some typical Python libraries, and take a peek at the dataset.

```
import seaborn as sns

import pandas as pd

import numpy as np

import matplotlib.pyplot as plt

penguins = sns.load_dataset("penguins")
penguins.head()
```

Note we used penguins.head() because this is a 343 row dataset. While usually you should be looking more deeply into a dataset and conducting some exploratory analysis to validate and ensure it's ready to be worked with, we're lucky enough to be using a built-in dataset that comes baked into Seaborn. Here is what the dataset's first five rows look like:

<img width="574" alt="image" src="https://user-images.githubusercontent.com/44441178/221719927-928db570-defb-4b7d-bab4-bb5355cd268d.png">

Okay, we can work with this. First thing's first: XGBoost is not going to like the null values, so let me take this opportunity to quickly show you how to drop rows with null values.

```
penguins.dropna(inplace = True)
penguins
```
<img width="589" alt="Screenshot 2023-02-27 at 7 29 51 PM" src="https://user-images.githubusercontent.com/44441178/221720874-32f84b44-748c-417d-8387-021276fab46b.png">

Much better. Next, let's frame our problem. We want to predict a penguin's weight given its body characteristics, its species, as well as the island it's found on. To do this, split our dataset into our prediction variable and our inputs. Note that variable _x_ is typically the input variable, and _y_ is the prediction variable.

```
from sklearn.model_selection import train_test_split

#Split x variable to inputs and y to prediction
x, y = penguins.drop('body_mass_g', axis = 1), penguins[['body_mass_g']]
```

Our dataset has some numerical values, and some categorical (string) values. Typically, we would have to deal with this ourselves, but XGBoost has a built-in capability for this. All we need to do is use the cast function .astype() in conjunction with a for loop to set these string values as the category type. 

```
labs = x.select_dtypes(exclude=np.number).columns.tolist()


for value in labs:
    x[value] = x[value].astype('category')
    x.dtypes
```

<img width="269" alt="image" src="https://user-images.githubusercontent.com/44441178/221723277-c82ea9ef-56fe-455b-9c1f-a94976300d86.png">

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

Another important factor you can customize is the verbose_evals number, which essentially tells XGBoost: Okay, I don't want to see how accurate you are _every_ round (we are doing 10000 rounds, which certainly adds up). The verbose_eval line will set the interval at which XGBoost will update you on it's accuracy, in our example, let's do every 50 rounds.


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

<img width="502" alt="Screenshot 2023-02-28 at 12 50 02 AM" src="https://user-images.githubusercontent.com/44441178/221765739-98ef85b7-a796-45cb-ba09-cc4951330c1c.png">

As you can see, the accuracy of our model starts off abysmal at a staggering 3033 root mean squared error, but quickly reaches its optimal 342.93. However, we aren't done. We can use cross-validation, or XGBoost's .cv() function to figure out the _best_ RMSE our model can produce. 

In the next code block, it'll all come together and you'll see just how accurate we can get. 
This is the boosting round now completed (see how simple XGBoost is?). The model has trained on the dataset we fed it, and has found all the patterns possible. The most important part comes next: measuring the accuracy of its results using unseen data. 



```
from sklearn.metrics import mean_squared_error

preds = model.predict(dtest_reg)

rmse = mean_squared_error(y_test, preds, squared=False)

print(f"The base RMSE of our model is {rmse:.2f}")

results = xgb.cv(
    params, dtrain_reg,
    num_boost_round=n,
    nfold=5,
    early_stopping_rounds=20
)


accurate_rmse = results['test-rmse-mean'].min()

accurate_rmse

The base RMSE of our model is 342.93

333.0457361916397
```

This one time, let's go through the above code line-by-line.

The first line imports the mean_squared_error function from the scikit-learn library, which will be used to calculate the RMSE.

The second line uses the trained XGBoost model which we've appropriately named "model" to predict the target variable on the test set "dtest_reg".

Then, we've asked XGBoost to calculate the RMSE of the predictions by comparing them to the true target variable values y_test using the mean_squared_error function. The "squared=False" argument tells the function that we want to compute the RMSE, rather than the mean squared error. 

The next block of code uses the XGBoost "cv" function to perform cross-validation and select the best set of hyperparameters for the model. The "params" argument specifies the hyperparameters to be optimized, and "dtrain_reg" is the training data.

The "results" variable stores the cross-validation results, including the mean RMSE and standard deviation across all folds for each boosting round.

The last line of code extracts the best RMSE value from the cross-validation results and stores it in the "accurate_rmse" variable which we then display. We get an impressive final RMSE value of 333.046.


## How do I know if this is a good RMSE value? Does this mean the model is accurate?

Typically, we want an RMSE as close to 0 as possible, without an overfitting. Obviously 333.046 isn't close to zero, but as a general guideline, an RMSE value of less than 10% of the range of the target variable (in this case, the weight of the penguin) is considered to be a good performance for a model. The range of our penguin weights is vast at 3600 grams from the lightest penguin to the heaviest. Since the range of our penguins' weights is 3600 grams, an RMSE of less than 360 grams would be considered good performance. Therefore, we've achieved our goal. 

Note, this model could be vastly improved by tuning our hyperparameters more specifically, using more performance metrics to check model performance, having a far larger dataset than just 343 penguins, or even just examining the residuals more meticulously. With that being said, for an out-of-the-box working solution, an accuracy within 10% with so little performance load is staggering!


Thanks for reading and until next time,

Khalil (TheStatsGuy)
