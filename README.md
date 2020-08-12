
# Predict Warsaw Air Pollution
### Udacity Machine Learning Engineer Nanodegree

<span style="color: gray; font-size:1em;">Mateusz Zajac</span>
<br>
<span style="color: gray; font-size:1em;">Aug-2020</span>


## Project Objective
**Air pollution in Poland is the worst in Europe.** 36 of the 50 most polluted cities in the European Union are located in Poland (WHO, 2018). It mainly derives from the fact that the country is dependent on coal to power its homes and economy. Poland is the second largest coal-mining country in Europe (after Germany). On top of that, ageing cars (the average age of vehicles is 13 years) spew vast amounts of exhaust fumes into the air.

Air pollution in Warsaw is so bad that people breathing in polluted air might as well be smoking 1,000 cigarettes a year. The situation is especially bad in winter. Residents need to wear facemasks outdoors so as to try and protect themselves from the adverse effects of persistent air pollution. Children are especially at risk across the country, yet almost two-thirds of kindergartens around the country are believed to be in heavily polluted areas.

The main objective of this project is to accurately predict PM2.5 concentration according to different air quality levels. This may help people to decide on whether to wear masks on certain days/hours of the day, if it is safe to open a window, or even if it is a good idea to go for a walk/run during certain hours.

[World Bank: Air Quality in Poland](http://documents1.worldbank.org/curated/en/426051575639438457/pdf/Air-Quality-in-Poland-What-are-the-Issues-and-What-can-be-Done.pdf)


## Dataset

The meteorological data for Warsaw comes from [darksky.net API](https://darksky.net/dev/docs#time-machine-request). The pollution data comes from the [GIOŚ Archives](http://powietrze.gios.gov.pl/pjp/archives) (the archives of Chief Inspectorate of Environmental Protection) from three monitoring stations located in Warsaw. 

Note: notebooks assume data files are stored in `./data/` directory.

### Data
The darksky dataset consists of approximately 43,500 data points, with each datapoint having 20 features. 

**Features**

- `apparentTemperature`: the apparent (or “feels like”) temperature in degrees Fahrenheit.
- `cloudCover`: the percentage of sky occluded by clouds, between 0 and 1, inclusive.
- `dewpoint`: the dew point in degrees Fahrenheit.
- `humidity`: the relative humidity, between 0 and 1, inclusive.
- `icon`: a machine-readable text summary of this data point, suitable for selecting an icon for display.
- `ozone`: the columnar density of total atmospheric ozone at the given time in Dobson units.
- `precipAccumulation`: the amount of snowfall accumulation expected to occur in inches. If no snowfall is expected, this property will not be defined.
- `precipIntensity`: the intensity (in inches of liquid water per hour) of precipitation occurring at the given time. This value is conditional on probability (that is, assuming any precipitation occurs at all).
- `precipProbability`: the probability of precipitation occurring, between 0 and 1, inclusive.
- `precipType`: the type of precipitation occurring at the given time. If precipIntensity is zero, then this property will not be defined.
- `pressure`: the sea-level air pressure in millibars.
- `summary`: a human-readable text summary of this data point. This property has millions of possible values, so don’t use it for automated purposes: use the icon property, instead.
- `temperature`: the air temperature in degrees Fahrenheit.
- `time`
- `uvIndex`: the UV index.            
- `visibility`: the average visibility in miles, capped at 10 miles.          
- `windBearing`: the direction that the wind is coming from in degrees, with true north at 0° and progressing clockwise. If windSpeed is zero, then this value will not be defined.
- `windGust`: the wind gust speed in miles per hour.
- `windspeed`: the wind speed in miles per hour.

**Target Variable**
The target variable comes from the GIOS dataset which has approximately 43,500 data points, with each datapoint having 4 features:

- `pm25_nie`: Warsaw, Niepodległości Avenue
- `pm25_kon`: Warsaw, Kondratowicza Street
- `pm25_wok`: Warsaw, Wokalna Street
- `date`


## Methods Used
* Inferential statistics
* Data visualization
* Feature engineering
* Feature selection
* Machine learning
* Parameters optimization
* Predictive modeling


## Technologies
* Python >= 3.6
* numpy >= 1.19
* pandas >= 1.1.0
* scikit-learn >= 0.23.2
* xgboost == 1.10
* lightgbm == 2.3.1
* eli5 == 0.9.0
* hyperopt == 0.2.4


## Implementation 
The implementation of this project has been split into three parts and workbooks.

**EDA**
 - gathering data from the darksky and GIOS
 - data exploration and exploratory visualization
 - data cleaning and handling missing values

**Feature engineering**

 - **Time based features** like ‘year’, ‘month’, ‘day’, ‘weekday’, and ‘hour’ were extracted from the timestamp. Based on these features I created additional ones like ’season’ and ‘IsWeekend’.

 - **Log transformations** were performed on the most skewed distributions and features relevant for the prediction, meaning  'humidity', 'windSpeed', 'PM25_nie', 'PM25_wok'.

 - **Weather condition features:**
	 - Time lag: shifting the observations by 1h and 24h to the future
	 - Moving averages were calculated over the period of 12h, 24h, 72h and 168h
	- Mean and median aggregations per day and month

 - **Deviations from daily and monthly mean/median:** for the most correlated with PM2.6 particle features (temperature, humidity, wind speed and wind bearing), deviations from the daily and monthly mean/median were calculated (for instance, [June’s monthly average] – [current June’s temperature])

 - **Dummy variables** for the ‘season’ feature were created

 - **Factorization of Categorical Variables** was performed on ‘icon’

 - **Creating custom combinations:**
	- subtract moving averages (MA): 12h MA from 24h MA and 24h MA from 72h MA for temperature, apparent temperature, dew point, humidity, wind speed and wind bearing
	- True/False statements: for features that were the most correlated with PM2.5, for instance features like ‘isHumidWorkingDay’, ‘isColdWorkingDay’ were created with the assumption that air pollution is worse during business days with higher humidity and lower temperature. In addition, similar features were created for hours between 20:00 and 08:00 during business days, as looking at the above heat maps it looks like during these hours the concentration of PM2.5 particle in the air is higher.

**Training, evaluation and fine tuning the model**
 -  loading and splitting the dataset into train (01/01/2015-30/06/2018) and test (01/07/2019-31/12/2019). 
 - training and predicting PM2.5 with simple models
	 - on original darksky features
	 - on original darksky features extended with time-based features
	 - on all features created in the feature engineering process
 - model parameters optimization with HyperOpt
	 - on all features created in the feature engineering process
	 - on features selected by RFE
	 - on features selected manually


## Models evaluation / metrics

 **RMSE** and **R2** score has been used as the evaluation metrics.

- **Root mean square error** is commonly used in climatology, forecasting, and regression analysis to verify experimental results. RMSE is the standard deviation of the residuals – a measure of how far from the regression line data points are. In general, the lower the RMSE, the better the model fits your data.

- **R-squared** is a statistical measure of how close the data are to the fitted regression line. In other words, it is the percentage of the response variable variation that is explained by a linear model. R-squared = Explained variation / Total variation. In general, the higher the R-squared, the better the model fits your data.

## Results
Best performing models:
 - **LightGBM:** RMSE: 6.6977, r2: 59.0%
 - **XGBoost:** RMSE: 6.6978, r2: 59.0%
<br>

PM2.5 actual vs prediction for July:
![results](https://i.imgur.com/OR7RFGy.png)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4MDkzNjk1MTYsMTEzNjM5NDcwLC0xOT
M4ODUzMTM0LDUxMzQwMDkyMCw3MzM1MTIxMTYsLTg3NTA1OTk5
NCwtMTA0NzMzMjMzMSwxMjA1MTMzNDkxXX0=
-->