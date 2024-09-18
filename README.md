# Stock Option Scanner

Python Program that scans through 2000+ Stock Tickers to find stocks with high expected value for option sellers.

---

Basic Option Selling Premium Explanation:

Option sellers primarily make money through the **Variance Risk Premium**. In essence, every option contract has an **Implied Volatility**, based on its strike price, expiration, spot price, risk free interest rate, etc. (IV can be solved for using Black Scholes). A high implied volatility means the market believes the stock will vary wildly in price. This makes it more risky to sell options (since the sold option has a higher chance of expiring in the strike if the stock is highly volatile), which in turn leads to a higher premium for option sellers. The higher the IV, the more money an option seller makes via premium when they sell a contract.

This does **not** mean option sellers should blindly sell contracts with high implied volatility however. While you may be able to rake in high premiums, options that expire in the money can lose you a lot of money (theoretically an endless amount of money, in fact). Instead, option sellers should strive to find opportunities where implied volatility is higher than it should be. In essence, this means the market believes a stock will be highly volatile, but you believe it won't be highly volatile. If your prediction turns out to be right, you will rake in the premium and not have to worry about the stock expiring in the money, since it isn't moving around much. 

While you can try to predict Future Volatility using complex models, you can also find positive Expected Value (EV) opportunities by simply measuring the past. Essentially, by comparing an old option's Implied Volatility over a 30 day period, and comparing it to the option's Realized Volatility (measured by the stock's close prices within the 30 days after the option was written), you can see how **Realized Volatility** stacks up against historical volatility. The difference between a stock's implied volatility and realized volatility is known as the **Variance Risk Premium**, and when it's positive (i.e Realized Volatilty is **less** than implied volatility), the option seller makes money.

---
Volatility Scanner Explanation:

Based on this idea, I have created a program that scans through 2000+ stock tickers to find the stocks with the highest expected value. In this instance, highest expected value means the highest 10-day moving average VRP (meaning the VRP over the last 10 days).

In order to accomplish this, I needed to find a way to get Implied Volatility and Realized Volatility data. For implied volatility, I used the following [Dolt Repository](https://www.dolthub.com/repositories/post-no-preference/options/data/master), which is free to use and contains many years of option data for many tickers. I used MySQL to query the database, extracting volatility history data based on ticker and converting this output to a Pandas DataFrame. For realized volatility, I used Yahoo Finance! to get stock close price data, specifically using the [YFinance Python API](https://pypi.org/project/yfinance/) to get the data in a DataFrame format. My sincerest thanks go to the creators of the Dolt repository and the YFinance API, as this project would not be possible without them.

Once I gather the implied and realized volatility data, I link them based on date (offsetting RV by 30 days, since we want IV30D matched up with RV30D, i.e implied volatility over the next 30 days matched up with the realized volatility over those 30 days) and add a VRP column. This process is done for 2000+ tickers, only examining the last month or so of data (since we want 10-Day Moving Average VRP).

Once all the ticker's 10-Day Average VRP is found, the top 10 tickers are extracted to examine in further detail. Specifically, I run a similar process to before, but look for 4 years of data instead of just a month. By looking at VRP over a long period of time, I can get a much better idea of whether a stock actually has a high expected value or not (since trends over 4 years offer more compelling evidence than trends over 1 month). Along with the VRP calculation, I calculate 4 key metrics -- 4 year Average VRP, 10-day Moving Average VRP, VRP Win Rate (i.e a percent indicating how many times the VRP calculation was positive), and IV Rank (i.e how high IV is compared to how high it usually is).

I also used MatPlotLib to graph VRP over a 4-year period for the top 10 tickers, adding the aforementioned 4 metrics onto the graph to display more useful information. The area under the graph is green when VRP is positive and red when VRP is negative, clearly indicating periods of profit and loss. Additionally, the moving average VRP boxes are colored green if they have a value > 0 and red if they have a value < 0, the VRP Win Rate box is colored green if > 50% and red if < 50%, and the IV Rank is colored green if > 80 and red if < 80.

An example graph (for the ticker EMBC) can be seen below:

![image](https://github.com/user-attachments/assets/ef6fa1d0-e781-4a54-a43f-fd9250ee5fc8)

An option seller can use a graph like this to decide whether the stock is worth selling or not, based on the VRP chart and the key metrics.

---
File Breakdown:

optionCalc.py: Contains all the functions that power the volatility scanner. I have not uploaded this file (I can't give all the secrets away :) )
volatilityScanner.ipynb: iPynb notebook that uses optionCalc to grab the top 10 tickers and display the graphs.
