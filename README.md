# 🌡️ Sydney Temperature Derivatives Pricing Model
This project explores the pricing of temperature derivatives in Sydney, Australia. Particularly, the pricing of Winter Temperature Options. The methodology combines time-series modeling with stochastic simulation to capture the dynamics of temperature and evaluate the fair value of weather-linked derivatives.

# 📈 1. Temperature Modeling (sydney_temperature_model.ipynb)
Objective: Clean and model long-term Sydney temperature data from 1859 to 02 May 2025.

Steps:
- Data cleaning from BoM datasets (min/max temperatures, station merging)
- Feature engineering and missing value imputation
- Time-series decomposition into trend, seasonality, and residual using Classical Decomposition (Additive)
- Residual stationarity testing using the Augmented Dickey-Fuller (ADF) test and modeled using AR(15) determined through AIC 
- Trend modeled using ARIMA(1,1,2) diagnosed through the ACF/PACF chart.
- Seasonality component analyzed using Signal Processing methods and estimated using the Fourier Series with coefficients estimated through Least-Squares.

Output: 
A generalized model for Sydney’s average temperature $T_n = T_{trend} + T_{seasonal} + \epsilon$

$$T_{trend} = ARIMA(1,1,2)$$
$$T_{seasonal} = 3.4997cos(\frac{2\pi{x}}{365}) + 2.7523sin(\frac{2\pi{x}}{365}) - 0.3661cos(\frac{4\pi{x}}{365}) - 0.1726sin(\frac{4\pi{x}}{365})$$
$$\epsilon = AR(15)$$

# 💸 2. Derivative Pricing (pricing_temperature_derivatives.ipynb)
Objective: Simulate temperature evolution and price winter temperature call and put options.

Steps:
- Modeled the temperature process as a mean-reverting process through a Modified Ornstein Uhlenbeck (OU) equation with $\kappa$ estimated through Euler Discretization and an AR(1) model.
$$dT_t = [\frac{d\bar{T}}{dt} + \kappa(\bar{T}_t - T_t)]dt + \sigma_t{dW_t}$$

- Volatility confirmed to be time-varying through a 7-day rolling window and treated as a mean-reverting OU process with $\kappa$ estimated through Euler discretization and an AR(1) model.
$$d\sigma_t = \kappa(\bar{\sigma_t} - \sigma_t)dt + \gamma_t{dW_t}$$

Pricing Approaches:
- Risk-Neutral Pricing: Derive the Risk-Neutral closed-form solution by making the naive assumption that the probability of temperature going above 18 is infinitesimally small to achieve a normal distribution.
$$\mu_n = 18n - \sum^n_{i=1}E^Q[T_i | \mathcal{F}_t]$$
$$\sigma^2_n = \sum^n_{i=1}V^Q[T_i | \mathcal{F}_t] + 2\sum_{i<j}Cov[T_i, T_j | \mathcal{F}_t]$$
$$C_t = \alpha{e^{-r(T-t)}} [(\mu-K)\Phi(-z^*) + \frac{\sigma}{\sqrt{2\pi}}(e^{-\frac{(z^{*})^2}{2}})]$$
$$P_t = \alpha{e^{-r(T-t)}}\left[(K-\mu)\left(\Phi(z^*) - \Phi(-\frac{\mu}{\sigma})\right) + \frac{\sigma}{\sqrt{2\pi}}\left(e^{-\frac{(z^*)^2}{2}} - e^{-\frac{\mu^2}{2\sigma^2}}\right)\right]$$

- Monte Carlo Simulation: Simulates temperature paths and compute expected payoffs.
$$C_t = \frac{{e^{-r(T-t)}}}{n}\sum^n_{i=1}\alpha(DD_i-K)^+$$
$$P_t = \frac{{e^{-r(T-t)}}}{n}\sum^n_{i=1}\alpha(K-DD_i)^+$$

# 📌 3. Limitations
1. As this project was to learn about time-series, there were a lot of gaps in knowledge. One of which was combining a continuous process with a discrete time component, in the end, we tried to model the temperature data as a modified OU with a time-varying volatility (modelled using OU) process. However in the modified OU process, we used an ARIMA(1,1,2) model that only functions in discrete time. Luckily in our case, since we are utilizing daily temperature, the ARIMA model is still a decent approximation.

2. Instead of using daily volatility, we used a 7-day rolling window to approximate the daily standard deviation due to the lack of intraday temperature data. Therefore, the parameters influenced by the volatility may not be entirely accurate and could have smoothing effects applied.

3. The Risk-Neutral calculation uses an arbitrary $\lambda$ estimated through trial and error. Therefore, it is not reflective of actual market conditions and needs to be properly calibrated to get a realistic estimate of option price.

4. With the absence of readily available Temperature Derivative data in Sydney, Australia, the current prices are merely estimates of the fair-value and further evaluation is required to gain confidence in the calculated prices. 