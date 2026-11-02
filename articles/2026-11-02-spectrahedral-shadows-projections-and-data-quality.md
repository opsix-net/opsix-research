# When a Projection Changes the Shape of a Feasible Set

A model can be perfectly explicit in its working variables and still look surprising when you inspect only the quantities you care about. In semidefinite optimization, that surprise has a name: a spectrahedral shadow. A set described by a positive semidefinite matrix inequality is projected onto fewer coordinates, hiding auxiliary variables. The result remains convex and semialgebraic, but it may no longer be describable by a single linear matrix inequality in the visible coordinates.

That distinction matters in optimization and in scientific modeling. A lifted formulation can make a difficult constraint manageable, while projection can alter properties such as closedness. The selected chapter of *Geometry of Linear Matrix Inequalities* introduces these projected feasible sets, describes operations that preserve them, and connects them to polynomial optimization. This article develops those ideas through a small calculation, then considers what they mean for practical workflows.

## From a matrix inequality to a shadow

A spectrahedron is a set described by a linear matrix inequality (LMI). In one common form,

$$
S=\{z\in\mathbb{R}^k:M(z)\succeq 0\},
$$

where $M(z)$ is a symmetric matrix whose entries are affine-linear functions of $z$, and $\succeq 0$ means positive semidefinite. Equivalently, every eigenvalue of $M(z)$ is nonnegative. The set $S$ is convex: if two matrices are positive semidefinite, every convex combination of them is positive semidefinite too.

A spectrahedral shadow is the image of a spectrahedron under an affine-linear map. In the especially useful projection form, write the variables as visible coordinates $x$ and auxiliary coordinates $y$. Then

$$
P=\{x:\text{there exists }y\text{ such that }M(x,y)\succeq 0\}.
$$

The auxiliary variables are also called lifted variables. They can make a compact matrix description possible even when no equally simple LMI in $x$ alone is available. The chapter explains that any affine-linear image can be represented this way: one can encode the map's graph with linear equalities, then project away the original coordinates.

Convexity survives projection. If $x_1$ and $x_2$ have feasible lifts $y_1$ and $y_2$, then a convex combination of those lifts certifies the same combination of $x_1$ and $x_2$. The chapter also notes that spectrahedral shadows are semialgebraic, a consequence of the projection theorem. But they need not be spectrahedra, and, more unexpectedly, they need not be closed.

## A complete example: projecting to an open ray

Consider the two-variable matrix

$$
M(x,y)=\begin{pmatrix}x&1\\1&y\end{pmatrix}
$$
and the spectrahedron $S=\{(x,y):M(x,y)\succeq0\}$. We will project $S$ onto the $x$-axis.

For a real symmetric $2\times2$ matrix to be positive semidefinite, its diagonal entries must be nonnegative and its determinant must be nonnegative. Here those conditions are

$$
x\geq0,\qquad y\geq0,\qquad xy-1\geq0.
$$

Since $xy\geq1$, neither $x$ nor $y$ can be zero, so every feasible point has $x>0$. Conversely, for any chosen $x>0$, set $y=1/x$. Then $y>0$ and $xy-1=0$, so the matrix is positive semidefinite. Therefore

$$
\{x:\exists y,\ M(x,y)\succeq0\}=(0,\infty).
$$

Check a concrete point: if $x=2$, choosing $y=1/2$ gives determinant $2(1/2)-1=0$, and both diagonal entries are positive. The matrix is positive semidefinite, so $2$ is in the projection. If $x=0$, the determinant is $-1$ for every $y$, so zero is not in it. The projected set is an open ray, despite the original spectrahedron being closed. Thus projection of a closed set, even a spectrahedron, need not be closed.

This example is also a warning about modeling: a solver's feasible set in lifted coordinates and the set of visible outputs are related, but they are not interchangeable descriptions. Whether a boundary point is attained can depend on the existence of a finite lift.

## Lifts as a modeling language

A representation of the form $P=\{x:\exists y,\,M(x,y)\succeq0\}$ is often called a lifted LMI representation, an extended formulation, or a semidefinite representation. These names emphasize different uses of the same construction. The lift adds variables; the LMI gives a structured feasibility test; projection returns the quantities of interest.

The chapter's examples include the cone of sums of squares of polynomials. A polynomial that is a sum of squares can be represented using a positive semidefinite Gram matrix. The polynomial's coefficients are linear functions of the Gram matrix entries, so mapping feasible Gram matrices to coefficients gives a spectrahedral shadow. This is useful because the coefficient vector alone need not have a direct spectrahedral description. The lift provides a finite-dimensional certificate of the sum-of-squares representation.

