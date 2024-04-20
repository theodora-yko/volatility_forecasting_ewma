For the summary of my work, please refer to: ewma_stock_volatility_prediction_summary_report.pdf
<hr> 

# Goals of the Project: 

We are given the historical price of six stocks on a second basis from day 1 to day 365. For each stock, we are
asked to predict its volatility, whose meaning should be defined in our own terms. This paper summarizes the
analysis of the data and the algorithm used to create predictions.

# Methods in Use: 

## Baseline EWMA Model:
For day $t$, I use monthly volatility data from day $t-lookback-1$ to day $t-1$ to predict the value for day $t$. Based on the exponential decay of autocorrelation values, we assign exponential weights to the past data and build an exponential weighted moving average model (EWMA). The model predicts volatility of day $t$, $\hat{MV}_t$ by taking in
    * Input: 
          * Exponential decay rate $\alpha$.
          * Length of lookback ($L$) = highest lag whose autocorrelation is outside the confidence interval.
          * Previous days' monthly volatility data within the length of lookback from the previous day of prediction.
    * Score Equation:
          * Since we value the accuracy of the volatility prediction as well as the 1 stdev confidence interval, we decide to use following score equations for each for model comparisons. 
          * For the volatility prediction, we use Root Mean Squared Error (RMSE) to see the overall size of error for all predictions for each stock: 
          $$\sqrt{\frac{\sum_{t = t_0}^{t_1}(MV_t - \hat{MV_{t}})^2}{t_1 - t_0}}.$$
          *  For the confidence interval ($C(t)$), we use the coverage rate: For a confidence interval on date $t$ from date $t_0$ to $t_1$, 
         $$ \frac{\sum_{t = t_0}^{t_1}(\mathbb{1}_{MV_t \in C(t)})}{t_1 - t_0} * 100 % $$.

## Dynamic-Weight Adjusted EWMA Model: 
Inspired by weighted linear regression's mechanism, to decrease the lag, we want to assign higher weights to more recent and important data based on some feature that can be engineered from the historical data. We notice that one way to do so is coming up with an indicator of a distribution shock (e.g. stock c). Such can be detected by comparing the RMSE of the most recent prediction to the past RMSE values. 

Specifically, if RMSE becomes larger than some confidence interval (within 1 standard deviation) then it means there has been a shift in the recent distribution of data. Then, the model needs to adjust the weights of the moving average assigned to $MV_{t-1}, \cdots MV_{t-lookback-1}$, specifically with focus on increasing the weight of $MV_{t-1}$. 

\begin{enumerate}
    \item Initialize the "weight" for each power of each exponential coefficient as 1.  
    \item Compute autocorrelation to detect how much more "significant" the recent values got. 
    \item Based on the ratio of autocorrelation, decrease the power of more significant acf by subtracting autocorrelation $\cdot$ step\_size from the current power. 
\end{enumerate}

Note that since alpha $<$ 1, the lower the power of alpha is, the higher weight is imposed to the actual data. Also, this ensures that same weighting mechanism applies for negative powers. Furthermore, since the power weights will change over time, influenced by all past historical data, it also takes into account long term memory of the past data while adapting to the recent changes as well.  


    

