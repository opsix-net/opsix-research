# A Small P-Value Is Not the Whole Result

Imagine a tutoring program raises students’ average quiz scores by two points. With enough students, a statistical test might flag that difference as unlikely under a model of equal group means. But is two points meaningful? How uncertain is the estimate? And what if scores are lopsided or tied? A p-value alone cannot answer all three questions.

The selected section of *Math for Deep Learning: What You Need to Know to Understand* brings together tests of group differences, confidence intervals, effect size, and rank-based comparisons. Its central practical lesson is valuable: statistical evidence and practical importance are related, but they are not interchangeable. To use that lesson well, we also need to be precise about what each method tests and what assumptions it needs.

## Start with the question, not the test

A two-sample t-test asks whether the population means of two groups differ, subject to the test’s assumptions. Welch’s version allows the groups to have unequal variances. It is often a sensible default for independent groups because it does not require equal population variances.

For observations $X_1,\ldots,X_{n_1}$ and $Y_1,\ldots,Y_{n_2}$, Welch’s statistic is

$$
t = \frac{\bar X-\bar Y}{\sqrt{s_X^2/n_1+s_Y^2/n_2}},
$$

where $\bar X$ and $\bar Y$ are sample means, and $s_X^2$ and $s_Y^2$ are sample variances. The denominator estimates the standard error of the difference. The statistic measures the difference in units of that estimated uncertainty. A large absolute value is evidence against the null hypothesis that the population means are equal, given the model and assumptions.

Welch’s test uses an approximate degrees-of-freedom calculation, often called the Welch-Satterthwaite approximation:

$$
\nu = \frac{(s_X^2/n_1+s_Y^2/n_2)^2}{(s_X^2/n_1)^2/(n_1-1)+(s_Y^2/n_2)^2/(n_2-1)}.
$$

The selected passage presents this same calculation as part of its confidence-interval function. The degrees of freedom need not be an integer. A two-sided confidence interval at confidence level $1-\alpha$ is

$$
(\bar X-\bar Y) \mathbin{\pm} t_{1-\alpha/2,\nu}\sqrt{s_X^2/n_1+s_Y^2/n_2},
$$

where $t_{1-\alpha/2,\nu}$ is the corresponding quantile of a t distribution. At a 95% confidence level, $\alpha=0.05$, so the quantile is the 97.5th percentile. The interval describes uncertainty in the estimated difference under the model. If it excludes zero, it corresponds to rejecting a two-sided equal-means null at the matching significance level, subject to the same assumptions. It does not show that every value inside the interval is equally plausible, nor that the effect is important.

## Worked example: turn a test into an interpretable result

Suppose two independent teams try different ways to reduce the time needed to complete a data-cleaning task. A lower time is better. Team A has 25 measured sessions with mean 42 minutes and sample standard deviation 8 minutes. Team B has 30 sessions with mean 48 minutes and sample standard deviation 10 minutes. Treat the session times as independent observations and use Welch’s method. These are summary statistics for an illustration, not a claim about a real study.

Define the difference as A minus B. The estimated difference is

$$
42-48=-6\text{ minutes}.
$$

The estimated standard error is

$$
\sqrt{8^2/25+10^2/30}
=\sqrt{64/25+100/30}
=\sqrt{2.56+3.3333\ldots}
\approx 2.4276\text{ minutes}.
$$

Thus $t\approx -6/2.4276=-2.472$. The Welch degrees of freedom are approximately

$$
\nu=\frac{(2.56+3.3333\ldots)^2}{2.56^2/24+3.3333\ldots^2/29}\approx 52.9.
$$

For about 52 degrees of freedom, the 97.5th-percentile t critical value is about 2.006. The 95% interval is therefore

$$
-6\mathbin{\pm}2.006(2.4276)\approx [-10.87,-1.13]\text{ minutes}.
$$

The interval excludes zero, so the data are inconsistent with equal population means at the two-sided 5% level under this model. The estimate suggests Team A is faster, but the interval also reminds us that the size of the advantage is uncertain. Whether even the lower end of the benefit is useful depends on the project. For example, teams might care about a minimum worthwhile reduction, implementation costs, or effects on error rates. Statistical significance does not settle those decisions.

## Put effect size beside uncertainty

The section introduces Cohen’s $d$, which expresses a mean difference in units of a pooled scale. In the version shown there, the denominator is the square root of the average of the two sample variances:

$$
d=\frac{\bar X-\bar Y}{\sqrt{0.5(s_X^2+s_Y^2)}}.
$$

For the task-time example, the denominator is $\sqrt{0.5(64+100)}=\sqrt{82}\approx9.055$. Therefore $d=-6/9.055\approx-0.66$. The sign records the chosen subtraction order; the magnitude is about 0.66 pooled standard deviations.

This version of $d$ is a descriptive standardized difference. It is not a direct measure of value to a user, and labels such as “small,” “medium,” or “large” are rough conventions, not universal thresholds. A standardized effect can help compare results measured on different scales, but it can obscure the original units. Report the raw difference and its interval alongside it. Also name the formula used: several Cohen’s $d$ variants exist, and alternatives can use sample-size-weighted pooled variance or small-sample corrections.

