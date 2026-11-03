# From Ordered History to Predictions: Building Time-Series Features

A spreadsheet can contain a decade of market prices and still be a poor prediction dataset. The problem is not necessarily the number of rows. It is whether each training example connects the past to the right future. Shift a series by one row in the wrong direction, and a model can appear to learn tomorrow using information from tomorrow.

The opening section of Chapter 5 of *Python for Algorithmic Trading*, by Yves J. Hilpisch, introduces practical prediction of future market movements from past observations. It reviews ordinary least squares (OLS), shows how time ordering changes the way we prepare regression inputs, and frames logistic regression and neural networks as approaches to predicting movement direction. The central engineering lesson is simple: before fitting a model, define exactly which historical values are available at each prediction point and which later value is the target.

## A line can fit without understanding time

In ordinary linear regression, a model relates an input $x$ to an outcome $y$. For a straight line, the equation is

$$
y_i = \beta_0 + \beta_1 x_i + \varepsilon_i,
$$

where $\beta_0$ is the intercept, $\beta_1$ is the slope, and $\varepsilon_i$ represents the part not explained by the line. OLS chooses coefficients that minimize the sum of squared residuals, $\sum_i (y_i - \hat{y}_i)^2$.

A scatterplot of noisy observations around a line is a useful first experiment: fit the line, inspect its slope and intercept, then compare its predictions with the data. The chapter uses NumPy's `polyfit` for a degree-one fit and `polyval` to evaluate the fitted polynomial. With degree one, the fitted polynomial is a straight line. Extending its input range produces extrapolated values, but it does not make those values reliable. The line is only a mathematical continuation of the estimated relationship.

For a static dataset, row order does not change the fitted line if the same $(x_i, y_i)$ pairs remain together. Time-series prediction is different in an important way. A price observed on Tuesday must not be confused with one from Friday merely because both appear in the dataset. The order determines what was knowable when a prediction would have been made.

## Turn a series into past-to-future examples

Suppose observations are $z_0, z_1, \ldots, z_{T-1}$, and we choose $L$ lags. Each training row uses the previous $L$ observations as inputs and the next observation as its target:

$$
[z_t, z_{t+1}, \ldots, z_{t+L-1}] \longrightarrow z_{t+L}.
$$

There are $T-L$ such rows when every observation is available. In matrix notation, put the input rows into $A$ and the targets into $b$. A linear model seeks coefficients $w$ such that $Aw$ approximates $b$ in the least-squares sense. A practical implementation can use a least-squares solver such as NumPy's `lstsq`. Unlike a simple polynomial fit, this formulation handles multiple input columns.

Here is a small, complete example. Let the observed index-like series be $[10, 13, 12, 16, 15, 19]$, and use two lags. The correctly aligned design matrix and target are

$$
A = \begin{bmatrix}10&13\\13&12\\12&16\\16&15\end{bmatrix},
\qquad
b = \begin{bmatrix}12\\16\\15\\19\end{bmatrix}.
$$

The first row says that values 10 and 13 are used to predict 12. The last says that 16 and 15 are used to predict 19. There are $6-2=4$ rows. This is the key transformation: the time series has become paired input and target data, while preserving temporal order.

For a transparent prediction calculation, suppose a model has already been fitted and its coefficients, in the order of the two inputs, are $w=[0.4, 0.6]$, with no intercept. For the latest available pair, 16 and 15, its prediction is

$$
0.4(16) + 0.6(15) = 6.4 + 9 = 15.4.
$$

The arithmetic checks: the weighted contributions sum to 15.4. This is an illustration of evaluating a fitted model, not a claim that these coefficients are the OLS solution for the four rows above. In real work, the fitting procedure estimates coefficients from training data, and an intercept may also be appropriate. The example makes the data alignment explicit without pretending a tiny dataset is a useful market model.

## Choose the thing you actually want to predict

Predicting an absolute future level and predicting a direction are different tasks. A regression model can estimate a numeric outcome, such as a future price or return. A directional task instead labels an outcome, for example whether a future return is positive or not. Logistic regression is a common baseline for classification because it estimates a probability for a class; a decision rule can then convert that probability into a label. Neural networks can also be used for classification, but a more complicated model does not repair a faulty target or a time leak.

The chapter presents these as practical families of methods: linear regression for levels or direction-related prediction, logistic regression as a classification-oriented linear model, and neural networks for directional prediction. It motivates them with the assumption that historical patterns may contain information about future movements, rather than treating the efficient-market hypothesis as universally true. That is a modeling premise to investigate, not a guarantee that a pattern exists or will persist.

A useful research workflow makes the prediction question concrete before selecting an algorithm:

1. Define the prediction time and the information available then.
2. Define a target, such as the next observation, a future return, or a direction label.
3. Construct lagged inputs so that every row uses only earlier information.
4. Split observations chronologically for evaluation, rather than randomly mixing past and future.
5. Compare a simple baseline with more complex models, and inspect errors as well as summary metrics.

The first three steps establish a valid problem. The remaining steps help test whether a model offers useful predictive performance on data it did not train on. A good historical fit alone does not establish that it will work in a later period.

## Applications beyond trading

Lagged regression is useful anywhere measurements arrive in sequence. A laboratory can use prior sensor readings to estimate the next reading, provided the target is recorded consistently. An operations team can model the next hour's demand from recent demand. An asset pipeline can study whether file counts or storage use follow recurring patterns. In each case, the same design question applies: what was available at prediction time, and what later measurement is the target?

The method is a starting point, not a universal solution. A straight-line relationship may be too simple; a model with many lags may be unstable or hard to interpret. Financial observations can also change behavior across periods. Model evaluation should therefore respect time and make its assumptions visible. A prediction is not a causal explanation, and a useful historical association is not proof of a dependable future signal.

## Assumptions and common mistakes

The basic least-squares setup assumes a numeric target and a design matrix whose rows correspond to the intended prediction cases. It does not require observations to be interchangeable in time, but the analyst must preserve time order when building and evaluating those rows. A regression with an intercept has the form $Aw + c$; omitting the intercept is a modeling choice, not an automatic property of linear regression.

Watch for off-by-one errors: the target must occur after all inputs in its row. Avoid fitting preprocessing steps or model parameters on future evaluation data. Be cautious when several lags are strongly correlated, since coefficient estimates may be poorly determined even if predictions on the training rows look close. Finally, extrapolating a line beyond the observed range can produce a number, but the calculation alone says nothing about how trustworthy that number is.

## Exercises

1. A series has 20 observations and you use 4 lags. How many one-step training rows can you construct? **Answer:** $20-4=16$.
2. With 3 lags, should the target be the third input or the observation immediately after the third input? **Answer:** The observation immediately after it.
3. If a model predicts a future return's sign, is its output necessarily a predicted price level? **Answer:** No. A direction label and a numeric price level are different targets.

The practical takeaway is not that every market movement can be forecast. It is that a prediction experiment begins with a disciplined mapping from ordered history to a future target. Once that mapping is correct, regression and classification methods can be tested honestly, and their limits become easier to see.

---

Published 2026-11-03.

**Source:** Python for algorithmic trading - from idea to cloud -- Yves J Hilpisch, section **Chapter 5. Predicting Market Movements with Machine Learning**.

This is an original explanatory note; the source book is not redistributed.
