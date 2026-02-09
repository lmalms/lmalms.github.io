---
layout: post
title: "Gridsight - Forecasting UCI Electricity Demand"
date: 2026-02-07
categories: blog
---

# Gridsight - UCI Electricity Demand Forecasting

For this project, I wanted to learn more about advanced deep learning approaches to time series forecasting, in particular Temporal Convolutional Networks (TCNs). TCNs were among the first promising alternatives to recurrent neural networks (such as RNNs, LSTMs, and GRUs) for sequence modelling. Empirical results have shown that TCNs can capture much longer temporal dependencies than recurrent network architectures, despite the theoretical infinite memory of the latter. In addition, the convolution-based architecture of TCNs makes their computation much easier to parallelise on modern hardware compared to the inherently sequential processing of recurrent networks.

To understand the TCN architecture in detail, I re-implemented the model from scratch. My implementation is based on [this paper](https://arxiv.org/pdf/1803.01271) and closely follows [its implementation](https://github.com/locuslab/TCN/blob/master/TCN/tcn.py). I then evaluated its forecasting performance by predicting electricity demand for 20 different sites from the UCI Electricity Load Diagrams dataset.

## Dataset

The UCI Electricity Load Diagrams dataset contains electricity consumption data for 371 client sites at a 15-minute resolution, spanning the period from 01/01/2011 to 01/01/2015. To keep the scope of this project manageable, I chose to fit a separate model per site rather than training a single global model across all sites. Training 371 individual models would have required significant computational resources and training time, so instead I randomly sampled 20 sites for this analysis. All results shown below are based on those 20 sites.

The plot below shows electricity demand for all 20 sites during 2014. All sites exhibit strong daily seasonality with no obvious long-term trend over this period. The average demand level is fairly consistent across most sites (typically around 300 kW), although a small number of sites show substantially higher average loads, reaching up to approximately 1500 kW.

![demand-timeseries](/assets/gridsight-uci/demand_timeseries_2014.png)
_Time series of electricity demand data for all 20 sites for 2014. Grey lines show electricity demand for individual sites. The blue line shows the average demand across all sites for each 15-minute interval._

The plot below examines the seasonal structure in more detail by plotting the average electricity load per hour for each weekday across all 20 sites. For 18 of the 20 sites, the average hourly load profile is very consistent across days of the week. These sites generally show low demand between approximately 00:00 and 06:00, followed by higher demand throughout the daytime and early evening hours. In contrast, sites _MT_156_ and _MT_162_ show a clear weekday–weekend effect, with significantly lower consumption on weekends compared to weekdays.

![demand-seasonality](/assets/gridsight-uci/demand_by_hour_of_day_and_weekday_2014.png)
_Average hourly electricity load by day of week for all 20 sites. Each scatter point represents the average load in a given one-hour period (e.g. 05:00–06:00). Trend lines show the average load across all sites for each corresponding hourly period._

I also explored alternative ways of characterising the seasonal structure in the electricity load data, such as analysing the FFT spectra of the time series (see Appendix). These analyses lead to the same conclusion: for the majority of sites, the dominant seasonal pattern is a single daily cycle. Only sites _MT_156_ and _MT_162_ exhibit multiple seasonal components, with both daily and weekly seasonality present.

## Models, Training Setup, and Evaluation

My implementation of the TCN model closely follows the implementation [here](https://github.com/locuslab/TCN/blob/master/TCN/tcn.py). I did not perform any architectural hyperparameter tuning and used a default model configuration consisting of three temporal convolution blocks with 8, 16, and 32 output channels respectively. All blocks used a kernel of size 2 and used exponentially increasing (2^l where l is the layer index) dilation factors.

I trained all models with batch size 32 and learning rate 1e-03 for 50 epochs. For some of the earlier experiments I sampled data randomly when constructing data batches but later realised that the training data exhibited a distributional shift over time, so I trained later models without random sampling and instead maintained the temporal nature of the data when constructing batches.

This was a univariate time series forecasting problem so there was relatively little feature engineering involved. In addition to previous observations of the target variable, I also included sin / cosine encoded temporal features (e.g. hour of day, day of week, and month).

I benchmarked TCN performance against two baseline models, namely a seasonally naive model with a daily seasonality and an exponential smoothing model with a daily additive seasonality and no trend. For each of the 20 sites I evaluated each model on 10 different 2-day validation windows from 02/12/2014 to 21/12/2014.

## Results

Results were surprising and perhaps somewhat disappointing from the perspective of the performance of the TCN forecasts. The plot below shows the root mean squared scaled error by client site averaged over all validation windows. As shown in the left plot, the seasonally naive baseline model either outperforms or closely matches the performance of the TCN model for nearly all sites. The only exceptions are sites _MT_156_ and _MT_162_, which as mentioned above are the only sites in the set of randomly selected sites that show a weekday vs. weekend effect in addition to the daily seasonality exhibited by all sites. The poor performance of the seasonally naive model will therefore likely come from validation folds that either start with or contain a weekday/weekend boundary. For those validation windows the naive model with a daily seasonal period will produce poor forecasts.

![summary-forecast-errors](/assets/gridsight-uci/summary_forecast_errors.png)
_Average forecast errors and error variability for all sites and models._

This is nicely illustrated in the plots below which compare forecasts made by all three models for site _MT_162_ for a validation window starting on a Friday. The forecasts produced by the naive model (left plot) are very decent for the first 24 hours where the actual observations from the previous day (Thursday) are still a good estimate for future demand. For the second 24-hour period however (Saturday), predicting the same electricity load as on Thursday results in poor forecasts. The ETS and TCN models provide better forecasts in this regime with the TCN model producing the most accurate forecasts overall. However, from the results above it also seems likely that the seasonally naive method would have also produced strong results for sites _MT_156_ and _MT_162_ if I had used a weekly seasonal period (i.e. always predict the electricity demand from one week ago.)

<!-- | Naive | ETS | TCN | -->

| ![naive-forecasts-mt-162](/assets/gridsight-uci/naive_forecasts_MT_162_fold_2.png) | ![ets-forecasts-mt-162](/assets/gridsight-uci/ets_forecasts_MT_162_fold_2.png) | ![tcn-forecasts-mt-162](/assets/gridsight-uci/tcn_forecasts_MT_162_fold_2.png) |

_Forecasts for site MT162, fold 2. Left: Seasonal Naive; centre: ETS; right: TCN._

For comparison, the plot below shows forecasts for site _MT_263_ which only shows a daily seasonality. Here, the seasonal naive model provides very strong forecasts. The forecasts generated by the ETS and TCN models are also performant but they are not significantly more accurate than the forecasts made by the naive model. In fact, on average, as shown in the summary plot above, the naive model is slightly more accurate than both the ETS and TCN models for this site.

<!-- | Naive | ETS | TCN | -->

| ![naive-forecasts-mt-263](/assets/gridsight-uci/naive_forecasts_MT_263_fold_1.png) | ![ets-forecasts-mt-263](/assets/gridsight-uci/ets_forecasts_MT_263_fold_1.png) | ![tcn-forecasts-mt-263](/assets/gridsight-uci/tcn_forecasts_MT_263_fold_1.png) |

_Forecasts for site MT263, fold 1. Left: Seasonal Naive; centre: ETS; right: TCN._

## Conclusions

So it looks like most of the data from the set of sites selected for this project is essentially a random walk with seasonality. That is, after adjusting the demand data for the seasonal period (either daily or weekly) the time series is just white noise. This would explain why the seasonal naive model is the best performing model on average. This is not to say that the TCN model produces particularly bad forecasts; for several sites it matches the performance of the naive model. It just so happens that for this dataset with such strong autocorrelation it is hard to do better than the naive baseline.

In other settings, for example with more complex seasonal patterns or a somewhat weaker autocorrelation where the time series for the next seasonal period is not just exactly the same as from the previous period, the TCN model and other modelling approaches might have outperformed the naive model.

Either way, this was a good reminder to always include a naive baseline model as part of any forecasting work and to benchmark any more complex forecasting methods against it.

## Appendix

![demand-fft-spectra](/assets/gridsight-uci/demand_fft_spectra.png)
_FFT spectra of electricity load time series by site._

## References

- [UCI Electricity Load Dataset](https://archive.ics.uci.edu/dataset/321/electricityloaddiagrams20112014)
- [Temporal Convolutional Networks Paper](https://arxiv.org/pdf/1803.01271)
