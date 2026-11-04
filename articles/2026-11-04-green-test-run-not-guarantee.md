# A Green Test Run Is a Starting Point, Not a Guarantee

A test suite passes on your laptop. A week later, a clean build fails because a dependency was never declared, and the code path that handles an empty input has never been exercised. The green check was real, but it answered a smaller question than you thought.

Reliable testing is not one command or one percentage. It is a chain of evidence: tests exercise behavior, coverage points to unvisited code, isolated environments expose dependency assumptions, and automation repeats the checks consistently. Each link catches a different kind of uncertainty.

## Coverage is a map of execution, not correctness

Coverage tools observe which parts of a program run while tests execute. Line coverage asks which executable lines were reached. Branch coverage can ask whether alternatives in a decision were taken. A report can therefore direct attention toward code that tests have not visited.

The selected passage makes two practical points. First, coverage can be used alongside a test runner such as pytest, and reports can be viewed in a terminal or as generated HTML. Second, a high score, even 100 percent, does not prove that every meaningful condition or behavior has been tested. The number records execution, not whether assertions are strong or requirements are satisfied.

Consider this function:

```python
def shipping_cost(weight_kg, express=False):
    if weight_kg <= 0:
        raise ValueError("weight must be positive")
    if express:
        return 12 + 3 * weight_kg
    return 5 + 2 * weight_kg
```

A test with `weight_kg=2` and `express=False` reaches the validation condition, the normal-price calculation, and its return. It does not establish that the express price is right, or that zero and negative weights raise the intended error. Depending on the coverage tool's measurement, that test might still produce a deceptively attractive line-coverage figure. Testing the valid express and invalid-weight cases gives evidence about distinct behavior.

Coverage is best treated as a search aid. Find unvisited code, ask why it is unvisited, then decide whether it needs a test, is unreachable by design, or is obsolete. Do not write tests merely to make colored lines turn green. A test that executes a line without checking its effect provides little protection against a wrong result.

## A worked example: turn an uncovered branch into a behavior check

Suppose a sensor importer accepts a reading in Celsius and an optional threshold. Its policy is simple: reject non-finite readings, otherwise flag values at or above the threshold. The conversion from Celsius to Fahrenheit is $F = 32 + 1.8C$.

For an input of $20$ degrees Celsius, the calculation is:

$$
F = 32 + 1.8(20) = 32 + 36 = 68
$$

If the threshold is $68$, the policy says the reading is flagged because it is at or above the threshold, not only when it is greater. A useful test checks both the numeric conversion and the boundary decision. A separate test should check a value below the threshold. Another should check that `NaN` or infinity is rejected if that is the stated input policy.

This small set matters more than a broad but unassertive test that merely calls the importer. It covers an ordinary value, a decision boundary, and a specified invalid-input case. Coverage can reveal that the non-finite handling line has not run, but only the test's assertion establishes what the program should do.

The example also illustrates a general rule: derive expected values independently. Here, the arithmetic is explicit, and equality at the threshold follows from the phrase “at or above.” If expected values are copied from the implementation, the same mistake can live in both code and test.

## Isolate dependencies so tests mean what they say

A passing test can depend on packages installed globally on one machine. Another developer or build agent may not have those packages, or may have incompatible versions. The passage describes virtual environments as per-application locations for Python dependencies, separate from system-installed modules. That isolation helps expose missing dependency declarations and reduces accidental interference between projects.

For supported Python versions, `venv` can create an environment, and packages can be installed into it with `pip`. In a project, the dependency list should be explicit and the test command should run in the environment the project intends to support. Activation is optional if a command directly names the environment's Python executable.

A clean environment is not a magic guarantee. It only reflects the dependencies and interpreter selected for it. If a package is absent from the project declaration but happens to be present through system-site packages, the environment may conceal the omission. Similarly, one environment cannot demonstrate compatibility with every Python version or operating system.

## Make repeatability routine with environment automation

The passage presents tox as a way to standardize test runs in clean environments. A project configuration can specify dependencies and commands, allowing tox to build an environment, install the project, and run the tests. Multiple environments can check more than one interpreter or task, such as tests and style checks. Recreating an environment is useful when validating that a fresh checkout can be set up from the declared configuration.

This is valuable because “works on my machine” often means “works with my machine's accumulated state.” An automated environment makes that state more visible. It also turns setup instructions into executable configuration that can be reviewed and rerun.

A test matrix is only as useful as its declared scope. If a project supports three Python versions but automation runs only one, the other two remain assumptions. If the test command is missing from the environment configuration, creating an environment alone does not test behavior. Make the supported versions and commands explicit, and inspect the logs when an environment is skipped or fails to build.

## Put checks on the path to changes

The selected passage advocates making tests part of the normal change process, including automated runs in continuous integration. A local run gives rapid feedback; a CI run applies a repeatable check to a commit or proposed change. Review still matters: tests can be incomplete, and automation can faithfully run an inadequate test suite.

Coverage thresholds deserve care. A minimum can prevent a project's measured coverage from falling unnoticed, but an arbitrary threshold can reward superficial tests. Coverage percentage is a summary of the measured code and configuration. It does not measure the quality of assertions, the importance of covered paths, or whether the test suite reflects user expectations. Use the report to ask better questions, not to end the conversation.

A practical workflow looks like this:

1. Run focused tests while developing a behavior.
2. Run the full suite in a clean, declared environment.
3. Review coverage for meaningful unvisited paths and branches.
4. Add tests that verify requirements, including boundaries and error cases.
5. Have automation repeat the chosen checks for each change.
6. Review failures and coverage changes rather than treating a passing badge as proof.

## The same discipline helps with research assets

Testing software and managing scientific files are different tasks, but both benefit from repeatable inspection and cautious interpretation. A duplicate-file report can identify identical assets that may be consuming redundant storage. It does not decide whether one copy is safe to remove. A researcher may need to preserve directory structure, provenance, or independently managed copies even when bytes match.

That distinction mirrors coverage: an observation can point to a question without answering it. Matching file sizes and hashes are strong evidence that the files have identical bytes, not proof or a sign that they are operationally interchangeable. Human review, documented policy, and a safe follow-up action remain necessary.

## Assumptions, limitations, and useful skepticism

Coverage describes execution under the tests and configuration that were actually run. It cannot prove correctness for untested inputs. A clean environment reduces hidden dependency assumptions, but it cannot represent every deployment target. CI repeats configured checks, not checks nobody configured. These are complementary controls, not substitutes for one another.

There are also practical limits. Tests can be slow, flaky, or poorly isolated. Coverage instrumentation can affect runtime, and a report's meaning depends on what source files and test runs it includes. Dependency resolution may change over time unless versions are controlled. Maintain the process: keep test commands understandable, make supported environments explicit, and investigate sudden changes in results.

## Exercises

1. A module reports 100 percent line coverage. Name two questions that the number does not answer.
2. A local test passes, but a clean environment fails to import a package. What assumption did the local run rely on?
3. A duplicate report finds two identical files. Does that prove one can be deleted?

**Short answers:**

1. Whether assertions check correct behavior, and whether all relevant conditions or inputs were tested.
2. The package may have been installed outside the project's declared dependencies or environment.
3. No. Byte identity does not establish that deletion is safe for the workflow or provenance.

---

Published 2026-11-04.

**Source:** Serious Python - black-belt advice on deployment, section **Selected body passage**.

This is an original explanatory note; the source book is not redistributed.
