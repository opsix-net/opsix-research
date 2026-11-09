# Reading Regression Tests Without Fooling Yourself

A regression report says one predictor matters, another does not, and the model explains 86 percent of the outcome's variation. What should you do next? It is tempting to turn those lines into a simple story: keep the significant predictor, discard the other, and trust the model. But a coefficient test answers a narrower question than that. To use regression responsibly, you need to understand the test, the model behind it, and the relationships among the predictors.

This article unpacks three connected ideas: two-sided tests for regression coefficients, the equivalent p-value decision, and multicollinearity. It also shows why an individual test is not a verdict on a whole model, and how those ideas translate into a careful analysis workflow.

## What a coefficient test asks

In a multiple linear regression, a coefficient describes the model's estimated change in the response associated with a one-unit change in a predictor, while the other included predictors are held fixed. For predictor $X_j$, the usual two-sided test asks whether its population coefficient could be zero:

$$
H_0: \beta_j = 0 \qquad\text{versus}\qquad H_1: \beta_j \ne 0.
$$

The test statistic is usually

$$
t = \frac{\hat{\beta}_j - 0}{SE(\hat{\beta}_j)},
$$

where $\hat{\beta}_j$ is the fitted coefficient and $SE(\hat{\beta}_j)$ is its estimated standard error. The statistic measures how many estimated standard errors separate the coefficient from the null value. A large positive or negative value is evidence against zero, provided the model's assumptions and the test's reference distribution are appropriate.

For a two-sided test at significance level $\alpha=0.05$ with 4 residual degrees of freedom, the positive critical value is about 2.776. The corresponding lower cutoff is -2.776 because the t distribution is symmetric around zero. The decision rule is to reject $H_0$ when $t>2.776$ or $t<-2.776$. Otherwise, do not reject it. The degrees of freedom matter: this cutoff is not a universal threshold for regression coefficients.

The source section uses this rule to interpret three coefficients. Its reported statistics are 3.512, 3.238, and 0.874. The first two exceed 2.776, so their null hypotheses are rejected in the stated test. The third lies between -2.776 and 2.776, so its null hypothesis is not rejected. This is evidence about the coefficients in that fitted model. It is not proof that a predictor causes the response, or proof that a nonsignificant predictor has no relationship in every setting.

## The p-value is another route to the same decision

A p-value is the probability, assuming the null hypothesis and the test model are correct, of obtaining a test statistic at least as extreme as the observed one. For a two-sided t-test, “as extreme” means at least as far from zero in either direction. At a 0.05 significance level, reject the null when the p-value is below 0.05; otherwise, do not reject it.

The reported p-values for the three statistics are 0.0246, 0.0317, and 0.431. The first two are below 0.05 and the third is above it, giving the same decisions as the critical-value method. This agreement is expected when both methods use the same test, degrees of freedom, and significance level. A p-value is not the probability that the null hypothesis is true, and 0.0246 does not mean there is a 2.46 percent chance the finding is “random.” It describes data extremeness under a particular null model.

A significance threshold is a decision convention, not a boundary between important and unimportant effects. A small effect can be statistically distinguishable from zero with sufficiently precise data; a practically important effect can remain uncertain if the estimate is noisy. Read the coefficient and its units, uncertainty interval, and practical context alongside the test result.

## A worked example, from estimate to decision

Suppose a lab models daily instrument drift, in micrometers, using room temperature as one predictor. The fitted temperature coefficient is $\hat{\beta}=0.42$ micrometers per degree, and its standard error is 0.12. Assume the model has 4 residual degrees of freedom, and the question is whether the coefficient differs from zero. These are illustrative values, not figures from the source section.

First calculate the statistic:

$$
t = \frac{0.42-0}{0.12} = 3.5.
$$

For a two-sided 5 percent test with 4 degrees of freedom, compare $|3.5|$ with 2.776. Since $3.5>2.776$, reject the zero-coefficient null at this threshold. Equivalently, the two-sided p-value is about 0.024, which is below 0.05. The arithmetic is consistent: the statistic is positive, its magnitude clears the critical value, and the p-value gives the same decision.

The careful conclusion is that, under the fitted model and its assumptions, the data provide evidence that the temperature coefficient differs from zero. The estimated association is an increase of 0.42 micrometers per degree, conditional on the other predictors. This test alone does not establish that changing the room temperature will cause drift to change by that amount. Nor does it tell us whether 0.42 micrometers per degree is operationally important. Those questions need context and, for causal interpretation, a design that supports causal claims.

## Why correlated predictors complicate the story

Multicollinearity occurs when two or more predictors in a multiple regression carry overlapping information because they are strongly related. For example, a model that includes both a sensor's raw temperature and a near-duplicate smoothed temperature may struggle to separate their individual contributions. The model can often estimate their combined relationship with the response more reliably than it can attribute that relationship to each predictor separately.