The section’s test-score example illustrates the same separation: its reported p-value is below 0.05, its confidence interval excludes zero, and it reports a standardized effect. Those are complementary descriptions, not three independent guarantees of practical importance. A p-value is the probability, under a specified null model and assumptions, of observing a result at least as extreme as the one obtained. It is not the probability that the null hypothesis is true, and it does not quantify the chance that a result will replicate.

## Rank tests answer a different question

When data are strongly non-normal, contain outliers, or are naturally ordinal, a rank-based method may be useful. The Mann-Whitney U test pools observations, ranks them, and compares the rank totals between groups. Tied observations receive average ranks. It does not depend on the original measurement units in the way a mean-based test does.

The selected passage describes the null in terms of the chance that a randomly chosen value from one group exceeds a randomly chosen value from the other being 0.5. This is a useful intuition when the groups have comparable distributions and ties are absent. More generally, the Mann-Whitney procedure concerns relative ordering, and its interpretation depends on distributional conditions. It is not simply a universal test of whether two groups have different means. A difference in spread or shape can also affect its result. With ties, equality needs careful treatment; for continuous identical distributions, the pairwise probability of a win is 0.5, while discrete data may have ties with nonzero probability.

Calling Mann-Whitney “nonparametric” does not mean assumption-free. Independent observations and an appropriate sampling design still matter. Nor does ranking automatically make a test robust to every problem. In particular, dependence between observations, selection bias, or a badly chosen comparison can invalidate an otherwise correct calculation.

The passage notes a minimum group size of 21 as a caution for the SciPy use it discusses. That should not be treated as a universal law for every implementation or data set. Exact and approximate p-value methods, ties, sample sizes, and the design all affect calibration. Check the documentation for the software version and options you use, and do not treat a sample-size rule as proof of reliability. When the scientific question is about medians or a location shift, make sure the assumptions needed for that interpretation are justified. The test itself is based on ranks, not a direct comparison of sample medians.

## A practical analysis workflow

1. **Write down the estimand.** Is the target a difference in means, a shift in ordering, or a practically important threshold?
2. **Check the design.** Are the groups independent, or are observations paired or repeated? An independent-samples test is not a substitute for a paired analysis.
3. **Inspect the data.** Look for errors, outliers, skew, ties, missing values, and unusual group differences. A histogram or dot plot can reveal things a single summary cannot.
4. **Choose a method that matches the question.** Use Welch’s t-test for independent-group mean comparisons when unequal variances are plausible. Consider rank methods when their estimand and assumptions fit, not simply because a normality test produced a small p-value.
5. **Report magnitude and uncertainty.** Give group summaries, the raw difference, a confidence interval, a suitable effect-size measure, and the test result. Explain the direction and units.
6. **Interpret in context.** Compare plausible effects with a predeclared or substantively justified threshold. Account for multiple comparisons and the study design when relevant.

In scientific research, this workflow can help compare assay conditions, device configurations, or model evaluation scores. In software engineering, it can help evaluate build times, query latency, or memory use. But repeated measurements from the same machine, dataset, or participant are often correlated. Treating them as independent can make uncertainty look smaller than it is. A sound design and analysis plan matter more than a fashionable test name.

## Common traps, and a few checks

- **“The p-value is below 0.05, so the effect matters.”** No. Magnitude, uncertainty, costs, and the application’s threshold matter too.
- **“The confidence interval excludes zero, so the result is robust.”** It establishes a result under a model and sampling assumptions. A wide interval may still allow substantially different practical conclusions.
- **“Mann-Whitney is the non-normal t-test.”** Not quite. Its rank-based null and interpretation differ from a test of equal means.
- **“A larger sample fixes a weak design.”** More observations do not repair confounding, dependence, measurement error, or biased sampling.
- **“A non-significant result proves no difference.”** It may reflect a small effect, noisy measurements, or limited sample size. Inspect the interval to see what effect sizes remain compatible with the data.

## Exercises

1. A 95% confidence interval for a mean difference is $[-1.2, 4.8]$. Does it reject a two-sided equal-means null at 5%? **Answer:** No. Zero is inside the interval. The data still allow both a small negative and a positive difference.
2. Two groups have a statistically significant difference, but the estimated change is smaller than the minimum useful improvement. Is it necessarily a successful intervention? **Answer:** No. Statistical evidence does not establish practical value.
3. Why might a rank test be preferable for an ordinal rating, and what should you still check? **Answer:** Ranks respect ordering without assuming equal spacing between rating levels. Check independence, ties, sampling design, and whether the rank-based question matches the scientific question.

The durable lesson is not “always use a t-test” or “switch to ranks when data look messy.” It is to keep the question, assumptions, uncertainty, and practical stakes in view at the same time. Tests can organize evidence. They cannot decide what counts as a worthwhile result.

---

Published 2026-11-24.

**Source:** Math for Deep Learning - What You Need to Know to Understand, section **0.8 Large**.

This is an original explanatory note; the source book is not redistributed.
