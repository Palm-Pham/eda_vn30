# Exploratory Data Analysis (EDA) Guideline
## VN30 Stock Prices & Volume Dataset (Vietnam)
### Using R Programming

---

## Table of Contents
1. [Setup and Data Loading](#1-setup-and-data-loading)
2. [Initial Data Inspection](#2-initial-data-inspection)
3. [Data Cleaning and Preprocessing](#3-data-cleaning-and-preprocessing)
4. [Univariate Analysis](#4-univariate-analysis)
5. [Time Series Analysis](#5-time-series-analysis)
6. [Multivariate Analysis](#6-multivariate-analysis)
7. [Comparative Analysis Across Stocks](#7-comparative-analysis-across-stocks)
8. [Advanced Visualizations](#8-advanced-visualizations)
9. [Statistical Summary and Insights](#9-statistical-summary-and-insights)

---

## 1. Setup and Data Loading

### 1.1 Install and Load Required Packages

```r
# Install packages (run once)
install.packages(c("tidyverse", "lubridate", "zoo", "quantmod", 
                   "PerformanceAnalytics", "corrplot", "plotly", 
                   "gridExtra", "scales", "TTR", "tseries"))

# Load libraries
library(tidyverse)      # Data manipulation and visualization
library(lubridate)      # Date handling
library(zoo)            # Time series
library(quantmod)       # Financial analysis
library(PerformanceAnalytics)  # Financial performance metrics
library(corrplot)       # Correlation plots
library(plotly)         # Interactive plots
library(gridExtra)      # Multiple plots
library(scales)         # Scale functions for visualization
library(TTR)            # Technical trading rules
library(tseries)        # Time series analysis
```

### 1.2 Load All CSV Files

```r
# Set working directory
setwd("path/to/your/dataset")

# Get list of all CSV files
csv_files <- list.files(pattern = "*.csv")

# Load all files into a list
stock_list <- lapply(csv_files, function(file) {
  df <- read.csv(file, stringsAsFactors = FALSE)
  df$Company <- tools::file_path_sans_ext(file)  # Add company name
  return(df)
})

# Combine all stocks into one dataframe
all_stocks <- bind_rows(stock_list)

# View the structure
str(all_stocks)
head(all_stocks)
```

### 1.3 Load Individual Stock for Detailed Analysis

```r
# Example: Load one stock for detailed analysis
stock_sample <- read.csv("company_name.csv")
```

---

## 2. Initial Data Inspection

### 2.1 Basic Information

```r
# Dimensions
dim(all_stocks)
dim(stock_sample)

# Column names
colnames(all_stocks)

# Data types
str(all_stocks)

# First and last rows
head(all_stocks, 10)
tail(all_stocks, 10)

# Summary statistics
summary(all_stocks)

# Unique companies
unique(all_stocks$Company)
length(unique(all_stocks$Company))
```

### 2.2 Check for Missing Values

```r
# Missing values per column
colSums(is.na(all_stocks))

# Percentage of missing values
missing_pct <- colMeans(is.na(all_stocks)) * 100
print(missing_pct)

# Visualize missing values
library(naniar)
vis_miss(all_stocks)

# Missing values by company
all_stocks %>%
  group_by(Company) %>%
  summarise(
    total_rows = n(),
    missing_close = sum(is.na(Close)),
    missing_volume = sum(is.na(Volume))
  )
```

### 2.3 Check for Duplicates

```r
# Check for duplicate rows
duplicates <- all_stocks %>%
  group_by(Company, Date) %>%
  filter(n() > 1)

nrow(duplicates)

# Remove duplicates if any
all_stocks <- all_stocks %>%
  distinct(Company, Date, .keep_all = TRUE)
```

---

## 3. Data Cleaning and Preprocessing

### 3.1 Date Conversion

```r
# Convert Date column to Date type
all_stocks$Date <- as.Date(all_stocks$Date, format = "%Y-%m-%d")

# Alternative formats if needed
# all_stocks$Date <- dmy(all_stocks$Date)  # for DD/MM/YYYY
# all_stocks$Date <- mdy(all_stocks$Date)  # for MM/DD/YYYY

# Sort by company and date
all_stocks <- all_stocks %>%
  arrange(Company, Date)

# Check date range
range(all_stocks$Date, na.rm = TRUE)
```

### 3.2 Handle Missing Values

```r
# Option 1: Remove rows with missing values
all_stocks_clean <- all_stocks %>%
  na.omit()

# Option 2: Forward fill missing values (for time series)
all_stocks <- all_stocks %>%
  group_by(Company) %>%
  arrange(Date) %>%
  fill(Open, High, Low, Close, Volume, .direction = "down") %>%
  ungroup()

# Option 3: Interpolation for numeric columns
all_stocks <- all_stocks %>%
  group_by(Company) %>%
  mutate(across(c(Open, High, Low, Close, Volume), 
                ~na.approx(., na.rm = FALSE, rule = 2))) %>%
  ungroup()
```

### 3.3 Create Additional Features

```r
# Daily returns (percentage change)
all_stocks <- all_stocks %>%
  group_by(Company) %>%
  arrange(Date) %>%
  mutate(
    Daily_Return = (Close - lag(Close)) / lag(Close) * 100,
    Log_Return = log(Close / lag(Close)),
    Price_Change = Close - lag(Close),
    Volume_Change = Volume - lag(Volume),
    High_Low_Range = High - Low,
    High_Low_Pct = (High - Low) / Low * 100
  ) %>%
  ungroup()

# Moving averages
all_stocks <- all_stocks %>%
  group_by(Company) %>%
  arrange(Date) %>%
  mutate(
    MA_7 = rollmean(Close, k = 7, fill = NA, align = "right"),
    MA_30 = rollmean(Close, k = 30, fill = NA, align = "right"),
    MA_90 = rollmean(Close, k = 90, fill = NA, align = "right")
  ) %>%
  ungroup()

# Volatility (rolling standard deviation)
all_stocks <- all_stocks %>%
  group_by(Company) %>%
  arrange(Date) %>%
  mutate(
    Volatility_30 = rollapply(Daily_Return, width = 30, 
                               FUN = sd, fill = NA, align = "right")
  ) %>%
  ungroup()

# Time-based features
all_stocks <- all_stocks %>%
  mutate(
    Year = year(Date),
    Month = month(Date, label = TRUE),
    Quarter = quarter(Date),
    DayOfWeek = wday(Date, label = TRUE),
    WeekOfYear = week(Date)
  )
```

---

## 4. Univariate Analysis

### 4.1 Distribution of Closing Prices

```r
# Histogram
ggplot(all_stocks, aes(x = Close)) +
  geom_histogram(bins = 50, fill = "steelblue", color = "black") +
  facet_wrap(~Company, scales = "free") +
  labs(title = "Distribution of Closing Prices by Company",
       x = "Closing Price", y = "Frequency") +
  theme_minimal()

# Density plot
ggplot(all_stocks, aes(x = Close, fill = Company)) +
  geom_density(alpha = 0.5) +
  labs(title = "Density Plot of Closing Prices",
       x = "Closing Price", y = "Density") +
  theme_minimal()

# Box plot
ggplot(all_stocks, aes(x = Company, y = Close, fill = Company)) +
  geom_boxplot() +
  labs(title = "Box Plot of Closing Prices by Company",
       x = "Company", y = "Closing Price") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1),
        legend.position = "none")
```

### 4.2 Volume Analysis

```r
# Volume distribution
ggplot(all_stocks, aes(x = Volume)) +
  geom_histogram(bins = 50, fill = "coral", color = "black") +
  facet_wrap(~Company, scales = "free") +
  labs(title = "Distribution of Trading Volume by Company",
       x = "Volume", y = "Frequency") +
  theme_minimal()

# Average volume by company
avg_volume <- all_stocks %>%
  group_by(Company) %>%
  summarise(Avg_Volume = mean(Volume, na.rm = TRUE)) %>%
  arrange(desc(Avg_Volume))

ggplot(avg_volume, aes(x = reorder(Company, Avg_Volume), y = Avg_Volume, fill = Company)) +
  geom_col() +
  coord_flip() +
  labs(title = "Average Trading Volume by Company",
       x = "Company", y = "Average Volume") +
  theme_minimal() +
  theme(legend.position = "none")
```

### 4.3 Daily Returns Distribution

```r
# Histogram of returns
ggplot(all_stocks %>% filter(!is.na(Daily_Return)), 
       aes(x = Daily_Return)) +
  geom_histogram(bins = 100, fill = "darkgreen", color = "black") +
  labs(title = "Distribution of Daily Returns",
       x = "Daily Return (%)", y = "Frequency") +
  theme_minimal()

# QQ plot to check normality
qqnorm(all_stocks$Daily_Return[!is.na(all_stocks$Daily_Return)])
qqline(all_stocks$Daily_Return[!is.na(all_stocks$Daily_Return)], col = "red")

# Statistical tests for normality
shapiro.test(sample(all_stocks$Daily_Return[!is.na(all_stocks$Daily_Return)], 5000))
```

### 4.4 Summary Statistics

```r
# Comprehensive summary by company
summary_stats <- all_stocks %>%
  group_by(Company) %>%
  summarise(
    N_Days = n(),
    Min_Price = min(Close, na.rm = TRUE),
    Max_Price = max(Close, na.rm = TRUE),
    Mean_Price = mean(Close, na.rm = TRUE),
    Median_Price = median(Close, na.rm = TRUE),
    SD_Price = sd(Close, na.rm = TRUE),
    CV_Price = sd(Close, na.rm = TRUE) / mean(Close, na.rm = TRUE),
    Mean_Volume = mean(Volume, na.rm = TRUE),
    Mean_Return = mean(Daily_Return, na.rm = TRUE),
    SD_Return = sd(Daily_Return, na.rm = TRUE),
    Min_Return = min(Daily_Return, na.rm = TRUE),
    Max_Return = max(Daily_Return, na.rm = TRUE)
  )

print(summary_stats)

# Export summary
write.csv(summary_stats, "summary_statistics.csv", row.names = FALSE)
```

---

## 5. Time Series Analysis

### 5.1 Price Trends Over Time

```r
# All stocks price movement
ggplot(all_stocks, aes(x = Date, y = Close, color = Company)) +
  geom_line(alpha = 0.7) +
  labs(title = "Stock Price Trends - All Companies",
       x = "Date", y = "Closing Price") +
  theme_minimal() +
  theme(legend.position = "bottom")

# Individual stock with moving averages
stock_sample %>%
  ggplot(aes(x = Date)) +
  geom_line(aes(y = Close, color = "Actual Price")) +
  geom_line(aes(y = MA_7, color = "MA 7")) +
  geom_line(aes(y = MA_30, color = "MA 30")) +
  geom_line(aes(y = MA_90, color = "MA 90")) +
  labs(title = "Stock Price with Moving Averages",
       x = "Date", y = "Price", color = "Legend") +
  theme_minimal()

# Faceted time series
ggplot(all_stocks, aes(x = Date, y = Close)) +
  geom_line(color = "steelblue") +
  facet_wrap(~Company, scales = "free_y", ncol = 5) +
  labs(title = "Individual Stock Price Trends",
       x = "Date", y = "Closing Price") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

### 5.2 Volume Trends

```r
# Volume over time
ggplot(all_stocks, aes(x = Date, y = Volume)) +
  geom_line(color = "coral") +
  facet_wrap(~Company, scales = "free_y", ncol = 5) +
  labs(title = "Trading Volume Trends by Company",
       x = "Date", y = "Volume") +
  theme_minimal()

# Price vs Volume
ggplot(all_stocks, aes(x = Date)) +
  geom_line(aes(y = Close, color = "Price")) +
  geom_bar(aes(y = Volume / max(Volume, na.rm = TRUE) * max(Close, na.rm = TRUE)), 
           stat = "identity", alpha = 0.3, fill = "gray") +
  facet_wrap(~Company, scales = "free_y", ncol = 3) +
  labs(title = "Price and Volume Trends",
       x = "Date", y = "Value") +
  theme_minimal()
```

### 5.3 Seasonality Analysis

```r
# Monthly average returns
monthly_returns <- all_stocks %>%
  group_by(Company, Month) %>%
  summarise(Avg_Return = mean(Daily_Return, na.rm = TRUE), .groups = "drop")

ggplot(monthly_returns, aes(x = Month, y = Avg_Return, fill = Company)) +
  geom_col(position = "dodge") +
  labs(title = "Average Returns by Month",
       x = "Month", y = "Average Daily Return (%)") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

# Day of week effect
dow_returns <- all_stocks %>%
  group_by(DayOfWeek) %>%
  summarise(Avg_Return = mean(Daily_Return, na.rm = TRUE),
            Median_Return = median(Daily_Return, na.rm = TRUE))

ggplot(dow_returns, aes(x = DayOfWeek, y = Avg_Return)) +
  geom_col(fill = "steelblue") +
  labs(title = "Average Returns by Day of Week",
       x = "Day of Week", y = "Average Return (%)") +
  theme_minimal()

# Yearly performance
yearly_perf <- all_stocks %>%
  group_by(Company, Year) %>%
  summarise(
    Yearly_Return = (last(Close) - first(Close)) / first(Close) * 100,
    .groups = "drop"
  )

ggplot(yearly_perf, aes(x = Year, y = Yearly_Return, color = Company, group = Company)) +
  geom_line(size = 1) +
  geom_point(size = 2) +
  labs(title = "Yearly Returns by Company",
       x = "Year", y = "Yearly Return (%)") +
  theme_minimal()
```

### 5.4 Volatility Analysis

```r
# Rolling volatility
ggplot(all_stocks %>% filter(!is.na(Volatility_30)), 
       aes(x = Date, y = Volatility_30, color = Company)) +
  geom_line(alpha = 0.7) +
  labs(title = "30-Day Rolling Volatility",
       x = "Date", y = "Volatility (%)") +
  theme_minimal()

# Volatility ranking
vol_ranking <- all_stocks %>%
  group_by(Company) %>%
  summarise(Avg_Volatility = mean(Volatility_30, na.rm = TRUE)) %>%
  arrange(desc(Avg_Volatility))

ggplot(vol_ranking, aes(x = reorder(Company, Avg_Volatility), 
                        y = Avg_Volatility, fill = Company)) +
  geom_col() +
  coord_flip() +
  labs(title = "Average Volatility by Company",
       x = "Company", y = "Average 30-Day Volatility (%)") +
  theme_minimal() +
  theme(legend.position = "none")
```

---

## 6. Multivariate Analysis

### 6.1 Correlation Analysis

```r
# Select one stock for correlation analysis
stock_numeric <- stock_sample %>%
  select(Open, High, Low, Close, Volume, Daily_Return, High_Low_Range) %>%
  na.omit()

# Correlation matrix
cor_matrix <- cor(stock_numeric)
print(cor_matrix)

# Correlation plot
corrplot(cor_matrix, method = "color", type = "upper", 
         addCoef.col = "black", tl.col = "black", tl.srt = 45,
         title = "Correlation Matrix of Stock Variables")

# Pairwise scatter plots
pairs(stock_numeric[, 1:5], pch = 19, col = rgb(0, 0, 1, 0.3),
      main = "Pairwise Relationships")
```

### 6.2 Price Correlation Between Stocks

```r
# Create wide format for closing prices
price_wide <- all_stocks %>%
  select(Date, Company, Close) %>%
  pivot_wider(names_from = Company, values_from = Close)

# Calculate correlation
price_cor <- cor(price_wide[, -1], use = "pairwise.complete.obs")

# Correlation heatmap
corrplot(price_cor, method = "color", type = "upper",
         tl.col = "black", tl.srt = 45, tl.cex = 0.7,
         title = "Correlation Between Stock Prices")

# Find highly correlated pairs
highly_correlated <- which(abs(price_cor) > 0.8 & price_cor != 1, arr.ind = TRUE)
```

### 6.3 Return Correlation

```r
# Returns correlation
return_wide <- all_stocks %>%
  select(Date, Company, Daily_Return) %>%
  pivot_wider(names_from = Company, values_from = Daily_Return)

return_cor <- cor(return_wide[, -1], use = "pairwise.complete.obs")

corrplot(return_cor, method = "color", type = "upper",
         tl.col = "black", tl.srt = 45, tl.cex = 0.7,
         title = "Correlation Between Daily Returns")
```

### 6.4 Price vs Volume Relationship

```r
# Scatter plot with regression
ggplot(all_stocks, aes(x = Volume, y = Close)) +
  geom_point(alpha = 0.3, color = "steelblue") +
  geom_smooth(method = "lm", color = "red", se = TRUE) +
  facet_wrap(~Company, scales = "free") +
  labs(title = "Price vs Volume Relationship",
       x = "Trading Volume", y = "Closing Price") +
  theme_minimal()

# Calculate correlation by company
price_volume_cor <- all_stocks %>%
  group_by(Company) %>%
  summarise(Correlation = cor(Close, Volume, use = "complete.obs")) %>%
  arrange(desc(abs(Correlation)))

print(price_volume_cor)
```

---

## 7. Comparative Analysis Across Stocks

### 7.1 Performance Comparison

```r
# Normalize prices to compare performance
normalized_prices <- all_stocks %>%
  group_by(Company) %>%
  arrange(Date) %>%
  mutate(Normalized_Price = Close / first(Close) * 100) %>%
  ungroup()

ggplot(normalized_prices, aes(x = Date, y = Normalized_Price, color = Company)) +
  geom_line(alpha = 0.7) +
  labs(title = "Normalized Stock Performance (Base = 100)",
       x = "Date", y = "Normalized Price") +
  theme_minimal() +
  theme(legend.position = "bottom")
```

### 7.2 Risk-Return Profile

```r
# Calculate risk and return by company
risk_return <- all_stocks %>%
  group_by(Company) %>%
  summarise(
    Avg_Return = mean(Daily_Return, na.rm = TRUE),
    Volatility = sd(Daily_Return, na.rm = TRUE),
    Sharpe_Ratio = mean(Daily_Return, na.rm = TRUE) / sd(Daily_Return, na.rm = TRUE)
  )

# Risk-return scatter plot
ggplot(risk_return, aes(x = Volatility, y = Avg_Return, label = Company)) +
  geom_point(size = 4, color = "steelblue") +
  geom_text(hjust = -0.1, vjust = 0.5, size = 3) +
  geom_hline(yintercept = 0, linetype = "dashed", color = "red") +
  labs(title = "Risk-Return Profile of VN30 Stocks",
       x = "Risk (Volatility)", y = "Average Daily Return (%)") +
  theme_minimal()
```

### 7.3 Best and Worst Performers

```r
# Total return calculation
total_returns <- all_stocks %>%
  group_by(Company) %>%
  arrange(Date) %>%
  summarise(
    Start_Price = first(Close),
    End_Price = last(Close),
    Total_Return = (End_Price - Start_Price) / Start_Price * 100,
    Trading_Days = n()
  ) %>%
  arrange(desc(Total_Return))

# Bar plot
ggplot(total_returns, aes(x = reorder(Company, Total_Return), 
                          y = Total_Return, fill = Total_Return > 0)) +
  geom_col() +
  coord_flip() +
  scale_fill_manual(values = c("red", "green"), guide = FALSE) +
  labs(title = "Total Returns by Company",
       x = "Company", y = "Total Return (%)") +
  theme_minimal()

# Top 5 and Bottom 5
cat("Top 5 Performers:\n")
print(head(total_returns, 5))
cat("\nBottom 5 Performers:\n")
print(tail(total_returns, 5))
```

### 7.4 Liquidity Analysis

```r
# Average daily turnover
liquidity <- all_stocks %>%
  mutate(Turnover = Close * Volume) %>%
  group_by(Company) %>%
  summarise(
    Avg_Turnover = mean(Turnover, na.rm = TRUE),
    Avg_Volume = mean(Volume, na.rm = TRUE)
  ) %>%
  arrange(desc(Avg_Turnover))

ggplot(liquidity, aes(x = reorder(Company, Avg_Turnover), 
                      y = Avg_Turnover, fill = Company)) +
  geom_col() +
  coord_flip() +
  scale_y_continuous(labels = scales::comma) +
  labs(title = "Average Daily Turnover by Company",
       x = "Company", y = "Average Turnover") +
  theme_minimal() +
  theme(legend.position = "none")
```

---

## 8. Advanced Visualizations

### 8.1 Candlestick Chart

```r
# Candlestick chart for one stock (last 90 days)
library(quantmod)

recent_data <- stock_sample %>%
  arrange(desc(Date)) %>%
  head(90) %>%
  arrange(Date)

# Create xts object
stock_xts <- xts(recent_data[, c("Open", "High", "Low", "Close", "Volume")],
                 order.by = recent_data$Date)

chartSeries(stock_xts, type = "candlesticks", 
            theme = chartTheme("white"),
            name = "Stock Candlestick Chart - Last 90 Days")
```

### 8.2 Interactive Plot with Plotly

```r
# Interactive price chart
p <- plot_ly(stock_sample, x = ~Date, y = ~Close, 
             type = 'scatter', mode = 'lines',
             name = 'Close Price',
             line = list(color = 'steelblue')) %>%
  add_trace(y = ~MA_30, name = 'MA 30', 
            line = list(color = 'red', dash = 'dash')) %>%
  layout(title = "Interactive Stock Price Chart",
         xaxis = list(title = "Date"),
         yaxis = list(title = "Price"),
         hovermode = "x unified")

p

# Save interactive plot
# htmlwidgets::saveWidget(p, "interactive_chart.html")
```

### 8.3 Heatmap of Daily Returns

```r
# Monthly heatmap
monthly_heatmap <- all_stocks %>%
  filter(Company == "COMPANY_NAME") %>%  # Replace with actual company
  mutate(Year_Month = floor_date(Date, "month")) %>%
  group_by(Year_Month) %>%
  summarise(Monthly_Return = (last(Close) - first(Close)) / first(Close) * 100)

ggplot(monthly_heatmap, aes(x = month(Year_Month, label = TRUE), 
                            y = year(Year_Month), fill = Monthly_Return)) +
  geom_tile(color = "white") +
  scale_fill_gradient2(low = "red", mid = "white", high = "green", 
                       midpoint = 0) +
  labs(title = "Monthly Returns Heatmap",
       x = "Month", y = "Year", fill = "Return (%)") +
  theme_minimal()
```

### 8.4 Distribution Comparison

```r
# Violin plot
ggplot(all_stocks, aes(x = Company, y = Daily_Return, fill = Company)) +
  geom_violin(trim = FALSE, alpha = 0.7) +
  geom_boxplot(width = 0.1, alpha = 0.3) +
  coord_flip() +
  labs(title = "Distribution of Daily Returns by Company",
       x = "Company", y = "Daily Return (%)") +
  theme_minimal() +
  theme(legend.position = "none")
```

---

## 9. Statistical Summary and Insights

### 9.1 Time Series Decomposition

```r
# For one stock
library(stats)

# Create time series object
stock_ts <- ts(stock_sample$Close, 
               frequency = 252)  # 252 trading days per year

# Decompose
decomp <- decompose(stock_ts, type = "additive")
plot(decomp)
```

### 9.2 Stationarity Test

```r
# Augmented Dickey-Fuller test
library(tseries)

adf_test <- adf.test(stock_sample$Close, alternative = "stationary")
print(adf_test)

# Test on returns
adf_test_returns <- adf.test(na.omit(stock_sample$Daily_Return), 
                              alternative = "stationary")
print(adf_test_returns)
```

### 9.3 Outlier Detection

```r
# Identify outliers in returns
outliers <- all_stocks %>%
  group_by(Company) %>%
  mutate(
    Mean_Return = mean(Daily_Return, na.rm = TRUE),
    SD_Return = sd(Daily_Return, na.rm = TRUE),
    Z_Score = (Daily_Return - Mean_Return) / SD_Return,
    Is_Outlier = abs(Z_Score) > 3
  ) %>%
  filter(Is_Outlier) %>%
  select(Company, Date, Close, Daily_Return, Z_Score)

# Count outliers by company
outlier_summary <- outliers %>%
  group_by(Company) %>%
  summarise(Outlier_Count = n()) %>%
  arrange(desc(Outlier_Count))

print(outlier_summary)
```

### 9.4 Final Report Summary

```r
# Create comprehensive summary report
final_report <- list(
  Dataset_Info = data.frame(
    Total_Stocks = length(unique(all_stocks$Company)),
    Total_Observations = nrow(all_stocks),
    Date_Range = paste(min(all_stocks$Date), "to", max(all_stocks$Date)),
    Missing_Values = sum(is.na(all_stocks))
  ),
  
  Performance_Summary = total_returns,
  
  Risk_Profile = risk_return,
  
  Correlation_Summary = data.frame(
    Avg_Price_Correlation = mean(price_cor[upper.tri(price_cor)]),
    Avg_Return_Correlation = mean(return_cor[upper.tri(return_cor)], na.rm = TRUE)
  ),
  
  Market_Statistics = all_stocks %>%
    summarise(
      Overall_Avg_Return = mean(Daily_Return, na.rm = TRUE),
      Overall_Volatility = sd(Daily_Return, na.rm = TRUE),
      Max_Single_Day_Gain = max(Daily_Return, na.rm = TRUE),
      Max_Single_Day_Loss = min(Daily_Return, na.rm = TRUE)
    )
)

# Print report
print(final_report)

# Save report
saveRDS(final_report, "eda_final_report.rds")
```

---

## Additional Tips

### Best Practices
1. **Always check data quality** before analysis
2. **Document your findings** as you explore
3. **Save intermediate results** to avoid re-running long computations
4. **Use version control** for your R scripts
5. **Create reproducible reports** using R Markdown

### Next Steps After EDA
1. Feature engineering for modeling
2. Build predictive models (ARIMA, LSTM, etc.)
3. Portfolio optimization
4. Risk assessment and VaR calculations
5. Trading strategy development

### Export Results
```r
# Save plots
ggsave("price_trends.png", width = 12, height = 8)

# Export cleaned data
write.csv(all_stocks, "vn30_cleaned.csv", row.names = FALSE)

# Save workspace
save.image("vn30_eda_workspace.RData")
```

---

## Conclusion

This guideline provides a comprehensive framework for exploring the VN30 stock dataset. Adapt and expand sections based on your specific research questions and objectives.

**Remember:** EDA is an iterative process. Don't hesitate to revisit earlier steps as you uncover new insights!
