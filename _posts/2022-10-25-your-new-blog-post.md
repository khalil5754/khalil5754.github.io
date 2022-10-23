# Granger Causality and Cross-correlation: Determining if the VIX Index is a Reliable Predictor of Foreign Direct Investment in Canada

VIX and FDI Causality Analysis
This section will take a broad approach to analyzing the macroeconomic impact on FDI, instead using the VIX index as the independent variable. The VIX market index was developed by the Chicago Board Options Exchange to track market uncertainty. It measures the market’s annualized implied volatility over the next 30 days. The VIX index’s 52-week high for each year will be used for all calculations because taking the average yearly price of VIX makes it difficult to discern when a significant systemic shock has occurred. VIX is a forward-looking index calculated with a rolling 30-day window, meaning that in years where global shocks occur, VIX can spike and regress to the mean very quickly. It could also see small consistent rises because of S&P 500 companies’ earnings results, domestic interest rate hikes, as well as other microeconomic factors across the stock market. This paper is intended to elucidate the impact of global shocks on FDI, not the impact of general stock market volatility – when there is a large systemic shock, VIX will rise violently to peak levels. If the relationship between VIX’s year high prices and FDI were correlated to a significant degree, that would suggest that the Canadian economy is susceptible to sudden changes in volatility.
It may be difficult to see a relationship in Figure 5 without using statistical analysis techniques to interpret the data due to the inherent time it takes for factors like a global shock or even market uncertainty to ripple through the economy and impact an investment as significant as FDI. To confidently interpret the data, statistical tools are used to analyze linear time-series with a time-lag, such as a Granger causality test.

Granger causality Test
Developed by prominent statistician Clive Granger, a Granger causality test is a statistical analysis method intended to determine if one variable is a reliable predictor of another, even if their effects suffer from a time lag (Granger, 1969). To determine if market volatility is a predictor of Canadian FDI levels, or if the reverse is true, the Granger causality test will be used to compare FDI inflow in Canada to VIX prices. Since the VIX index serves as an indicator for volatility, if FDI is in fact susceptible to volatility, there will be a correlation between the two variables. Therefore, the hypothesis is that VIX serves as a predictor of FDI. The opposite correlation will also be studied to determine if there is a bilateral relationship. The null hypothesis is that there is no evidence that VIX causes FDI. A standard p-value of <0.05 (95%) is required to confidently reject the null hypothesis.

Cross-correlation Analysis
The Granger causality test resulted in extremely low p-values, meaning VIX likely serves as a predictor of FDI. While the test has become widely accepted in the fields of econometrics and statistics, it is still vital to conduct at least another test to ensure there is no case of spurious regression. Spurious regression is a phenomenon wherein two independent variables seem to have a significant correlation but are, in actuality, uncorrelated. 
To avoid spurious regression, we can employ another statistical equation called cross-correlation to confirm our findings. Cross-correlation is a method of causality analysis wherein two series are tracked as relative functions to each other. The equation takes two series as an input and calculates correlation coefficients across the series with shifting data, thus creating lags, and calculating the correlation for each individual lag. The process then produces a third series consisting of the correlation coefficients for each applicable lag, which can be graphed and analyzed on its own (Caltech, 2001). 
After normalization, using the equation 1.96/(√n)  to determine the significant correlation coefficient, the value of significance equates to +/- 0.41, indicated by the range within the black dotted lines (See Figure 6).

Methods
Annual FDI inflow in Canada was compared to VIX levels to determine if VIX causes Canadian FDI inflow, or if the reverse is true. The VIX index over 22 years (1999-2021) was used. 

Granger causality:
The Granger causality equation can be defined by:
V_(t= ) ∑_(y=1)^l▒〖a_y V_(t-y)+ ∑_(y=1)^l▒〖b_t F_(t-y)+∈_t 〗〗 
F_(t= ) ∑_(y=1)^l▒〖c_y V_(t-y)+ ∑_(y=1)^l▒〖d_t V_(t-y)+n_t 〗〗 
Equation 1: Granger causality equation
Where V_t is the first time-series VIX, and F_t is the second time-series FDI. In equation 1, l is the maximum lag used, a_y,b_t,c_y,d_t are coefficients of the model. ∈_t and n_t are two uncorrelated i.i.d. processes. If VIX causes FDI then a_y must be statistically significant, and if FDI causes VIX then it must follow that c_y be statistically significant. If both a_y and c_y  are statistically significant, then there is a bi-directional correlation between the two variables. A maximum time lag of 4 years was used throughout all tests, as any further could not be confidently correlated.
Cross-Correlation:
The cross-correlation equation employed is defined as:
 
Equation 2: Cross-Correlation Equation of a Discrete Finite Series
To determine where cross-correlation values are significant, the equation 1.96/(√n) is used, a critical value of 1.96 is used indicating a 95% confidence interval. 

VIX Formula:
 
Equation 3: Formula for VIX Index
Where T is the future time horizon, and ‖𝜎‖2𝐿2 is the volatility average squared, over the time-period of length T. For the VIX Index, T is 30 days, as it is calculated over a 30-day rolling-window.
![image](https://user-images.githubusercontent.com/44441178/197414297-329df489-60fe-42e2-94b6-7c8082b80d1b.png)
