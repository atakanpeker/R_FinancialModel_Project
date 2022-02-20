EC48T Financial Model Project
================
Atakan Peker

## About

This is our group project (3 contributing members) for our “Financial
Modelling and Its Applications Using R” course.

This report’s aim is trying to find a cross correlation between RUB/TRY
and WTI/TRY price through evaluating “Adjusted” prices.

We collected data, which is from 2010 and 2021, from yahoo finance,
divided them into 4 time frames, and evaluating each one of them.

Also, this report presents a basic financial model based on the trading
opportunity that may come out with WTI crude oil prices and the
currency.

Through this model, we wanted to know whether we can make profit out of
this opportunity.

After the calculations, we found a positive correlation and trading
opportunity. However, we make very small profit out of it.

We used “quantmod” and “tidyverse” packages in our project.

    library(quantmod)
    library(tidyverse)

## Collecting the data

We are collecting the data from yahoo finance. However, yahoo finance
does not have RUB/TRY and WTI(according to TRY) data so we basically
took RUB/USD, TRY/USD, and WTI (in terms of USD).

Then, we adjusted those to have RUB/TRY and WTI values in TRY through
assigning new variables.

    rub <- getSymbols("RUB=X", src = "yahoo", from = as.Date("2010-01-01"), to = as.Date("2021-12-31"), auto.assign = F)
    try <- getSymbols("TRY=X", src = "yahoo", from = as.Date("2010-01-01"), to = as.Date("2021-12-31"), auto.assign = F)
    wti_crude_oil <- getSymbols("CL=F", src = "yahoo", from = as.Date("2010-01-01"), to = as.Date("2021-12-31"), auto.assign = F)

    rubtry <- try/rub

    wti <- wti_crude_oil*try

## Cleaning the data

To clean the “rubtry” and “wti” data, we need to create dataframes out
of them.

    rubtry <- data.frame(Date = index(rubtry), rubtry, row.names = NULL)

    wti <- data.frame(Date = index(wti), wti, row.names = NULL)

First, we cleaned the rubtry data.

We decided that we do not need the “Volume” of rubtry, we omitted the
null values, and assigned much more cleaner column names.

We found only one outlier in the “Adjusted” column which is a incorrect
as we controlled it, so we removed it from our dataframe.

    rubtry <- select(rubtry, -TRY.X.Volume)

    rubtry <- na.omit(rubtry)

    colnames(rubtry) <- c("Date", "Open", "High", "Low", "Close", "Adjusted")

    if (max(rubtry["Adjusted"]) > 3) {
      maxrow <- which.max(rubtry$Adjusted)
      rubtry <- rubtry[c(-(maxrow)), ]
    }

For the wti data, we did not encounter with outliers or incorrect data.

We ommitted the null values and assigned new column names.

    wti <- na.omit(wti)

    colnames(wti) <- c("Date","Open", "High", "Low", "Close", "Volume", "Adjusted")

We found out that row numbers of both dataframe are not matched with
each other since we ommitted null values and an outlier (Also, the
dataset we get may be problematic.)

To solve this problem, we match each row on a dataframe with another
dataframe based on their “Date” data so that if one date consists of
both rubtry and wti data, then it will not be removed.

    xrubtry <- data.frame()

    for (i in 1:length(rubtry$Date)){
      
      if (any(rubtry$Date[i] == wti$Date)){
        xrubtry <- rbind(xrubtry, rubtry[i, ])
      }
    }

    rubtry <- xrubtry

    xwti <- data.frame()

    for (i in 1:length(wti$Date)){
      
      if (any(wti$Date[i] == rubtry$Date)){
        xwti <- rbind(xwti, wti[i, ])
      }
    }

    wti <- xwti

Now, the row numbers are matched and we can calculate the cross
correlation.

## Filtering the data

For our research, we took the data from 2010 to 2021 and we divided it
to 4 time frame to investigate each one of them (2010-2012, 2013-2015,
2016-2018, 2019-2021).

    rubtry_1 <- filter(rubtry, between(Date, as.Date("2010-01-01"), as.Date("2012-12-31")))
    rubtry_2 <- filter(rubtry, between(Date, as.Date("2013-01-01"), as.Date("2015-12-31")))
    rubtry_3 <- filter(rubtry, between(Date, as.Date("2016-01-01"), as.Date("2018-12-31")))
    rubtry_4 <- filter(rubtry, between(Date, as.Date("2019-01-01"), as.Date("2021-12-31")))

    wti_1 <- filter(wti, between(Date, as.Date("2010-01-01"), as.Date("2012-12-31")))
    wti_2 <- filter(wti, between(Date, as.Date("2013-01-01"), as.Date("2015-12-31")))
    wti_3 <- filter(wti, between(Date, as.Date("2016-01-01"), as.Date("2018-12-31")))
    wti_4 <- filter(wti, between(Date, as.Date("2019-01-01"), as.Date("2021-12-31")))

## Plots

Here are the plots of each time frame that we filtered.

