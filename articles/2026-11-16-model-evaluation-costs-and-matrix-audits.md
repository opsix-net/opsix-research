# When a Model Is Right, What Does That Cost?

A fraud model labels 99 out of every 100 transactions correctly. That sounds excellent, until you learn that only one transaction in a hundred is fraudulent and the model calls every transaction genuine. Its accuracy is 99 percent. Its ability to catch fraud is zero.

That mismatch is the practical heart of model evaluation: a score is only useful when it matches the decision someone needs to make. The selected passage from *Practitioners Guide to Data Science* discusses classification and regression metrics, then sketches a model-development workflow. Despite the section heading, the excerpt does not explain a clustering algorithm. Its focus is supervised prediction, where known test outcomes let us compare predictions with actual results.

This article develops that focus with a worked example, clarifies what common metrics do and do not say, and connects evaluation to a small exact-arithmetic matrix utility. The utility is useful for checking linear-algebra inputs in research and engineering workflows, but it is not a model-scoring tool.

## Start with the decision, not the score

A classification model assigns cases to categories, often by estimating a probability. A threshold converts that probability into a decision. For a binary classifier, suppose we call a case positive when its estimated probability is at least $t$. Changing $t$ changes which cases are predicted positive, so it can change every count in the confusion matrix.

The four counts are defined relative to a chosen positive class:

- True positive (TP): a positive case predicted positive.
- False positive (FP): a negative case predicted positive.
- True negative (TN): a negative case predicted negative.
- False negative (FN): a positive case predicted negative.

These names are not abstract grades. They describe outcomes with different consequences. In screening for a dangerous condition, a false negative might be especially costly. In deciding who should receive a treatment with substantial risks, a false positive may deserve more attention. The appropriate operating point depends on the decision, resources, and error costs, not on a universal rule that one metric must always win.

## A checked classification example

Imagine a maintenance model evaluated on 100 elevator inspections. For this example, define “positive” as “maintenance is needed.” The test labels and predictions yield $TP=18$, $FP=6$, $TN=70$, and $FN=6$. The counts add to 100: $18+6+70+6=100$.

Precision asks: among cases predicted positive, how many were truly positive?

$$\text{Precision}=\frac{TP}{TP+FP}=\frac{18}{18+6}=\frac{18}{24}=0.75.$$

Thus, 75 percent of the maintenance alerts in this sample correspond to actual maintenance needs. Recall asks: among all actual positive cases, how many did the model catch?

$$\text{Recall}=\frac{TP}{TP+FN}=\frac{18}{18+6}=\frac{18}{24}=0.75.$$

The model caught three quarters of the cases that needed maintenance. The false positive rate is the fraction of actual negatives incorrectly called positive:

$$\text{FPR}=\frac{FP}{FP+TN}=\frac{6}{6+70}=\frac{6}{76}\approx0.0789.$$

That is about 7.9 percent, not “specificity.” Specificity is the true negative rate, $TN/(TN+FP)=70/76\approx0.9211$. The excerpt labels FP divided by total negatives as “specificity,” but that expression is conventionally the false positive rate. Keeping these names distinct prevents a consequential reporting error.

The F1 score is the harmonic mean of precision and recall:

$$F_1=2\frac{\text{Precision}\cdot\text{Recall}}{\text{Precision}+\text{Recall}}=2\frac{0.75\cdot0.75}{0.75+0.75}=0.75.$$

Accuracy counts all correct predictions:

$$\text{Accuracy}=\frac{TP+TN}{TP+FP+TN+FN}=\frac{18+70}{100}=0.88.$$

So accuracy is 88 percent. None of these numbers tells us by itself whether the maintenance policy is worthwhile. We would also want to know the consequences and costs of missed faults, unnecessary visits, and the model’s performance on representative future data.

## Thresholds change the operating point

A probability model can be evaluated at many thresholds. Lowering the threshold generally makes positive predictions more common. That may increase recall, while also increasing false positives. Raising it may reduce unnecessary alerts but miss more true cases. The exact changes depend on the data and model; they are not guaranteed to move by a fixed amount.

A receiver operating characteristic (ROC) curve plots true positive rate against false positive rate across thresholds. Here, true positive rate is recall, $TP/(TP+FN)$, and false positive rate is $FP/(FP+TN)$. The area under the ROC curve (AUC) summarizes ranking performance across thresholds. It does not select a threshold for a particular operation, establish calibration of predicted probabilities, or account automatically for the costs and prevalence of errors. A curve can compare models from one perspective, but a deployment decision still needs context.

The passage’s examples make this point in different settings: missed bad credit customers, false claims about treatment benefit, and both kinds of maintenance error can carry unequal costs. Its general lesson is sound: choose evaluation criteria in light of the use case. In a rare-event problem, accuracy can be especially misleading because predicting the common class for every case may still produce a high score.

## Regression errors need units and context

Regression predicts a continuous quantity. For observation $i$, the residual is

$$r_i=\hat{y}_i-y_i,$$

where $\hat{y}_i$ is the prediction and $y_i$ is the observed value. A positive residual means overprediction under this convention; a negative residual means underprediction. The residual sign matters for diagnosis, but summing signed residuals can hide errors because they cancel.

