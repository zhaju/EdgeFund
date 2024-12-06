# EdgeFund

## 0. Outline
In order to beat the market, we form a portfolio with the follwing steps:
1. Filter tickers CSV to eliminate tickers that we aren't allowed to invest in, and convert the remaining legal ones to CAD
2. Calculate the Beta values of each ticker, then rank them
3. Calculate the Expected Percent Returns of each ticker, then rank them
4. Rank the stocks based on highest (normalized Beta times expected return), to reward stocks with high return and low market correlation
5. Calculate weights with a function that rewards stocks with high Beta and punishes low Beta, while remaining in [5%, 15%] of the portfolio
6. Iterate between using the top 12-24 stocks by plotting their historical returns for the past 2 weeks
7. Determine the optimal number of stocks and calculate the optimal weights. Output results to a CSV.

## 1. Filtering

We start by looping through the CSV file to remove tickers which do not meet the following criteria:
- Are not denominated in USD or CAD
- Have an average monthly volume of <100,000 shares from ``10-01-2023`` to ``09-30-2024``
- For the average monthly volume calculation, drop any month with <18 trading days

## 2. Calculating Betas
Beta is a measure of how correlated the value of one security is with the market. We calculate the betas of each stock in our list to determine the relation of their performace with the performance of the market (in this case, the average of the S&P500 and TSX60). Our intention is to select stocks with low correlation to the market, so we can beat the market instead of meeting it. Beta is calculated with the following formulas, where $X$ and $Y$ represent two securities:
### Standard Deviation
$$
\begin{align*}
\sigma_X=\sqrt{\frac{\sum(x_i-\overline{X})^2}{N}}
\end{align*}
$$
### Variance
$$
\begin{align*}
\sigma^2_X=\frac{\sum(x_i-\overline{X})^2}{N}
\end{align*}
$$
### Covariance
$$
\begin{align*}
COV(X,Y)=\frac{\sum(x_i-\overline{X})\times(y_i-\overline{Y})}{N}
\end{align*}
$$
### Beta
$$
\begin{align*}
\beta=\frac{COV(x_i,r_M)}{\sigma^2(r_M)}
\end{align*}
$$

where:
- $COV$ = covariance
- $\sigma$ = variance
- $x_i$ = expected return on an asset $i$
- $r_M$ = average expected rate of return on the market

## 3. Calculating Percentage Returns
Now that we've calculated the correlation of each stock with the market, we don't want to blindly select stocks with a high beta in case their value decreases while the market increases. To ensure that our high-risk stocks also have high returns, we calculate the percent returns of each stock to find which ones have the largest returns.

### Expected returns
$$
\begin{align*}
E(X)=\overline{X}=\frac{\sum x_i}{N}
\end{align*}
$$

where 
- $x_i$ are individual returns of some security $X$
- $N$ is the total number of observations (time periods for us)

## 4. Filtering by Beta and ER
Now that we have our stocks ranked by both risk and reward, we must determine which ones we want for our portfolio. To do this, we first need to normalize our beta values so they're bounded by $[0,1]$ just like our percent returns. To do this, we'll use a sigmoid (or logistic) function, which is commonly used in statistics and machine learning as an activation function. The sigmoid function takes any real number value and scales it so it's between 0 and 1. The sigmoid function on some value $x$ is represented by $\sigma(x)$, and is defined as follows:

$$
\sigma(x)=\frac{1}{1+e^{-x}}
$$

Now, we want to reward stocks with both high risk and high reward. After taking the normalized beta values, we then define a constant that is the product of beta and percent returns. We then rank our list of tickers based on which ones have the highest value of

$$
\sigma(\beta (x_i, r_M))\cdot x_i
$$

where
- $x_i$ = average expected rate of return on a security $x$
- $r_M$ = average expected rate of return on the market

NOTE: there are some stocks with negative percent returns. We are fine with keeping them negative, as we want to beat the market and would thus never want to invest in stocks with negative returns. Those stocks would have a negative product, and fall to the bottom of our rankings.

## 5. Calculating Weights
Given the top $n$ stocks, we calculate how much each one should be weighted in our portfolio while abiding by the bounds of $\frac{100}{2n}$ to $15\%$

We calculate weights by summing our weighted scores, then taking the weighted average of each stock in proportion to the total. This ensures that stocks with high reward and high return are rewarded, while stocks with low risk and low return are penalized.

## 6. Running Simluations
To determine how many stocks we want in our portfolio ($n\in[12,24]$), we iterate through picking the top 12 to top 24 stocks, and project their portfolio values had we bought them two weeks ago (``2024-11-06`` to ``2024-11-20``). This is so we can get an accurate understanding of how these stocks move in a short time period, as the competition also lasts two weeks. We then select the portfolio with the highest historical return.

Limitations:
- We have $\$1,000,000$ CAD to spend
- There is a flat fee of $\min(3.95, 0.001\times\text{shares})$ for every stock we purchase
- We may pick n stocks, where $n\in[12,24]$
- Each stock must make up a minimum of $\frac{100}{2n}\%$ of our portfolio, and a maxmimum of $15\%$.

We will simulate $24-12+1=13$ different portfolios, graph their performance, and select the one with the highest return. Finally, we plot this optimal portfolio against the performance of the ``TSX60`` and ``S&P500`` portfolio to demonstrate that we beat the market.

## 7. Outputting Data
We're almost done! To finish, we purchase our portfolio using the optmial weights. To ensure that our portfolio value ends up as $\$1,000,000$ and our weights sum to $100\%$, we scale our values accordingly. Finally, we output a DataFrame displaying the adjusted Ticker, Price, Currency, Value, Shares, and Weight of our optimal portfolio, and outputting the weights of our optimal portfolio to a CSV :)