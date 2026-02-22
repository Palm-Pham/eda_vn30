# note for this **rmd** branch only


# TSA - only show top 5 companies 

Moving into Time Series Analysis (TSA) is the natural next step! In financial data science, observing the trajectory of closing prices over time allows us to identify long-term trends, market cycles, and structural breaks.

Since stock prices can have vastly different absolute values (e.g., one company's stock might trade at 15,000 VND while another trades at 120,000 VND), plotting them all on a single Y-axis can squash the lower-priced stocks into a flat line.

To handle this, I will provide you with code that creates two versions of the Price Trend plot:

An overlaid line chart: Great for seeing general market synchronicity.

A faceted line chart: The best approach when price ranges vary wildly, as it gives each stock its own independent Y-axis.
