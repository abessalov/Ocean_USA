# Predict FOMC Interest Rate Decision: Dec 17-18, 2024

## Files description

- README.md - task and files description.
- _startup.ipynb - libraries importing that executed in every notebook.
- 1_get_fomc.ipynb - script with parsing of FOMC meetings data.
- 2_analyze_fomc.ipynb - script with text analysis of the FOMC meeting statements.
- 3_get_fred.ipynb - script with parsing of economic indicators from FRED API. 
- 4_analyze_fred.ipynb - analysis of target rate and economic indicators.
- 5_modeling.ipynb - script developing model for predicting target rates differences to the following categories: -0.50%, -0.25%, 0%, +0.25%, +0.50%.

### Target preparing

We have done analytics for the last 25 years (Jan,2000 – Dec, 2024), therefore all the data have been collected for this period of time with some extension for feature preparing (lags, differences, etc).

1. We collected all dates of FOMC meetings taken from here: https://www.federalreserve.gov/monetarypolicy/fomccalendars.htm. Historical meetings parsed using the following pattern: https://www.federalreserve.gov/monetarypolicy/fomchistorical{year}.htm 
2. We collected all daily Federal Funds Target Rates based on DFEDTARU and DFEDTAR indicators using FRED API: https://fred.stlouisfed.org/docs/api/fred/. Effective Dec 16, 2008, target rate is reported as a range so we took the upper value here.
3. We took only the dates where there were changes in the rates since we need these events for predicting. But it's not enough, because we also need a periods when the rate didn't change. Therefore we merged two previous datasets to see the full picture of change or constancy of the rates. 
4. Then we converted this table to a monthly format. If there were several meetings in one month (usually this happened in the difficult periods of time – a pandemic, financial crisis..), then we sum up all the rate changes in that month.
5. The month value shifted on 1 month back since for example, for December rate we have the data till November and etc. 
6. We converted all the target rates differences to the following categories: -0.50%, -0.25%, 0%, +0.25%, +0.50%. If the target rate lies within this range, then we determine the nearest bucket.

### Features preparing

To create a features on which we will predict Federal Funds Target Rate changes we did the following steps:

1. We collected all the texts of FOMC meetings statements for the last 6 years. 
2. By analyzing these texts we retrieved the main topics discussed at these meetings.
3. Based on them we were searching for the suitable economic indicators available on FRED. While conducting text analysis and indicators searching we actively used the Open AI GPT-4o model: https://openai.com/index/hello-gpt-4o/  
4. We collected the values of these indicators by using FRED API.
5. We additionally prepared a required data transformations for some indicators. 
Based on the updating frequency:
- Monthly – took value as is.
- Lag monthly – took month value with the lag (since not all data are fresh when the Target Rate changes).
- Quarter – took last available quarter value.
- Weekly – calculated the average value of the month.
Based on the feature differences:
- Percent change 12 (pct12) – calculate a percent changes of 1 year back value.
- Difference12 (diff12) – calculated the difference with 1 year back value.

### Model developing
1. Output target:  Target rates differences from the following categories: -0.50%, -0.25%, 0%, +0.25%, +0.50%.2.
2. Input features: All features described before + previous value of Target Rate.
3. Models: We built Random Forest Classification models here since they are: 
- not prone to overfitting, 
- not sensitive to features outliers, 
- suitable for small samples,
- and have rather good quality.
  
4. Evaluation metrics:
- precision, 
- recall,
- weighted f1-score. 

5. Evaluation procedure: 
- For hyperparameters tuning we used RepeatedStratifiedKFold with 4 folds and repeat_num = 5.
- We used StratifiedKFold with 4 folds to have the values of evaluation metrics and compare them with simple last value solution. 

6. Finally we made a best model prediction for the Target Rate difference bucket on the Dec 17-18, 2024 FOMC meeting

