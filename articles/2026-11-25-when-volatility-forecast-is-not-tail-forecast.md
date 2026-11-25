# When a Volatility Forecast Is Not a Tail Forecast

A risk model reports tomorrow’s volatility as 2 percent. You need a five-day loss threshold. Is multiplying 2 percent by the square root of five enough? Only under assumptions that can fail precisely when risk matters most.

A useful discussion in *Practical Financial Econometrics* connects this forecasting problem to backtesting: simulate multi-day returns from a fitted GARCH model, estimate a tail quantile from those simulated returns, and test the resulting exceedances. It also explains why a coverage test has two distinct jobs. It checks whether the number of exceptions is plausible, and whether exceptions arrive independently. Those are related questions, but one cannot substitute for the other.

This article builds the intuition, works through a small coverage example, and follows the workflow into multivariate forecasts and covariance checks.

## A volatility path is not a return distribution

Value at Risk (VaR) is a quantile of a loss distribution over a specified horizon. A one-day VaR forecast and a five-day VaR forecast are not interchangeable: the latter concerns the distribution of the sum of five returns, not merely a rescaled one-day standard deviation.

In a GARCH model, conditional variance changes as new returns arrive. A large shock can raise volatility, which then affects later returns in the same forecast horizon. If we replace the future variance path with a single expected variance forecast, we suppress this pathwise feedback. That effectively treats future innovations as typical rather than allowing a large innovation to trigger a volatility response.

There is a second issue. Even if each daily innovation is conditionally normal, the multi-day return need not be normal. Its distribution combines returns generated at different, random conditional volatility levels. A normal quantile applied to an analytic multi-day standard deviation therefore need not be the correct tail quantile. The key distinction is between forecasting a variance summary and forecasting the distribution of a sum under evolving volatility.

Simulation retains that evolution. Fit the model on an estimation window, generate many possible return paths over the risk horizon, sum each path, then estimate the desired quantile from the simulated sums. Repeating this as the estimation window moves forward produces a series of forecasts that can be compared with observed horizon returns.

The source section lays out this rolling procedure for post-sample testing: fit on daily log returns, simulate the next $h$ days repeatedly, sum each simulated path, estimate the required quantile, advance the window by $h$ days, and continue. Finally, compare the realized $h$-day returns with the forecasts, record exceedances, and test their sequence. A multivariate extension simulates correlated asset returns, computes each portfolio return using fixed weights, and estimates the quantile from the resulting portfolio distribution. This can be computationally demanding, especially for complicated portfolios.

## Two questions hidden inside a coverage test

Suppose a forecast is designed for a lower-tail probability $p$. Define an exceedance indicator $I_t$ to be 1 when the realized return falls below its forecast threshold, and 0 otherwise. Under correct unconditional coverage, the long-run exception probability should be $p$. This is a frequency claim: over many forecasts, does the observed exception rate agree with the nominal rate?

But frequency alone does not describe timing. If exceptions cluster, the model may miss changes in risk even when the total count is reasonable. An independence test asks whether the exception sequence behaves like independent Bernoulli trials, or instead shows dependence such as autocorrelation.

The conditional coverage likelihood-ratio test combines these two checks. In the notation of the passage,

$$
LR_{cc}=LR_{uc}LR_{ind}.
$$

Taking negative twice the natural logarithm turns the product into a sum:

$$
-2\ln LR_{cc}=-2\ln LR_{uc}-2\ln LR_{ind}.
$$

Consequently, the independence component can be recovered by subtracting the unconditional-coverage statistic from the conditional-coverage statistic. Under the test’s null and its usual large-sample reference approximation, $-2\ln LR_{ind}$ is compared with a chi-squared distribution having one degree of freedom. The passage emphasizes a practical outcome: conditional coverage can fail because exceptions are dependent even when the exception rate itself passes its test. A model can get the count roughly right and still get the pattern wrong.

This decomposition is useful for diagnosis, not just a pass/fail label. A coverage failure driven by too many exceptions suggests a different issue from a failure driven by clustering. Neither result alone proves why the model failed, and a test’s conclusion depends on the sample and its assumptions.

## Worked example: plausible count, suspicious ordering

Consider a deliberately small illustrative sequence of 10 forecasts with a nominal exception probability $p=0.2$. Suppose the observed indicators are

$$
(1,1,0,0,0,0,0,0,1,0).
$$

There are three exceptions, so the observed rate is $3/10=0.3$. For an unconditional-coverage likelihood calculation, let $\hat p=3/10$. The log likelihood under the fitted exception probability is

$$
\ell(\hat p)=3\ln(0.3)+7\ln(0.7)\approx -6.1086.
$$

Under the null rate $p=0.2$ it is

$$
\ell(p)=3\ln(0.2)+7\ln(0.8)\approx -6.3903.
$$

Thus the likelihood-ratio statistic is

