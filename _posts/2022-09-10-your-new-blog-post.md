# Python for Finance: Variance, Volatility, and Matrix Multiplication

Obviously, Python is a very powerful tool. I have yet to see a problem that I can't tackle using a Python library in some way. Some libraries are more efficient than others, but overall, Python is a versatile tool that can be used to tackle a vast array of coding problems.

Let's look at a couple of financial applications for Python in this post. 

I've put together a 3 company portfolio consisting of SPY (the S&P 500 ETF), OPEN (Opendoor Inc.) - an iBuyer, and CCI (Crown Castle International) - a telecommunications infrastructure giant based in the U.S..

We can start by importing the basic libraries that you will generally always be using when analyzing time-series data, and (as usual) reading in our CSV's which I bet you already know how to do, but in case you don't:

```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import datetime as dt

SPY = pd.read_csv(r"C:\Users\kalka\Downloads\SPY1.csv", parse_dates=["Date"], index_col="Date")
print(SPY.head())

OPEN = pd.read_csv(r"C:\Users\kalka\Downloads\OPEN1.csv", parse_dates=["Date"], index_col="Date")
print(OPEN.head())

CCI = pd.read_csv(r"C:\Users\kalka\Downloads\CCI1.csv", parse_dates=["Date"], index_col="Date")
print(CCI.head())
```

You can tell Python which column is the "date" column that you'd like it to parse by using one of the parameters for reading the CSV into Python: parse_dates, then we tell Python that we want it to index each column by the "date" column as well.

Then, we use the .head() function to print the top 5 lines of each CSV.

Here's what Python returns for our print function:

