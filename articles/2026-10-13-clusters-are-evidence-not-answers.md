# Clusters Are Evidence, Not Answers

Imagine a research team divides customer records, cells, or sensor traces into groups and discovers that the groups look wonderfully distinct on a chart. A week later, a colleague changes the feature scaling and gets a different picture. Which result is true? The uncomfortable answer is that neither chart, by itself, establishes a natural set of groups. Clustering is a way to organize observations under chosen rules. Its value depends on how carefully we test and interpret those rules.

The discussion in *An Introduction to Statistical Learning with Applications in R* makes this central point: clustering can be useful, but small choices, including scaling and linkage, can substantially change the result. It recommends examining results across choices and subsets, then treating the findings as a starting point for scientific hypotheses and further study, ideally with independent data. That is a practical standard for any team using unsupervised learning.

## A cluster is conditional on a recipe

A clustering algorithm does not usually receive a known answer key. It receives a representation of observations and a procedure for grouping them. The resulting clusters therefore depend on both the data and the recipe.

For example, Euclidean distance between two observations with features $x_1,\ldots,x_p$ is

$$
d(x_i,x_j)=\sqrt{\sum_{k=1}^{p}(x_{ik}-x_{jk})^2}.
$$

A feature measured in thousands can dominate a feature measured in fractions, because its differences contribute much larger squared terms. Standardizing a feature by subtracting its mean and dividing by its standard deviation changes the geometry: each feature is expressed in standard-deviation units. This can be sensible when units differ, but it is not a neutral cleanup step. It gives low-variance features more influence relative to their raw scale, which may or may not fit the scientific question.

The source illustrates this issue with principal components analysis (PCA) on state-level crime and urbanization measurements. The variables have different means, variances, and units. Without scaling, variation in assault measurements can dominate PCA; standardizing puts the variables on comparable standard-deviation scales. PCA is not clustering, but it demonstrates the wider lesson: preprocessing determines what differences a method can see. A plot of the first two principal components is also only a projection of the data, not a complete view of all dimensions.

Clustering has its own consequential choices. In hierarchical clustering, linkage defines how the distance between two candidate groups is measured. Single linkage uses the closest pair of points; complete linkage uses the farthest pair; average linkage uses an average of pairwise distances. These rules can build different trees from the same observations. Cutting a tree at a chosen height or into a chosen number of groups adds another decision.

K-means has different sensitivities. It partitions observations into a specified number $K$ of groups, seeking to minimize the total within-cluster sum of squares:

$$
\sum_{r=1}^{K}\sum_{i\in C_r}\lVert x_i-\bar{x}_r\rVert^2,
$$

where $C_r$ is cluster $r$ and $\bar{x}_r$ is its mean. The objective can have local optima, so different initial assignments can yield different answers. The source recommends multiple starts, such as 20 or 50, and setting a random seed for reproducibility. Multiple starts reduce dependence on a poor initialization; they do not prove that the selected number of clusters or the cluster interpretation is correct.

## A small worked example: a scale choice changes the story

Consider four observations with two measurements, $A$ and $B$:

| Observation | $A$ | $B$ |
|---|---:|---:|
| P | 0 | 0 |
| Q | 2 | 0 |
| R | 0 | 1 |
| S | 2 | 1 |

Suppose we use two-means clustering and compare two natural partitions. The first groups by $A$: $\{P,R\}$ and $\{Q,S\}$. The second groups by $B$: $\{P,Q\}$ and $\{R,S\}$. For either partition, each cluster contains two points, and its mean is the midpoint of those points.

For the partition by $A$, each point is $0.5$ units from its cluster mean in the $B$ direction. Each squared distance is $0.25$, giving a total of $4\times0.25=1$. For the partition by $B$, each point is one unit from its cluster mean in the $A$ direction, so each squared distance is $1$. Across four observations, the within-cluster sum of squares is $4$. With the raw values, the partition by $A$ has the lower objective.

Now multiply every $B$ value by 3. The vertical separation becomes 3. In the partition by $A$, the vertical deviations from each cluster mean are $1.5$, so their squared contributions total $4\times2.25=9$. In the partition by $B$, the vertical deviations are zero because each cluster has a single $B$ value. The horizontal deviations are one per point, giving a total of $4$. The preferred partition switches to grouping by $B$.

The arithmetic does not show that one result is universally correct. It shows that the distance geometry expresses a choice about relative feature importance. In actual K-means runs, initialization and the optimization path can add variation too. A careful analyst would inspect plausible scaling choices and repeated starts, then ask which grouping is meaningful for the domain rather than declaring the numerically best partition to be discovered truth.

## Robustness checks make uncertainty visible

