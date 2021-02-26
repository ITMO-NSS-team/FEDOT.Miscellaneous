## Sea surface height forecasting with multiscale chain

The page with time series forecasting case you can find [here](https://itmo-nss-team.github.io/FEDOT.Docs/real_cases/metocean-forecasting).
Below is an example of multi-scale time series forecasting with separate trend and trend residual forecasting. 

The main concept of how a time series can be predicted with multiscale chain can be seen in Figure 1.

<img src="img_metocean/multiscale_schema.png" alt="drawing" width="400"/>

Figure 1. Main idea of multiscale chain

As an example for this task, the time series of the surface height at the point obtained from the modelling results of the NEMO model for the Arctic region was taken.

This approach consists of the application of the LSTM network and the subtraction of its prediction results from the original time series. 
Using this architecture, a non-linear trend can be distinguished from a non-stationary time series during the first iteration. 
The next iteration gives some resemblance to the seasonal component.
The advantage of this approach can be considered to be the extraction of the trend and seasonal component without a priori assumptions about their shape. 
However, the same feature can be understood as a disadvantage, meaning the resulting components are less interpretable because a neural network is used.
An LSTM model was used to predict values at the next point in time. 
Its architecture is shown below (Figure 2).

<img src="img_metocean/LSTM-architecture.png" alt="drawing" width="400"/>

Figure 2. Architecture of LSTM neural network

A lag window equal to 12 hours was chosen for the experiment. 
Thus, using values in the previous 12 hours to train the neural network, we try to predict what will happen at the next point in time.
At the initial block of the model, Conv1d layers can be used to find patterns in a time series (such as curvature). 
An adding noise layer from the normal distribution to the input data was also added - this technique helps to avoid over-learning of the model. 
The last TimeDistributed layer converts the resulting features from the previous layers to the output value. Inside it, a Dropout layer is used - which also helps avoid over-learning.
 
Two decomposition scales are shown as an example. The first of them is a trend component. An example of the highlighted trend is shown below.

<img src="img_metocean/trend-component-example.png" alt="drawing" width="700"/>

Figure 3. Trend and residuals, determined by the algorithm for full dataset. On the left: selected trend component (orange) against the original time series; on the right, the difference between the selected trend and the original time series (the seasonal component).

<img src="img_metocean/seasonal-component-example.png" alt="drawing" width="700"/>

Figure 4. Trend and residuals, determined by the algorithm for a part of dataset. On the left: the selected trend component (orange) vs the background of the original time series; on the right: the difference between the selected trend and the original time series is the seasonal component.
 
After training, the trend model  was tested on a validation sample. 
All validation samples, predictions and their difference (seasonal component) is shown below (Figure 5). 

<img src="img_metocean/trend-model1.png" alt="drawing" width="700"/>

Figure 5. Top-down: the resulting trend model, validation sample and their difference

The resulting trend model has a standard error of 0.01 m on the validation sample.
 
The model for the seasonal component was obtained similarly. The results of the validation sample prediction are shown below.

<img src="img_metocean/validation-residuals-prediction1.png" alt="drawing" width="700"/>

Figure 6. The result of predicting the seasonal component model (orange) on the entire validation sample (blue)
 	
The implementation of such structure (LSTM+regression model for different scales) as Fedot composite model can be represented as follows:

<img src="img_metocean/fedot-implementation.png" alt="drawing" width="700"/>

Figure 7. Example of multiscale chain for time series forecasting

So, the implementation of the described model can be obtained by following code:

```python
# Define PrimaryNode models - its first level models
node_first = PrimaryNode('trend_data_model')
node_second = PrimaryNode('residual_data_model')

# Define SecondaryNode models - its second level models
node_trend_model = SecondaryNode('lstm', nodes_from=[node_first])
node_residual_model = SecondaryNode('rfr', nodes_from=[node_second])

# Root node - make final prediction
node_final = SecondaryNode('ridge', nodes_from=[node_trend_model,
                                              node_residual_model])
chain = TsForecastingChain(node_final)
```
To obtain a forecast, the chain_lstm.predict(dataset_to_validate) should be called.

The forecasts with different depth are differs as:

<img src="img_metocean/forecast-lstm.gif" alt="drawing" width="500"/>

The example of the optimisation for the predictive chain:

<img src="/FEDOT.Docs/real_cases/img_metocean/ts_opt.gif" alt="drawing" width="1000"/>