![](https://github.com/atakanpeker/R_FinancialModel_Project/blob/main/2010-2012.png)

![](https://github.com/atakanpeker/R_FinancialModel_Project/blob/main/2013-2015.png)

![](https://github.com/atakanpeker/R_FinancialModel_Project/blob/main/2016-2018.png)

![](https://github.com/atakanpeker/R_FinancialModel_Project/blob/main/2019-2021.png)

We may say that each plot seems RUB/TRY and WTI/TRY are highly
correlated, but this claim should be supported by calculations.

## Analysis

We calculated the cross correlation and make linear regression to each
time frame.

### Cross Correlation

    ccf_1 <- print(ccf(rubtry_1["Adjusted"], wti_1["Adjusted"]))

![](EC48T_Project_files/figure-markdown_strict/unnamed-chunk-9-1.png)

    ## 
    ## Autocorrelations of series 'X', by lag
    ## 
    ##   -25   -24   -23   -22   -21   -20   -19   -18   -17   -16   -15   -14   -13 
    ## 0.759 0.766 0.772 0.778 0.784 0.788 0.793 0.799 0.805 0.809 0.814 0.818 0.823 
    ##   -12   -11   -10    -9    -8    -7    -6    -5    -4    -3    -2    -1     0 
    ## 0.827 0.831 0.835 0.840 0.845 0.849 0.854 0.859 0.863 0.867 0.871 0.875 0.880 
    ##     1     2     3     4     5     6     7     8     9    10    11    12    13 
    ## 0.881 0.880 0.878 0.876 0.874 0.873 0.872 0.870 0.869 0.868 0.866 0.865 0.864 
    ##    14    15    16    17    18    19    20    21    22    23    24    25 
    ## 0.864 0.863 0.861 0.861 0.859 0.857 0.853 0.851 0.848 0.847 0.844 0.841

    ccf_2 <- print(ccf(rubtry_2["Adjusted"], wti_2["Adjusted"]))

![](EC48T_Project_files/figure-markdown_strict/unnamed-chunk-9-2.png)

    ## 
    ## Autocorrelations of series 'X', by lag
    ## 
    ##   -25   -24   -23   -22   -21   -20   -19   -18   -17   -16   -15   -14   -13 
    ## 0.855 0.858 0.860 0.863 0.866 0.869 0.872 0.876 0.879 0.882 0.884 0.887 0.888 
    ##   -12   -11   -10    -9    -8    -7    -6    -5    -4    -3    -2    -1     0 
    ## 0.890 0.892 0.893 0.895 0.897 0.899 0.901 0.902 0.903 0.905 0.906 0.907 0.909 
    ##     1     2     3     4     5     6     7     8     9    10    11    12    13 
    ## 0.905 0.900 0.895 0.889 0.883 0.878 0.872 0.865 0.858 0.852 0.845 0.839 0.832 
    ##    14    15    16    17    18    19    20    21    22    23    24    25 
    ## 0.824 0.815 0.807 0.799 0.792 0.784 0.777 0.769 0.762 0.754 0.746 0.738

    ccf_3 <- print(ccf(rubtry_3["Adjusted"], wti_3["Adjusted"]))

![](EC48T_Project_files/figure-markdown_strict/unnamed-chunk-9-3.png)

    ## 
    ## Autocorrelations of series 'X', by lag
    ## 
    ##   -25   -24   -23   -22   -21   -20   -19   -18   -17   -16   -15   -14   -13 
    ## 0.817 0.824 0.830 0.835 0.841 0.847 0.853 0.858 0.863 0.868 0.874 0.880 0.885 
    ##   -12   -11   -10    -9    -8    -7    -6    -5    -4    -3    -2    -1     0 
    ## 0.890 0.894 0.899 0.903 0.908 0.912 0.916 0.920 0.924 0.928 0.932 0.937 0.942 
    ##     1     2     3     4     5     6     7     8     9    10    11    12    13 
    ## 0.940 0.938 0.935 0.934 0.933 0.931 0.928 0.926 0.923 0.921 0.918 0.916 0.913 
    ##    14    15    16    17    18    19    20    21    22    23    24    25 
    ## 0.910 0.906 0.903 0.899 0.896 0.892 0.888 0.885 0.881 0.878 0.875 0.871

    ccf_4 <- print(ccf(rubtry_4["Adjusted"], wti_4["Adjusted"]))

![](EC48T_Project_files/figure-markdown_strict/unnamed-chunk-9-4.png)

    ## 
    ## Autocorrelations of series 'X', by lag
    ## 
    ##   -25   -24   -23   -22   -21   -20   -19   -18   -17   -16   -15   -14   -13 
    ## 0.564 0.573 0.582 0.593 0.606 0.620 0.633 0.648 0.662 0.676 0.689 0.704 0.720 
    ##   -12   -11   -10    -9    -8    -7    -6    -5    -4    -3    -2    -1     0 
    ## 0.736 0.751 0.767 0.783 0.802 0.822 0.834 0.843 0.852 0.860 0.869 0.880 0.893 
    ##     1     2     3     4     5     6     7     8     9    10    11    12    13 
    ## 0.883 0.874 0.865 0.858 0.851 0.845 0.837 0.824 0.811 0.798 0.787 0.777 0.767 
    ##    14    15    16    17    18    19    20    21    22    23    24    25 
    ## 0.757 0.749 0.741 0.733 0.724 0.714 0.705 0.696 0.689 0.682 0.676 0.668

With respect to these calculations, we can see that 2016-2018 time frame
has the highest correlation by “0.942”. Then the 2013-2015 comes as the
second, 2019-2012 comes as the third, and the last one is 2010-2012.

Clearly, we can supported the claim that RUB/TRY and WTI/TRY are
positively and highly correlated within each time frame, therefore they
are correlated from 2010 to 2021.

Within these findings, we can now create a financial model that can be
evaluating this correlation, finding a trading opportunity, and making
profit out of it.

But before making the model, we need to do the linear regression
approach.

### Linear Regression

    test_1 <- lm(rubtry_1$Adjusted~wti_1$Adjusted)
    test_2 <- lm(rubtry_2$Adjusted~wti_2$Adjusted)
    test_3 <- lm(rubtry_3$Adjusted~wti_3$Adjusted)
    test_4 <- lm(rubtry_4$Adjusted~wti_4$Adjusted)

    print(test_1)

    ## 
    ## Call:
    ## lm(formula = rubtry_1$Adjusted ~ wti_1$Adjusted)
    ## 
    ## Coefficients:
    ##    (Intercept)  wti_1$Adjusted  
    ##      0.0315341       0.0001567

    print(test_2)

    ## 
    ## Call:
    ## lm(formula = rubtry_2$Adjusted ~ wti_2$Adjusted)
    ## 
    ## Coefficients:
    ##    (Intercept)  wti_2$Adjusted  
    ##      0.0187952       0.0002037

    print(test_3)

    ## 
    ## Call:
    ## lm(formula = rubtry_3$Adjusted ~ wti_3$Adjusted)
    ## 
    ## Coefficients:
    ##    (Intercept)  wti_3$Adjusted  
    ##      0.0297485       0.0001507

    print(test_4)

    ## 
    ## Call:
    ## lm(formula = rubtry_4$Adjusted ~ wti_4$Adjusted)
    ## 
    ## Coefficients:
    ##    (Intercept)  wti_4$Adjusted  
    ##      6.263e-02       9.715e-05

## Financial Model

We created a financial model that calculates the profit if there is a
trading opportunity.

The model’s code is as follows:

    ProfitCalculate <- function(wti_price, rub_price, coeff, int, day_lag, initial_budget, trade_amount)
    {
      if(length(wti_price)!=length(rub_price))
      {
        return("Prices series should have the same length")
      }
      
      rub_hold <- 0
      wti_hold <- 0
      budget <- initial_budget
      
      start <- day_lag+1
      end <- length(rub_price)-1
      
      for(i in start:end)
      {
        
        if(rub_price[i] >= wti_price[i-day_lag])
        {
          budget + 100
          rub_hold <- rub_hold - rub_price/100
        }
        
        else if(rub_price[i] < wti_price[i-day_lag])
        {
          budget - 100
          rub_hold <- rub_hold + rub_price[i]/100
        }
      }
      
      calced_profit <- budget + rub_price[length(rub_price)] * rub_hold - initial_budget
      
      return(paste("Profit = ", calced_profit))
    }

In this function, we considered our wti and rubtry prices, coefficients
and intercepts of the time frame that we calculate and we assumed a day
lag as we kind of applying backtesting approach, initial budget, and
amount of trade in our research.

Also, we wanted to remove our last time frame which is 2019-2021 in this
report because its coefficients are a really odd comparing with other
time frames.

    profit_1 <- ProfitCalculate(wti_price = wti_1$Adjusted, rub_price = rubtry_1$Adjusted,
                                coeff = test_1$coefficients[2], int = test_1$coefficients[1],
                                day_lag = 10, initial_budget = 1000, trade_amount = 100)

    profit_2 <- ProfitCalculate(wti_price = wti_2$Adjusted, rub_price = rubtry_2$Adjusted,
                                coeff = test_2$coefficients[2], int = test_2$coefficients[1],
                                day_lag = 10, initial_budget = 1000, trade_amount = 100)

    profit_3 <- ProfitCalculate(wti_price = wti_3$Adjusted, rub_price = rubtry_3$Adjusted,
                                coeff = test_3$coefficients[2], int = test_3$coefficients[1],
                                day_lag = 10, initial_budget = 1000, trade_amount = 100)

    print(profit_1)

    ## [1] "Profit =  0.0239845851167502"

    print(profit_2)

    ## [1] "Profit =  0.0160696861847782"

    print(profit_3)

    ## [1] "Profit =  0.0346343394402311"

We found that we can make profit, but it is very small, nearly 2-3%.
