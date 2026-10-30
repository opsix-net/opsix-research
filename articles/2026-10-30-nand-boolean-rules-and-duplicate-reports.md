# NAND: One Small Rule, Many Boolean Circuits

A circuit designer wants to build a logic function, but the available hardware offers only one kind of gate. Or a researcher has a long list of procedural rules and needs to know which combinations can actually hold. In both cases, the useful first move is to make the logic explicit. A single connective, NAND, is a good place to start because it can express every Boolean function, and because reasoning about it illuminates a broader engineering habit: translate first, simplify second, and verify the result.

## The connective that means “not both”

Let $p$ and $q$ be propositions, each either true or false. The NAND connective, written $p \uparrow q$, is true except when both inputs are true. Equivalently,

$$
p \uparrow q = (pq)'
$$

Here, juxtaposition means conjunction, or “and,” and the prime means negation. Its truth table is:

| $p$ | $q$ | $pq$ | $p \uparrow q$ |
|---|---|---|---|
| 0 | 0 | 0 | 1 |
| 0 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 0 |

In ordinary English, “not both $p$ and $q$” is a direct translation. “Either $p$ is false or $q$ is false” is equivalent. Be careful with “not $p$ and $q,” which can be read as $(p' q)$, rather than as the negation of the whole conjunction. Parentheses are small, but in logic they do a lot of work.

For sets, if $p$ and $q$ name regions, the truth set of $p \uparrow q$ consists of everything outside their intersection. Within the universe under consideration, it is the complement of $p \cap q$. This includes points in neither region and points in exactly one region. Only the overlap is excluded.

## Rebuilding familiar operations from NAND

NAND is called functionally complete because negation, conjunction, and disjunction can each be built from NAND. Once those operations are available, any Boolean expression can be constructed from NAND gates.

Feed the same proposition into both inputs:

$$
p' = p \uparrow p
$$

This works because $p \uparrow p = (pp)' = p'$. Negation is therefore a one-gate operation. To recover conjunction, negate the NAND output:

$$
pq = (p \uparrow q)'
   = (p \uparrow q) \uparrow (p \uparrow q)
$$

Disjunction follows from De Morgan’s law, $(p' q')' = p + q$:

$$
p + q = (p \uparrow p) \uparrow (q \uparrow q)
$$

The right-hand side first constructs $p'$ and $q'$, then NANDs them. This equals $(p'q')'$, which is $p+q$.

Implication can also be expressed in this language. Since $p \to q$ is equivalent to $p' + q$,

$$
p \to q = p \uparrow (q \uparrow q)
$$

The conversion is more than notation practice. It gives a systematic way to implement a Boolean specification with a restricted gate library. It also gives a useful check: derive the expression using identities, then compare its truth table against the intended behavior.

## Exclusive OR is a different promise

NAND says the inputs are not both true. Exclusive OR, often written $p \mathbin{\pm} q$, says exactly one is true:

$$
p \mathbin{\pm} q = pq' + p'q
$$

The distinction is visible when both inputs are false. NAND returns true; exclusive OR returns false. Their English descriptions should not be swapped: NAND means “not both,” while exclusive OR means “one or the other, but not both.”

Exclusive OR can replace ordinary OR in a disjunctive normal form only if the terms being combined cannot both be true. A disjunctive normal form is an OR of conjunctions. If two terms overlap, ordinary OR remains true when both are true, but exclusive OR becomes false. For example, let the terms be $p$ and $p$. Their ordinary disjunction is $p+p=p$, while $p \mathbin{\pm} p=0$. By contrast, distinct minterms in a canonical disjunctive normal form are mutually exclusive: each complete assignment satisfies at most one minterm. In that special setting, XOR and OR give the same result across those terms.

This is a practical reminder: algebraic substitutions need conditions. A connective that looks like a replacement for another may only work when the inputs satisfy a non-overlap property.

## From sentences to a single rule

The same discipline helps with policies, contracts, and workflow conditions. Suppose a lab’s sample-release policy uses three Boolean facts:

- $A$: the identity check is complete.
- $R$: the required report is attached.
- $H$: the sample is on hold.

The policy says: release is permitted only if identity is checked and the report is attached; a sample on hold must not be released. Define $L$ to mean “release is permitted.” A faithful translation is

$$
L = ARH'
$$

This is not just a compact expression. Each factor marks a necessary condition. If any one fails, the conjunction is false and release is not permitted. A NAND-only implementation follows by taking the complement of a three-input conjunction and then inverting that result, or by combining two-input NAND gates. For instance, set $X=A \uparrow R=(AR)'$. Then $AR=X'$, and $L= X'H'$; using only two-input NANDs, first form $X' = X \uparrow X$, then compute $L' = X' \uparrow H'$, and invert once more to obtain $L$.

Check the arithmetic of the logic with a concrete assignment. If $A=1$, $R=1$, and $H=0$, then $X=1 \uparrow 1=0$, so $X'=0 \uparrow 0=1$. Next, $H'=1$. Thus $L'=1 \uparrow 1=0$, and the final inversion gives $L=1$. If the hold flag changes to $H=1$, then $H'=0$, so $L'=1 \uparrow 0=1$, and $L=0$. The policy correctly blocks release. The example is deliberately tiny; production policies need clear definitions, validation, and a record of which rule caused a decision.

## Simplify without changing the meaning

A collection of rules can be combined by conjunction when every rule must hold. Simplifying that conjunction can expose a shorter equivalent policy, but only if the transformation preserves every truth assignment. A truth table is feasible for a few variables; for larger expressions, use established Boolean identities or a tested solver, and retain the original requirements for review.

The source passage’s applied examples emphasize this translation-and-simplification pattern, including conditional statements and conclusions drawn from premises. The general lesson is not that every complicated policy has a pleasing one-line form. It is that informal wording should be converted into explicit propositions before anyone claims that one rule is equivalent to several others. Pay particular attention to “unless,” “only if,” and “either ... or ... but not both.” They encode distinct logical relationships.

## A storage application: report before acting

The same cautious workflow applies outside logic circuits. In an asset pipeline, duplicated images, instrument outputs, or build artifacts can consume storage and complicate reproducibility. A safe first step is a report, not an automatic deletion. The accompanying `duplicate-report` utility groups files with matching SHA-256 digests and estimates potential redundant bytes. It skips hidden paths and symlinks, and it never deletes anything.

This tool is not a Boolean-algebra solver, and the book does not specify it. The connection is methodological: make the conditions explicit, compute a result that can be inspected, and avoid turning a descriptive finding into an unreviewed action. Two files with identical bytes may have different legitimate roles, such as a required copy in a packaged release. A report can guide a human decision, but it cannot infer ownership or intent.

## Assumptions, pitfalls, and practice

Classical Boolean algebra assumes each proposition has one of two values, true or false. Real workflows may have missing, unknown, or conflicting inputs. Treating “not recorded” as false can produce unsafe behavior. Define how such states are represented before translating requirements into Boolean expressions.

Other common mistakes include confusing NAND with XOR, dropping parentheses around a negated conjunction, and assuming OR and XOR are interchangeable. Test boundary cases, especially all-false and all-true inputs. In policy work, also check that the proposition definitions match the organization’s actual terms. A correct formula for the wrong interpretation is still wrong.

### Exercises

1. Express $p'$ using only NAND. **Answer:** $p \uparrow p$.
2. Express $p+q$ using only NAND. **Answer:** $(p \uparrow p) \uparrow (q \uparrow q)$.
3. When can XOR replace OR in a sum of terms? **Answer:** When no assignment makes two or more terms true simultaneously.
4. If $p=0$ and $q=0$, what are $p \uparrow q$ and $p \mathbin{\pm} q$? **Answer:** NAND is 1; XOR is 0.

A small connective can build a complete logic vocabulary. The real skill is not memorizing a gate recipe. It is stating what each symbol means, preserving that meaning through transformations, and checking the result against cases that matter.

---

Published 2026-10-30.

**Source:** Boolean Algebra and Its Applications (Dover Books on -- J_ Eldon Whitesitt [Whitesitt, J_ Eldon] -- INscribe Digital, [N_p_], 2012, section **1 (a) Choose suitable words to express this connective in English.**.

This is an original explanatory note; the source book is not redistributed.