![image](https://user-images.githubusercontent.com/44441178/197317954-d56c0769-35fc-4340-94ea-5ee67fa4c753.png)


Woah, that's _way_ too many decimal places for SPY and CCI. But we can fix that easily using the round() function, with the parameter (decimals=2) to return only two decimal places. 
We should also merge the individual tables we've created into one, which we can do efficiently in one line. 

```
portfolio = CCI.merge(OPEN, on='Date', suffixes=('_CCI', '_OPEN')).merge(SPY, on='Date', suffixes=('_OPEN', '_SPY'))
prices = portfolio[["Price_CCI", "Price_OPEN", "Price_SPY"]].round(decimals=2)

print(prices.head())
```

Giving each table its own suffix is important because all three tables being merged have a column named the same thing (Price). Python needs a way to distinguish each table's "Price" column from the others. If Python isn't told what suffix to use, it will create its own. Of course with all due respect to our ally Python, we don't want that.

If you've ever used a DBMS (Database Management System) like MySQL you know you normally must give the program a column to merge both tables "on". Otherwise, how will the program know? We've told Python what we want it to merge the tables on; in this case, the 'Date' column.

Here's what Python returns:

![image](https://user-images.githubusercontent.com/44441178/197318137-8d4707c0-3e7f-44f9-ae61-39aac4b5d9dc.png)


Now that our data is more cleaned up, we can create some visualizations.

## Portfolio Returns

Well if you're in finance you know the most important question a client will ask isn't to do with volatility, it's: "how did our portfolio do?".
Let's explore that.

Looking at our returns from February through the end of March, it's easy to figure this out. Lets kill two birds with one stone and plot our portfolio's total returns, as well as how each individual stock did to better understand why our returns are how they are. **It's always better to create too many visualizations than not enough.**

```
weights = [0.3, 0.4 , 0.3]
portfolio_returns = prices.pct_change().dot(weights)

plt.figure(figsize=(16,8), dpi=150)
portfolio_returns.plot(color = 'rebeccapurple')

plt.title('Portfolio Returns')
plt.ylabel('Daily Return, %')
plt.xlabel('Date')
plt.show()


plt.figure(figsize=(16,8), dpi=150)
plt.plot(prices.index, prices['Price_CCI'].pct_change(), label = 'CCI', color = 'forestgreen')
plt.plot(prices.index, prices["Price_SPY"].pct_change(), label = 'SPY', color ='black')
plt.plot(prices.index, prices["Price_OPEN"].pct_change(), label = 'OPEN', color = 'royalblue')
plt.xlabel('Date')
plt.ylabel('Daily Return, %')
plt.legend()
plt.show()
```
The above code block might seem like there's a lot going on but it's actually extremely simple. With the first line we assign the weights of each stock - this is the percentage of our portfolio each stock comprises. My portfolio was allocted as such: 30% SPY, 40% OPEN, and 30% CCI - leading to weights of 0.30, 0.40, 0.30 being assigned, respectively.
We then use the pct_change() function to find the percentage changes from day-to-day, while the dot() function returns the dot product of our arrays. In combination, the first two lines give us our portfolio returns every day, assigned to "portfolio_returns".

Next we build our matplotlib plot using plt.figure() to build the base of the plot, then .plot() to call matplotlib's plot function. We'll go over this in more detail in the next post, where I'll be writing about my **_absolute favourite_** type of visualization in Python: the exploding pie chart. Assigning the colour "rebeccapurple" just because I think it's the best purple there is.

Then, we plot the individual stocks' returns (remember, this is just the pct_change()) over the same period as our total portfolio return, creating a legend in the process for each stock.

This returns two plots:

![image](https://user-images.githubusercontent.com/44441178/197319785-9e16ee83-096b-46fb-ba81-71fe7cc7cde8.png)


![image](https://user-images.githubusercontent.com/44441178/197319780-fc46abf4-5179-4ab3-8661-086cab87e288.png)

Look at that - because of our high allocation of OPEN and the immense volatility in the stock compared to the relative steadiness of the other two picks in the portfolio, our portfolio's returns' shape looks very similar to OPEN's individual returns!


# Volatility and Variance using Matrix Multiplication

Now that we know our portfolio returns, we can find the variance and volatility. If you remember statistics class, first we need the variance, then the square root of the variance will give us the standard deviation, which we can use to find the volatility.

```
covariance = portfolio_returns.cov()

portfolio_variance = np.transpose(weights) @ covariance
portfolio_volatility = np.sqrt(portfolio_variance)
```
The first line of code sets the variable "covariance" equal to the covariance (cov()) of our portfolio returns. With the help of numpy's transpose() function, the portfolio variance matrix is transposed, returning a permuted array. This is when the "@" function comes into play, which may be something you haven't yet seen. The @ denotes matrix multiplication, so our weights matrix is multiplied by our covariance matrix.
The square root of variance gives us standard deviation - which we can use to find the volatility.

Finally, let's get to plotting the volatility:

```
plt.figure(figsize=(16,8), dpi=150)
returns_window = portfolio_returns.rolling(5)
volatility_series = returns_window.std()*np.sqrt(58)
volatility_series.plot(title='Portfolio Volatility', color = 'red').set_ylabel("Annualized Volatility, 30-day Window")

plt.show()
```
We set a rolling window of 5 days for portfolio returns, so volatility will be calculated over a short time period. 
After setting the rolling window on portfolio_returns, we can calculate the volatility by taking the standard deviation of returns_window using std(), then multiply that by the square root of n, where n is the total number of days - in this case, sqrt(57).

Now we can plot the volatility of our portfolio over the course of Febrary to March using a 5-day rolling window by calling the plot() function on volatility_series.

Here's the final result:

![image](https://user-images.githubusercontent.com/44441178/197320721-584ce926-5dd3-4d38-ba6f-702b3e60410e.png)

Look at that, our volatility plot matches up quite well with the plot of our portfolio's returns! Trust me, there's no better feeling than final results that make sense.

And finito!


Thanks for making it to the end of today's post. It certainly wasn't the most interesting one I plan on writing but hey - without the boring ones there can't be interesting ones right?


Until next time,

Khalil (TheStatsGuy)



