### Data

https://www.kaggle.com/datasets/robikscube/hourly-energy-consumption

### Seasonality

### Modelling details

Models:

- LSTNet
- TCN
- Seasonally naive with weekly seasonality
- Dynamic regression model where seasonality is modelled with fourier features + ARIMA errors (referred to as SARIMAX in plots below)
  DEFAULT_VALIDATION_WINDOWS = [
  (datetime(2017, 5, 10, 12), datetime(2017, 5, 12, 12)), # [Wednesday, Friday]
  (datetime(2017, 6, 8, 10), datetime(2017, 6, 10, 10)), # [Thursday, Saturday]
  (datetime(2017, 7, 14, 14), datetime(2017, 7, 16, 14)), # [Friday, Sunday]
  (datetime(2017, 8, 12, 3), datetime(2017, 8, 14, 3)), # [Saturday, Monday]
  (datetime(2017, 9, 24, 7), datetime(2017, 9, 26, 7)), # [Sunday, Tuesday]
  (datetime(2017, 10, 23, 18), datetime(2017, 10, 25, 18)), # [Monday, Wednesday]
  (datetime(2017, 11, 7, 23), datetime(2017, 11, 9, 23)), # [Tuesday, Thursday]
  (datetime(2017, 12, 20, 2), datetime(2017, 12, 22, 2)), # [Wednesday, Friday]
  (datetime(2018, 1, 25, 12), datetime(2018, 1, 27, 12)), # [Thursday, Saturday]
  (datetime(2018, 2, 16, 19), datetime(2018, 2, 18, 19)), # [Friday, Sunday]
  ]

BoxCoxScaler for all models.
