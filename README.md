# note for this **rmd** branch only

in EDA file, it only plot top 5

in EDA-part2, it will show all the companmies

# trading volume 

* ref: https://www.investopedia.com/ask/answers/041015/why-trading-volume-important-investors.asp
* ref: DOI:10.1002/9781119204800
* A stock rising on heavy trading volume tells a different story than one climbing on light volume
* Heavy volume often shows strong belief in a price move, while light volume can mean uncertainty. This relationship between price and volume helps investors validate trends and spot potential reversals.
* Trading volume indicates market participation and can confirm trends or signal potential reversals.
* Rising prices with increasing volume often suggest a strong uptrend, while rising prices with declining volume may indicate a weakening trend.
* Volume spikes can precede major price moves and indicate shifts in market sentiment.
* Volume-weighted average price (VWAP) helps traders find optimal buying or selling opportunities based on market activity.
  





# TSA - only show top 5 companies 

* ref: https://www.investopedia.com/terms/t/timeseries.asp

Def: A time series is a data set that tracks a sample over time. It's often used in investment analysis for predicting trends and patterns. Specifically, statistical methods, such as ARIMA, are used in the analysis and forecasting.

More def: A time series is crucial for tracking data points like a security's price over a specific time and at regular intervals. By using time series analysis, investors gain insights into the factors that influence trends over time, whether in the short-term movement of a stock or the long-term shifts in economic indicators. This comprehensive analysis helps investors and analysts understand how assets or economic variables change and correlate with other factors over time.


* gemini:
Moving into Time Series Analysis (TSA) is the natural next step! In financial data science, observing the trajectory of closing prices over time allows us to identify long-term trends, market cycles, and structural breaks.

Since stock prices can have vastly different absolute values (e.g., one company's stock might trade at 15,000 VND while another trades at 120,000 VND), plotting them all on a single Y-axis can squash the lower-priced stocks into a flat line.

 creates two versions of the Price Trend plot:

An overlaid line chart: Great for seeing general market synchronicity.

A faceted line chart: The best approach when price ranges vary wildly, as it gives each stock its own independent Y-axis.

<img width="1152" height="768" alt="image" src="https://github.com/user-attachments/assets/504f57fd-0313-4cb2-be49-578fa2712cea" />


<img width="1152" height="768" alt="image" src="https://github.com/user-attachments/assets/3445e8ac-feb7-4ef5-bf86-bd15ace01912" />


