# R_FinancialModel_Project


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

This project includes R markdown file, github document, and 5 graph images.

To read the report, please click on the github document called "EC48T_Project.md".

## Code and Resources Used

**R version:** 4.1.2

**Packages:** Tidyverse, Quantmod

## Profit Calculation Formula

Here is the script of our profit calculation model.

```
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
```

## Plots

Here is the graphs of one time frame as an example.

![](https://github.com/atakanpeker/R_FinancialModel_Project/blob/main/2016-2018.png)

![](https://github.com/atakanpeker/R_FinancialModel_Project/blob/main/2016-2018_ccf.png)