That difficulty commonly appears as larger standard errors for affected coefficient estimates. Since the t statistic divides the estimate by its standard error, a larger standard error can make an individual statistic smaller and its p-value larger. A predictor may therefore look statistically inconclusive even when the model as a whole predicts well, or when the predictors collectively contain useful information. This is not evidence that the variable is irrelevant. It is evidence that the data and model may not identify its separate contribution precisely.

The source passage points to a high $R^2$ alongside individually nonsignificant t-tests as a warning sign, and gives $R^2>0.8$ as an indicator. Treat that cutoff as a heuristic, not a diagnostic law. High $R^2$ can occur without severe multicollinearity, and collinearity can occur with a lower $R^2$. The pattern is a reason to investigate predictor relationships, not a proof of a particular problem.

Dropping one of a correlated pair is one possible remedy, but it is not an automatic cure. Removing a predictor changes the meaning of the remaining coefficients and can introduce omitted-variable bias if the removed variable belongs in the model. P-values may change, but they are not guaranteed to decrease in a particular way. Alternatives include collecting data with more independent variation, combining predictors when justified by the science, or using a method designed for prediction with correlated inputs. Choose based on the analysis goal, not just on which variables cross a threshold.

## A practical review before acting

When a regression output looks decisive, work through a short review:

1. **Check the question.** Is the goal explanation, prediction, or a causal estimate? A coefficient test directly addresses a model-based association, not all three goals equally.
2. **Check the estimate and uncertainty.** Record the coefficient's units, standard error, and interval. Ask whether plausible values are meaningful in the application.
3. **Check the test setup.** Confirm whether the test is one-sided or two-sided, the chosen significance level, and the degrees of freedom. Do not reuse a critical value from a different setup.
4. **Check the predictors together.** Look for strong relationships among inputs and consider whether the data can distinguish their effects. Review residual behavior and model fit as well as individual p-values.
5. **Revisit the model over time.** In operational settings, populations and processes can change. A model that once described a system well may need fresh validation or refitting.

Regression software handles many calculations, but it cannot decide whether a predictor is a meaningful measurement, whether an assumption fits the science, or whether a model will remain useful after conditions change. Those are engineering and research judgments.

## Applications beyond salary models

The same reasoning applies to scientific and operational work. A researcher may test whether a treatment-dose coefficient differs from zero while adjusting for baseline measurements. A reliability engineer may examine whether operating temperature is associated with failure time while controlling for load. An analyst may test whether a process setting is associated with defect rate. In each case, a significant coefficient is conditional on the included model, data, and assumptions; it does not automatically establish causality or practical value.

When predictors overlap, coefficient-level interpretation deserves particular care. If two environmental measurements track each other closely, the model may not have enough independent information to assign their shared association cleanly. Better measurement design or more varied observations may be more informative than simply removing whichever variable has the larger p-value.

## Assumptions and common pitfalls

The familiar t-test for a regression coefficient depends on the regression model and its uncertainty calculation. In the classical linear model, the usual exact small-sample inference relies on assumptions including a correctly specified linear mean structure, independent errors with constant variance, and normally distributed errors. Other methods can change some requirements, but they must be justified and reported. With only 4 residual degrees of freedom, results can be especially sensitive to individual observations and assumptions.

Avoid these common errors:

- Treating “do not reject” as proof that the coefficient is exactly zero.
- Treating a small p-value as a measure of effect size or importance.
- Interpreting a conditional regression coefficient as a causal effect without an appropriate design.
- Assuming a high $R^2$ proves multicollinearity, or that multicollinearity always makes the whole model useless.
- Deleting predictors only to obtain smaller p-values, without considering scientific rationale and changes in coefficient meaning.
- Treating a one-time fit as permanently valid when the data-generating process can change.

## Exercises with short answers

1. A coefficient has $t=-3.0$ with 4 degrees of freedom in a two-sided test at $\alpha=0.05$. What is the decision? **Answer:** Reject the zero-coefficient null because $-3.0<-2.776$.
2. A coefficient has a p-value of 0.08 at the 0.05 level. Has the analysis proved that the coefficient is zero? **Answer:** No. It has not provided enough evidence to reject zero at that threshold.
3. A model has high $R^2$, but two related predictors have large standard errors. What should you conclude? **Answer:** Investigate collinearity and model assumptions. The pattern is a warning, not proof that either predictor is useless.

A regression report is a starting point for reasoning, not a replacement for it. Critical values and p-values summarize evidence under a model. Coefficients describe conditional relationships. Collinearity warns when the model cannot cleanly separate overlapping predictors. Put those pieces together, and the output becomes more than a collection of stars beside variable names.

---

Published 2026-11-09.

**Source:** Statistics for Big Data For Dummies, section **2.764 3.169**.

This is an original explanatory note; the source book is not redistributed.