A certificate is not automatically a complete description of every polynomial nonnegative on a set. Degree bounds and the choice of polynomial generators matter. The chapter introduces Lasserre-Parrilo relaxations as a systematic way to construct spectrahedral shadows from truncated polynomial data. These relaxations can approximate semialgebraic sets and, in appropriate cases, help establish that a convex hull is a spectrahedral shadow. Exactness is a mathematical property to prove, not a consequence of merely writing down a relaxation.

## Useful operations, with a qualification

The selected chapter establishes that many convex-geometric operations preserve spectrahedral shadows. These include intersections, Cartesian products, affine-linear images and preimages, Minkowski sums, conic hulls, convex hulls, projective transformations, polars, dual cones, closures, and relative interiors, under the relevant definitions and hypotheses.

The practical idea is compositional modeling. Suppose two sets each have a lifted LMI description. Their intersection can be described by retaining both sets of auxiliary variables and imposing both matrix inequalities. A Cartesian product can be built from the two descriptions as well. For a Minkowski sum, introduce one point from each set and impose that their sum equals the visible point. Each construction adds bookkeeping, but it preserves the broad representation class.

Do not overgeneralize this result. It says these operations preserve the class of spectrahedral shadows, not that they preserve the narrower class of spectrahedra. Projection itself is a central reason for the distinction. Nor does representability guarantee that a formulation is small, numerically well-conditioned, or easy to solve. Those are separate engineering questions.

The chapter also gives a useful geometric scope check: every convex semialgebraic set in the plane is a spectrahedral shadow, while in higher dimensions there are convex semialgebraic sets that are not. Therefore, convexity plus polynomial inequalities does not by itself promise a semidefinite lift in every dimension.

## Applying the idea responsibly

In control, estimation, and operations research, an auxiliary variable can encode a certificate or a decomposition that makes a constraint convex. In polynomial optimization, a Gram matrix can certify a sum-of-squares identity. In scientific computing, lifted variables may represent latent quantities or intermediate physical states. In each case, projection expresses the set of observable or decision variables compatible with at least one valid internal explanation.

A careful workflow starts by writing the variables and the map explicitly. Identify which coordinates are reported and which are existentially quantified. Check whether the desired property concerns the lifted feasible set or its projection. If boundary behavior matters, test it directly: the open-ray example shows why assuming closedness is unsafe. If a hierarchy of polynomial relaxations is used, record its degree and generator assumptions, and distinguish a relaxation from an exact representation.

For computational data, the same discipline applies in a more modest way: inspect the input before trusting the output. A small profile of missing values, row widths, and numeric ranges does not prove a scientific model, but it can expose data-shape problems before those problems enter an optimization pipeline. The accompanying `csv-profile` utility performs that kind of descriptive inspection. It is inspired by the general engineering lesson of making intermediate structure visible; it is not a tool specified by the mathematics text.

## Assumptions and common pitfalls

The LMI discussion uses real symmetric matrices and finite-dimensional real vector spaces. A matrix inequality is not a collection of unrelated scalar tests: positive semidefiniteness is a joint condition on the whole matrix. In the worked example, the diagonal and determinant tests are sufficient because the matrix is only $2\times2$; do not transfer that shortcut blindly to larger matrices.

A projection means existential feasibility over the hidden coordinates. It does not mean fixing those coordinates, averaging them, or selecting a unique lift. Multiple lifts may correspond to one visible point. Also keep the distinctions among closedness, convexity, semialgebraicity, and spectrahedral representability: one does not imply all the others. Finally, a semidefinite representation can be mathematically valid while being computationally inconvenient. Numerical conditioning and formulation size deserve their own review.

## Exercises with short answers

1. For the matrix in the worked example, can $x=0$ be feasible for some $y$? **Answer:** No. Its determinant is always $-1$, which violates positive semidefiniteness.
2. If two points have feasible lifts, why does their midpoint have a feasible lift? **Answer:** Take the midpoint of the two lifts. The corresponding matrix is the midpoint of two positive semidefinite matrices and is therefore positive semidefinite.
3. Does the closure of the projected set in the example equal the projection? **Answer:** No. The closure is $[0,\infty)$, while the projection is $(0,\infty)$.
4. Does a sums-of-squares lift establish that every nonnegative polynomial has such a representation at the same degree? **Answer:** No. Degree restrictions and the underlying set matter; exactness requires justification.

The central lesson is simple but powerful: adding variables can simplify a representation, and hiding those variables can change the geometry visible to the modeler. Treat the lift and its projection as related objects, not as synonyms. That habit makes semidefinite models easier to reason about, validate, and explain.

---

Published 2026-11-02.

**Source:** Geometry of Linear Matrix Inequalities_ A Course in -- Tim Netzer, Daniel Plaumann -- Compact Textbooks in Mathematics, section **Selected body passage**.

This is an original explanatory note; the source book is not redistributed.
