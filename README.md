# Call the Ambulance

## Table of Contents
- [Introduction](#introduction)
- [Dataset](#dataset)
- [Analysis](#analysis)
	- [Nationwide](#nationwide)
	- [Statewide](#statewide)
	- [Citywide](#citywide)
- [Business Application](#business-application)
- [Conclusion](#conclusion)

## Introduction
Here is the mile-high summary of the analysis:
 - Each year the Centers for Medicare & Medicaid Services publishes the Ambulance Fee Schedule 
 - This schedule details how much the Feds will reimburse Medicare/Medicaid ambulance rides based on
 	- Where you live 
 	- What type of transportation is used (car, plane, helicopter)
 	- What services are provided (thermometer, ventilator, amputation, etc.)
 - I combined three datasets together 
 	- Ambulance Fee Schedule (described above)
 	- Carrier & Location Codes (states, counties, cities, etc.)
 	- HCPCS Codes (healthcare codes for services rendered)
 - With this information I 
 	- Analyzed the fees at the national, state, and city scale
 	- Performed a business case study

If you pressed me to answer "What is the value of this work?", I would say:
 - Time intensive ETL is complete for the dataset
 - Demonstrated ambulance fee regression and ambulance ride forecasting can help hospitals plan out their revenue

## Dataset
The most salient variables in the transformed dataset are:
 - Contractor/Carrier and Locality (e.g. State, City, etc.)
 - HCPCS Code – Service provided (e.g. Basic Life Support, Advanced Life Support, etc.)
 - Base Rate – A national cost multiplier used by ‘The Centers for Medicare & Medicaid Services’
 - RVU – Ratio that represents service complexity (e.g. Basic Life Support = 1.0, Advanced Life Support = 1.9, etc.)
 - GPCI - "Inflation factor" to adjust payment to account for regional differences (i.e. Manhattan should be more expensive than Fargo)
 - Urban Base Rate – A proxy for how much the ambulance ride costs

  ![image](/docs/imgs/data_sample.png)

  *Sample of dataset*

## Analysis

### Nationwide
For the nationwide analysis we will be looking at the “Basic Life Support, Non-emergency Transport” ambulance service over time. 

Figure 2 details the published ambulance fees for all U.S. locations over time. On any given year, basic life support services cost anywhere from 180 to 280 dollars depending on whether you live in an affluent area or a rural area (GPCI variable captures this). 

As we see, the cost of basic life support steadily increases over the years. The second y-axis plots the base rate multiplier. Essentially, this multiplier is used by the Centers for Medicare & Medicaid Services to increase cost consistently across the nation. The key takeaways are that the spread of data within a year is dependent on the geographical location (GPCI) and that the growth in ambulance fees is due to the national base rate multiplier. 

  ![image](/docs/imgs/program_history_scatter_line.png)

  *Basic life support fees across the nation over time*

Figures 3 and 4 show the distribution of fees from 2010 to 2020 for all locations across the U.S. From 2010 to 2011, there was less variability in the price of basic life support across the nation (small standard deviation). From 2012 to 2016 the fee structure was consistent from year to year. From 2017-2020 the fee structure starts to moves towards a left skew. The key takeaway from these graphs is that the fee structures from year to year have remained quite consistent since 2012.  Potential causes of fee structure change could be the addition of more rural and highly dense areas to the fee schedule in later years. 

  ![image](/docs/imgs/density.png)

  *Basic life support fee distribution over time*

  ![image](/docs/imgs/density_standardized.png)

  *Standardized basic life support fee distribution over time*

### Statewide
It can be helpful for state officials to visualize how ambulance fees have changed over time. To accomplish this, I created a plotting function that takes in an HCPCS code and the state of interest and outputs plots as shown in Figure 5. Again, the key takeaway is that the growth in ambulance fees is primarily due to the national base rate multiplier. 

Additionally, I created a plotting function that takes in the state and year of interest and outputs a plot of median ambulance fees as shown in Figure 6. As we might expect, rotary wing transport, fixed wing transport, and specialty care transport are far more expensive than ground advanced/basic life support services.

```python
state_time_plotter(hcpcs='A0428', carrier='empire new york')
```
  *Time Series Plotting Function*

  ![image](/docs/imgs/minnesota_prices.png)

  *Basic life support fees for Minnesota*

```python
state_bar_plotter(year=2020, carrier='tennessee')
```
  *Bar Plot Plotting Function*

  ![image](/docs/imgs/texas_bar_prices.png)

  *Ranking of ambulance fees for various services in Texas*

### Citywide
It can be helpful for city officials to visualize how ambulance fees have changed over time. To accomplish this, I created a plotting function that takes in an HCPCS code and the city of interest and outputs plots as shown in Figure 8. Again, the key takeaway is that the growth in ambulance fees is primarily due to the national base rate multiplier. 

Additionally, I created a plotting function that takes in the city and year of interest and outputs a plot of median ambulance fees as shown in Figure 9. As we might expect, rotary wing transport, fixed wing transport, and specialty care transport are far more expensive than ground advanced/basic life support services.

```python
city_time_plotter(hcpcs='A0428', carrier='empire new york', locality='manhattan')
```
  *Time Series Plotting Function*

  ![image](/docs/imgs/austin_prices_line.png)

  *Basic life support fees for Austin, Texas*

```python
city_bar_plotter(year=2020, carrier='texas', locality='austin')
```
  *Bar Plot Plotting Function*

  ![image](/docs/imgs/austin_prices_bar.png)

  *Ranking of ambulance fees for various services in Austin*

## Business Application
Let us assume the following scenario:
 - A hospital operates an ambulance service
 - They have data on how many monthly ambulance rides have occurred over the past 10 years
 - They want to project their revenue for the next few years before the ambulance fee schedule comes out

To accomplish this goal let us:
1.	Perform simple linear regression on the ambulance fee schedule to:
	 - Predict future fees
2.	Perform ARIMA time series forecasting on their historical monthly ambulance ride data to:
	 - Predict future ambulance usage
3.	Combine the regression and time series forecast to get revenue

#### Simple Linear Regression – Fee Prediction
Simple linear regression was used to predict future ambulance fees. Regression results for an example location can be found in Figure 10.

  ![image](/docs/imgs/austin_linear_regression.png)

  *Simple linear regression to predict future ambulance fees*

#### ARIMA Forecasting – Ambulance Ride Prediction
Autoregressive Integrated Moving Average (ARIMA) forecasting is a machine learning method that uses past values to predict future ones. This method can handle seasonality in a time series. At the time of this analysis, no ambulance ride data could be found. Synthetic data was introduced to show how this methodology could prove useful if hospitals/providers were to offer ambulance ride data. ARIMA ambulance ride forecasts are displayed in Figure 11.

  ![image](/docs/imgs/austin_ambulance_rides_bls.png)

  ![image](/docs/imgs/austin_ambulance_rides_als.png)

  *ARIMA forecast of basic life support and advanced life support ambulance rides*

#### Revenue Forecasting
Simply put, predicted ambulance fees from regression were combined with ambulance ride forecasts to project revenue into the future. Results for the example location can be found in Figure 12.

  ![image](/docs/imgs/revenue_forecaset.png)

  *Revenue forecasts from basic life support and advanced life support ambulance rides*
  
#### Revenue Forecasting Tool
A function was developed so that revenue could be projected for any city in the dataset, for any number of months into future, and for any type of ambulance service (Basic life support, advanced life support, etc.). Simple linear regression, ARIMA forecasting, and revenue calculation are performed within this function and generate three standard plots (see Figure 13).

```python
revenue_forecast(df_rides=df_rides, hcpcs='A0428', num_months=36, carrier='florida', locality='miami')
```

  *Revenue Forecasting Function*

  ![image](/docs/imgs/function_pt1.png)
 
  ![image](/docs/imgs/function_pt2.png)

  ![image](/docs/imgs/function_pt3.png)

  *Output from revenue forecasting tool*

## Conclusion
The Ambulance Fee Schedule dictates the pay-out to ambulance providers for Medicare and Medicaid eligible clients. Through data cleaning and transformation, this data can now be analyzed at a national, state, and city level. 

From preliminary analysis, it was found that the distribution of ambulance fees across the nation has not changed dramatically since 2012. Since 2012, the primary driving factor for increasing ambulance fees is the base level rate multiplier. Essentially, from year to year the *Centers for Medicare & Medicaid Services* increases the base level rate multiplier and nothing else. Additionally, this analysis demonstrates that ambulance revenue can be forecasted by combining simple linear regression and ARIMA forecasting. The primary value of this project is in that it organizes a previously disparate and disorganized dataset for use in a firm or dashboard that analyzes ambulance rides across the U.S.  

  