Several metrics summarize residuals in different ways. For $n$ observations:

$$\text{MSE}=\frac{1}{n}\sum_{i=1}^{n}r_i^2,\qquad \text{RMSE}=\sqrt{\text{MSE}},$$

$$\text{MAE}=\frac{1}{n}\sum_{i=1}^{n}|r_i|.$$

MSE heavily penalizes large errors and has squared units. RMSE returns to the target’s units, but still gives large residuals extra weight. MAE is also in the target’s units and is less sensitive to large individual errors than squared-error metrics. Median absolute error is the median of $|r_i|$ and is robust to some outliers, though it can hide a costly tail of rare errors. Comparing MAE with median absolute error can suggest an uneven error distribution, but does not prove why it arose.

Consider actual values $[10,12,20]$ and predictions $[12,9,19]$. Residuals are $[2,-3,-1]$. Their absolute values are $[2,3,1]$, so $\text{MAE}=(2+3+1)/3=2$. Squared residuals are $[4,9,1]$, giving $\text{MSE}=14/3\approx4.667$ and $\text{RMSE}=\sqrt{14/3}\approx2.160$. Sorted absolute errors are $[1,2,3]$, so median absolute error is 2. These calculations are descriptive summaries of this sample, not evidence that future errors will match.

The excerpt’s account of $R^2$ needs care. A common definition with an intercept is $R^2=1-\text{SSE}/\text{SST}$, where $\text{SSE}=\sum_i(y_i-\hat{y}_i)^2$ and $\text{SST}=\sum_i(y_i-\bar{y})^2$. It compares squared prediction errors with deviations from the observed mean. It is not generally one minus squared-error sum divided by the sum of squared raw outcomes, as the passage describes. Values and interpretation can vary by evaluation setup; $R^2$ can be negative on test data when predictions are worse than the mean baseline.

Adjusted $R^2$ penalizes model complexity in a specific regression setting. For ordinary least squares with an intercept, a common form is $1-(1-R^2)(n-1)/(n-p-1)$, with $p$ predictors and enough observations for the denominator to be positive. It is not a universal guarantee of generalization. Test-set evaluation, suitable validation design, and attention to distribution shift remain important.

## Evaluation belongs inside a workflow

The passage places metrics after prerequisites such as understanding the requirement, acquiring and exploring data, and preparing it for modeling. It then describes preprocessing, choosing relevant algorithms, splitting data into training and test sets, fitting, evaluating, and tuning. In practice, preserve the test set for a final assessment rather than repeatedly tuning against it. Repeated inspection can turn that test set into another training signal.

Document the positive class, threshold, metric definitions, data split, and relevant costs. For regression, inspect residuals as well as aggregate metrics, and check whether errors differ across meaningful operating conditions. A single average can conceal a bad subgroup or a failure concentrated at the extremes. Metrics describe observed evaluation data under their assumptions; they do not establish causation, fairness, safety, or future performance on their own.

## A matrix check in the engineering toolbox

A model workflow can also involve matrices: design matrices, calibration systems, sensor transformations, or numerical constraints. Before trusting a downstream calculation, it is useful to know whether a matrix has full rank, which columns are pivots, and, for a square matrix, its determinant. The `matrix-audit` utility computes those properties exactly for integer and rational-string inputs; JSON fractional numbers are parsed as floats first. It can help catch a dependent column or singular system before later numerical work.

That is a separate diagnostic from predictive evaluation. Matrix rank does not measure classifier quality, and a nonzero determinant does not validate a scientific model. The connection is workflow discipline: use a tool that answers the question you actually have, and state what its answer does not establish.

## Assumptions, pitfalls, and practice

Classification metrics require defined labels and a specified positive class. Precision is undefined when there are no predicted positives; recall is undefined when there are no actual positives. Regression summaries require paired predictions and observations on a meaningful scale. Missing values, nonfinite values, data leakage, unrepresentative test sets, and changing operating conditions can all undermine interpretation. The source passage supplies a useful overview, but its formulas should be checked against standard definitions, as the false-positive-rate and $R^2$ details illustrate.

Try these short exercises:

1. If $TP=8$, $FP=2$, $FN=4$, and $TN=86$, what are precision and recall? **Answer:** $8/10=0.8$ and $8/12\approx0.667$.
2. If all actual positives are rare, why might accuracy mislead? **Answer:** A classifier can predict the common negative class for every case and still be accurate overall while missing every positive.
3. If a system’s false negatives are more costly than false positives, what should guide threshold selection? **Answer:** Compare threshold-specific outcomes against the operational costs and constraints, including recall and the resulting false-positive burden.

A good evaluation is not the biggest number on a dashboard. It is a transparent account of what was measured, on which data, under which decisions, and with what known limitations.

---

Published 2026-11-16.

**Source:** PRACTITIONERS GUIDE TO DATA SCIENCE - streamlining data -- Mirza, Nasir Ali;, section **8 A clustering algorithm**.

This is an original explanatory note; the source book is not redistributed.
