# From Scores to Decisions: Reading Classifier Metrics Without Fooling Yourself

A classifier can report a confident-looking number and still make the wrong operational choice. Imagine a screening system that assigns each case a probability, then labels every case above 0.5 as positive. That final label is convenient, but it hides two distinct questions: how well does the model rank cases, and what happens when we turn its scores into decisions?

Section 6.4.2.3 of *Applied Data Science Using PySpark* moves through that chain: model output, probability, cutoff, final prediction, and evaluation. It also introduces confusion matrices, threshold-based curves, and ranking summaries such as deciles and the Kolmogorov-Smirnov statistic. The useful lesson is not that one metric settles model quality. It is that each summary answers a different question, under particular assumptions.

## A score is not yet a decision

A binary classifier may first produce a raw score, often written as $\hat{y}$. For a logistic model, a sigmoid can map a real-valued score $z$ to a value between zero and one:

$$
\sigma(z) = \frac{1}{1 + e^{-z}}
$$

That value is commonly interpreted as a positive-class probability, provided the model and its calibration justify that interpretation. The sigmoid is increasing: a larger score produces a larger output. But it does not make the choice of decision threshold for us.

For example, $\sigma(0)=0.5$. A score of $22.45$ gives a value extremely close to one, while a score of $0.0000021$ gives approximately $0.500000525$. Rounded to one decimal place, both become 0.5 and 1.0 respectively. If the rule is “positive when probability is greater than or equal to 0.5,” both scores are positive. The equality matters: a probability exactly at the cutoff is assigned to the positive class under this rule.

A threshold is a policy choice as well as a modeling choice. Lowering it generally catches more positive cases, but can also create more false alarms. Raising it generally reduces positive predictions, but can miss more true positives. The costs of those mistakes depend on the application. A monitoring alert, a clinical screening workflow, and a marketing contact list should not automatically share one cutoff.

## Confusion matrix: count the decisions

Once predictions are binary, compare each prediction with its known actual label. For the positive class, the four outcomes are:

- **True positive (TP):** actual positive, predicted positive.
- **False positive (FP):** actual negative, predicted positive.
- **True negative (TN):** actual negative, predicted negative.
- **False negative (FN):** actual positive, predicted negative.

The matrix is a compact count of decisions, not a score-ranking summary. Its orientation must be checked: some tools put actual labels in rows and predictions in columns, while others use the reverse. The names TP, FP, TN, and FN are safer than assuming a particular visual layout.

From the counts, several familiar measures follow. Let $N=TP+FP+TN+FN$.

$$
\text{Accuracy} = \frac{TP+TN}{N}
$$

$$
\text{Misclassification rate} = \frac{FP+FN}{N} = 1-\text{Accuracy}
$$

$$
\text{Precision} = \frac{TP}{TP+FP}, \qquad
\text{Recall} = \frac{TP}{TP+FN}
$$

$$
F_1 = 2\frac{\text{Precision}\,\text{Recall}}{\text{Precision}+\text{Recall}}
$$

Precision asks what fraction of positive predictions were correct. Recall asks what fraction of actual positives were found. Specificity, the true negative rate, is $TN/(TN+FP)$. These ratios are undefined when their denominators are zero, so a reporting system needs an explicit convention for that case.

## Worked example: six cases, two thresholds

Suppose a model assigns these scores to six cases. For this example, treat the scores as probabilities and use positive label 1. Actual labels are known only for evaluation.

| Case | Score | Actual label |
|---|---:|---:|
| A | 0.90 | 1 |
| B | 0.80 | 0 |
| C | 0.70 | 1 |
| D | 0.40 | 1 |
| E | 0.30 | 0 |
| F | 0.10 | 0 |

At cutoff 0.50, scores at or above the cutoff are positive. Thus A, B, and C are positive predictions. A and C are true positives; B is a false positive. D is a false negative, while E and F are true negatives. Therefore $TP=2$, $FP=1$, $TN=2$, and $FN=1$, with $N=6$.

The arithmetic checks as follows:

- Accuracy: $(2+2)/6=4/6=2/3\approx0.667$.
- Misclassification rate: $(1+1)/6=2/6=1/3\approx0.333$, also $1-2/3=1/3$.
- Precision: $2/(2+1)=2/3\approx0.667$.
- Recall: $2/(2+1)=2/3\approx0.667$.
- $F_1=2(2/3)(2/3)/(2/3+2/3)=2/3\approx0.667$.
- Specificity: $2/(2+1)=2/3\approx0.667$.

Now lower the cutoff to 0.35. D changes from negative to positive, so it becomes a true positive. The counts are $TP=3$, $FP=1$, $TN=2$, and $FN=0$. Recall becomes $3/(3+0)=1$, while precision is $3/(3+1)=0.75$. Accuracy is $5/6\approx0.833$. This small example shows why a metric value is incomplete without its threshold and class definitions.