A useful stability check varies one decision at a time. For example, compare standardized and unstandardized inputs, or compare single, complete, and average linkage. If a group appears under several reasonable specifications, that recurrence is evidence worth investigating. If it appears only under one narrow configuration, report that dependence rather than hiding it.

Subsampling is another check emphasized by the source. Fit the method on different subsets of observations, then examine whether similar groups or memberships recur among observations shared across runs. This probes whether the result depends heavily on particular records. It does not guarantee stability in every future sample, especially when the sample is small or the data distribution changes.

Cluster labels themselves are arbitrary. A software run may call a group “1” and another run may call the same group “2.” Compare membership or group structure, not numeric label equality. For a rigorous comparison, align labels or use a label-invariant measure of agreement. Also distinguish a stable partition from a useful one: a consistently recovered grouping can still reflect a confounder, a measurement artifact, or an unhelpful definition of similarity.

The source’s examples reinforce these cautions. Its K-means illustration shows that choosing three groups can split data that were generated with two groups. Its hierarchical example shows that complete and average linkage can yield different cuts from single linkage. These are demonstrations of method behavior on particular examples, not universal promises about which method will succeed on new data.

## From exploratory pattern to scientific claim

A responsible report describes the choices that produced the clusters: included observations and features, missing-data handling, transformations and scaling, distance, linkage or algorithm, chosen cluster count, and any random seed or initialization strategy. Show sensitivity results, not only the most attractive plot. State what is descriptive and what remains uncertain.

Avoid language that turns an algorithmic partition into an established natural category. “Under standardized Euclidean distance and complete linkage, these observations form a recurring group” is more informative than “the data contain this true type.” Cluster membership is not a diagnosis, causal explanation, or prediction of future behavior unless separate evidence supports those uses.

An independent data set is especially valuable. First formulate a hypothesis from the exploratory grouping, such as a proposed relationship between a cluster and a measured biological response. Then test that hypothesis using suitable independent observations and a prespecified analysis. Independent validation is not merely rerunning the same clustering until a preferred picture returns. It asks whether a meaningful claim survives new evidence.

## Applications and engineering habits

These ideas matter in genomics, where samples may group by biological subtype, batch, or both; in manufacturing, where sensor profiles may flag operating regimes; and in customer research, where segments can depend on how spending, age, and activity are scaled. In every case, a cluster can guide follow-up measurement or hypothesis generation. It should not silently replace subject-matter reasoning.

The same reproducibility discipline applies to the files behind an analysis. A research pipeline may produce a model, a figure, and a data extract. Before sharing a result or building a downstream step, a developer can inventory artifact paths, byte counts, and SHA-256 digests. Such an inventory does not validate the science or prove that an artifact is correct. It helps make the exact files visible for comparison and review. The accompanying `file-manifest.sh` utility is designed for this limited, useful role: it reads files and prints a deterministic inventory without modifying them.

## Assumptions and common pitfalls

Distance-based methods assume that the chosen features and distance express relevant similarity. Standardization assumes that equal standard-deviation units are a defensible comparison. K-means represents groups by means and is sensitive to outliers; a few extreme observations can pull a center. Hierarchical methods depend on both the distance and linkage definition. None of these methods automatically identifies the “right” number of groups.

Other frequent mistakes include treating a two-dimensional PCA plot as the whole data set, interpreting label numbers as meaningful, reporting only one random run, and selecting preprocessing after seeing which result supports a preferred narrative. Keep an analysis log, retain configuration and seeds, and separate exploratory decisions from confirmatory tests. If results change across reasonable choices, that instability is a result to report, not a nuisance to conceal.

## Exercises with short answers

1. If multiplying one feature by 100 changes a Euclidean-distance clustering, what should you conclude? **Answer:** The raw-scale analysis gave that feature much greater geometric influence. Decide whether that weighting is scientifically appropriate; do not treat either outcome as automatically true.

2. Why use multiple K-means starts and a fixed seed? **Answer:** Multiple starts explore alternative initial assignments and may avoid a poor local optimum. A fixed seed makes the chosen run reproducible, but neither step establishes scientific validity.

3. A cluster appears under three linkage choices but disappears in most subsamples. Is it robust? **Answer:** Not on that evidence. It is insensitive to those linkage choices but sensitive to the sampled observations, so describe both findings and investigate further.

4. What is a good next step after finding a plausible cluster pattern? **Answer:** Formulate a specific hypothesis and assess it with independent data or another appropriate validation design.

Clustering is most useful when treated as disciplined exploration. Try reasonable alternatives, inspect stability, document the recipe, and let the results suggest questions rather than dictate answers.

---

Published 2026-10-13.

**Source:** An Introduction to Statistical Learning  with Applications in R, section **401 A Tempered Approach to Interpreting the Results of Clustering**.

This is an original explanatory note; the source book is not redistributed.