$$
LR_{uc}=-2[\ell(p)-\ell(\hat p)]\approx -2[-6.3903-(-6.1086)]=0.5634.
$$

This arithmetic illustrates the likelihood comparison; with only 10 observations, it is not a persuasive basis for a production conclusion. More importantly, the first two observations are both exceptions. The count-based statistic does not encode that ordering. An independence component uses transition information in the sequence, such as how often exceptions follow exceptions versus non-exceptions, and compares an independent Bernoulli model with a model allowing different transition probabilities. If clustering makes the independence component large, the combined conditional-coverage statistic can reject even though the unconditional component does not. The exact outcome requires evaluating the transition likelihoods; the observed ordering alone is not a test result.

This distinction is easy to overlook when a dashboard shows only an exception percentage. Preserve the time series of binary outcomes. Without ordering, an independence assessment is impossible.

## Multivariate forecasts: covariance is part of the path

For a portfolio with asset-return vector $r_t$ and fixed weights $w$, the portfolio return is $w r_t$. If a multivariate GARCH model supplies a time-varying covariance matrix $H_t$, its implied portfolio variance is $w H_t w^{\mathsf T}$ when returns are represented as a column vector and weights as a row vector (equivalently, use the corresponding transposed convention consistently).

The source section notes that correlation is not directly observable as a single model-free truth. A correlation forecast is conditional on the model used to produce it. In-sample likelihood and penalized likelihood criteria such as AIC or BIC provide ways to compare multivariate GARCH specifications; direct autocorrelation-function checks are difficult because the theoretical functions are not generally easy to derive.

One practical summary is to apply each estimated covariance matrix to a standard portfolio, for example equal weights. The resulting variance series can be compared with the portfolio’s realized behavior as a way to examine whether the model captures changing volatility. This compresses a large covariance forecast into a scalar series, but it does not validate every covariance entry or every portfolio. Portfolio choice matters.

For tail prediction, simulating correlated asset paths and then applying portfolio weights preserves the model’s joint return dynamics. Using one covariance forecast directly in a quantile formula can miss the impact of unusually large returns during the horizon. Simulation costs more, but it answers a richer distributional question.

## Assumptions and practical traps

A defensible backtest needs careful alignment. A forecast made at time $t$ must be compared with the return over precisely the horizon it predicts. Rolling windows advanced by $h$ days, as in the described procedure, avoid treating overlapping horizon returns as if they were independent observations. If a different design uses overlapping returns, dependence in the evaluation sample needs explicit consideration.

Simulation error is another source of uncertainty. A finite number of paths gives an estimated quantile, not an exact one. Tail quantiles are especially sensitive to the number and quality of simulated paths. Record the random-number setup and simulation count, and check whether results stabilize as the count increases. The passage gives 10,000 simulations as an example, not a universal guarantee.

Coverage tests also rely on the exception definition, forecast timing, and test assumptions. Exceptions can be rare, leaving little information for an independence test. Passing a test is not proof that a model is correct; failing one does not identify a unique repair. Review calibration, sequence behavior, data quality, and model specification together.

Finally, distinguish model-based simulation from an analytic shortcut. If the model’s conditional variance evolves after shocks, a shortcut that inserts a single horizon variance into a normal quantile formula discards that mechanism. The shortcut may be convenient, but convenience does not make its distributional assumptions true.

## A useful matrix check before a portfolio simulation

Multivariate workflows depend on matrix structure. A covariance matrix should be square, and a singular matrix can indicate redundant asset exposures or a numerical/data construction problem. Exact rank checks on a small rational-valued matrix can help verify a hand-built example or test fixture before it enters a larger numerical pipeline. They do not establish that an estimated covariance is statistically appropriate, nor do they replace checks such as symmetry and positive semidefiniteness.

A small audit utility can report exact rank, pivot columns, and, for square matrices, determinant. Exact arithmetic is valuable for controlled examples because it avoids floating-point ambiguity about whether a pivot is zero. For real-valued estimated covariance matrices, floating-point tolerances and statistical diagnostics are still necessary.

## Exercises with short answers

1. A model has the correct total number of exceptions, but they occur in long runs. Which component of conditional coverage is most directly implicated? **Answer:** The independence component, because it tests the exception sequence rather than only its marginal rate.

2. Why can conditionally normal daily GARCH returns produce a non-normal multi-day return? **Answer:** Daily conditional volatility changes along the path, so the sum combines returns with changing, random conditional scales.

3. Does a full-rank covariance matrix prove that its forecasts are good? **Answer:** No. Rank is a structural property, not a test of forecast calibration or model validity.

The central lesson is simple: forecast the distribution that the model actually implies, then evaluate both how often tail events occur and how they are arranged in time. That turns a volatility number into a testable risk workflow.

---

Published 2026-11-25.

**Source:** Market Risk Analysis [vol 2] - Practical Financial Econometrics.pdf, section **360 Practical Financial Econometrics**.

This is an original explanatory note; the source book is not redistributed.