## Threshold curves answer a different question

A confusion matrix describes one cutoff. A receiver operating characteristic (ROC) curve evaluates many cutoffs, plotting true positive rate, or recall, against false positive rate. The false positive rate is $FP/(FP+TN)$, which equals $1-\text{specificity}$. The area under the ROC curve (ROC AUC) summarizes ranking across thresholds. A random ranking has expected ROC AUC of 0.5 under the usual binary setup; a perfect ranking has 1. AUC is not accuracy, and it does not select an operational cutoff.

A precision-recall (PR) curve plots precision against recall across thresholds. It is often informative when positive cases are relatively uncommon, because precision directly reflects the fraction of alerts that are correct. Its baseline depends on positive-class prevalence, unlike the ROC random baseline. The area or average-precision summary depends on the precise convention used by the software, so do not assume every implementation computes an identical geometric area.

The source section describes changing cutoffs to trace these curves. In practice, evaluate scores on data that was not used to fit or tune the model. Otherwise, the displayed performance can be optimistic. Also check whether the score column is the positive-class probability or an uncalibrated raw score. Ranking metrics can use scores without treating them as trustworthy probabilities.

## Deciles and KS: inspect the ranking in groups

A decile analysis sorts cases by score and partitions them into ten ranked groups. It can answer practical questions such as how many observed positives fall in the highest-scoring tenth. Ties need a stated handling rule: forcing tied scores into separate buckets can make group boundaries depend on ordering details. Small datasets may also produce groups that differ in size.

The Kolmogorov-Smirnov (KS) statistic compares cumulative distributions of positive and negative cases across score thresholds or ranked groups. A common presentation reports the largest absolute gap between cumulative positive and negative proportions. The sign depends on which cumulative distribution is subtracted from which, so report the convention. KS is a separation summary, not a test of causality or proof that the model will remain stable. A similar training and test value is encouraging evidence in that particular split, not a guarantee about future populations.

## Put the measures to work carefully

For reliability engineering, a team might compare alert thresholds using recall and precision, then inspect the confusion matrix to estimate missed incidents and unnecessary investigations. For scientific screening, recall may matter when missing a candidate is costly, but follow-up capacity can make precision important too. For outreach campaigns, deciles can help estimate how positives concentrate near the top of a ranked list. In every case, the operational cost and the quality of labels matter as much as the metric name.

A sound review records the positive class, threshold, evaluation dataset, confusion-matrix orientation, and metric definitions. It also checks class prevalence and compares results across meaningful groups or time periods. Accuracy alone can look high when a rare positive class is almost never predicted. AUC alone can look appealing while the chosen threshold produces an unacceptable number of false alarms. No single metric substitutes for inspecting the error counts and the workflow that follows a prediction.

## Assumptions and common pitfalls

These calculations assume a well-defined binary target and consistent labels. Precision and recall refer to a chosen positive class. Threshold metrics depend on the cutoff, while ROC and PR summaries aggregate behavior over thresholds. Probability interpretations require suitable model behavior, and good ranking does not guarantee calibration. Evaluation data should represent the intended use as closely as possible, with leakage avoided.

The passage’s small illustrative confusion matrix has three true positives, one false positive, no true negatives, and no false negatives. From those counts, accuracy is $3/4=0.75$, misclassification rate is $1/4=0.25$, precision is $3/4=0.75$, recall is $3/3=1$, and $F_1=2(0.75)(1)/(0.75+1)=0.857142\ldots$, approximately 0.86. Specificity cannot be calculated because there are no actual negatives. That is a useful reminder: a metric can be undefined even when software returns some chosen default.

## Exercises with short answers

1. A classifier has $TP=18$, $FP=6$, $TN=70$, and $FN=6$. What are accuracy, precision, and recall? **Answer:** Accuracy is $(18+70)/100=0.88$; precision is $18/24=0.75$; recall is $18/24=0.75$.
2. If a cutoff is lowered and one previously negative case becomes a positive prediction, which counts can change? **Answer:** Depending on its actual label, either FN decreases and TP increases, or TN decreases and FP increases.
3. Does ROC AUC tell you how many alerts a team will handle at a particular cutoff? **Answer:** No. Use the cutoff-specific predictions and their counts, then consider operational capacity.

---

Published 2026-10-27.

**Source:** Applied Data Science Using PySpark - Learn the End-to-End -- Ramcharan Kakarla & Sundar Krishnan & Sridhar Alla, section **6.4.2.3 y-hat prediction to probability to final prediction**.

This is an original explanatory note; the source book is not redistributed.
