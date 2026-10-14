# From Responsibilities to Reliable Tools: Designing Small Software Components

A researcher opens a 40-gigabyte application log after an experiment, hoping to answer a simple question: how many lines report errors? The file also contains user identifiers, request details, and other information that should not end up in a notebook or report. A quick script could count matching words, but before writing it, there is a more useful question: which parts of the job belong together, and what information should each part expose?

That question connects design practice to the everyday reliability of research and engineering tools. A small program still needs a structure that makes its responsibilities visible, its behavior checkable, and its output useful without revealing more than necessary.

## Start with actions, then assign components

A practical design technique is to write down a program's responsibilities and the components that might carry them. CRC, short for Class-Responsibility-Collaborator, cards offer one lightweight way to do this. Each card represents a component. Its responsibilities describe the work it performs, and its collaborators are the other components it needs to do that work.

The useful motion is an action/actor cycle. First identify an action, such as “count lines containing a selected pattern.” Then decide which component is best suited to own it. Repeat until every required action has an owner. This is not a claim that every action must become a separate class. Rather, it is a way to see whether a proposed component has a coherent job and whether its dependencies are becoming unwieldy.

Consider a small log-summary command-line tool. Its responsibilities might be:

- Parse the input path and optional search pattern.
- Read the file one line at a time.
- Recognize severity labels and count matching lines.
- Format a machine-readable summary.
- Report invalid input or file access errors.

A sensible design might assign argument parsing to a command-line boundary, line analysis to a counting routine, and JSON formatting to an output step. These can be functions rather than classes. The important point is that the work has clear owners and the boundaries are understandable.

The selected passage describes index cards as a way to name components, record responsibilities and collaborators, and iterate from actions to actors. It also makes an explicit assumption: each component should have a small set of responsibilities and collaborators. That constraint is a practical aid to manageability, not a mathematical guarantee that a design is good. A component with one responsibility can still be poorly named, and a tidy diagram can still encode incorrect behavior.

## Keep the design visible before implementation

Cards are not the only way to reason about structure. UML, or Unified Modeling Language, diagrams provide a standard visual notation for expressing object-oriented designs. Pseudocode gives a compact description of an algorithm without committing to the syntax of a programming language. The passage presents diagrams and pseudocode as alternatives that can help express organization and algorithmic steps.

For the log tool, a short pseudocode sketch could be:

```text
initialize line count, severity counts, matching-line count
for each input line:
    increment line count
    count the first recognized severity label, if present
    increment matching-line count if the search pattern matches
emit the counts as JSON
```

This sketch makes an important policy visible: severity counting and pattern matching are separate tests. A line can contribute to both. It also makes the intended memory behavior visible: process one line at a time instead of storing the entire file. The implementation still needs to settle details such as the exact severity vocabulary, whether matching ignores case, and how malformed text is handled. Those details belong in a precise specification, not in a diagram's vague arrows.

Design artifacts are useful when they help people ask better questions. Does a component need a collaborator for a job it could perform itself? Does the output expose message text when aggregate counts would answer the question? Is the algorithm's behavior clear enough to test? A diagram is not a substitute for these decisions. It is a place to make them easier to inspect.

## From readable code to an executable check

Once responsibilities are clear, implementation can begin in a text editor or an integrated development environment. The passage describes the familiar cycle: write code, compile it, fix syntax errors reported by the compiler, and run the resulting program. The specifics differ by language. C++ source is compiled to executable code, while a Python program is generally interpreted at runtime. Both workflows benefit from treating implementation as a cycle of edits and checks rather than as a single leap from idea to finished software.

Readable style supports that cycle. Meaningful identifiers let a reader infer a variable's role without decoding abbreviations. Named constants and enumerations give stable meaning to values that would otherwise appear as unexplained literals. Consistent indentation shows the boundaries of control flow. A predictable class layout helps readers find public operations and private implementation details. Comments earn their space when they clarify purpose or a confusing decision, rather than restating the code line by line.

These conventions are not universal laws. Naming styles vary across languages and teams, and a consistent local convention is often more useful than importing a convention mechanically. The underlying goal is communication: the code should help another person understand what it does and where its responsibilities live.

## Worked example: count without copying messages

Suppose a laboratory pipeline writes five lines to `application.log`:

```text
2026-10-14T09:00:00Z INFO run=R7 started
2026-10-14T09:00:03Z WARN sensor=S2 delayed
2026-10-14T09:00:05Z ERROR sample=S4 rejected
2026-10-14T09:00:08Z INFO run=R7 checkpoint
2026-10-14T09:00:11Z CRITICAL storage unavailable
```

The log-summary utility recognizes `DEBUG`, `INFO`, `WARN`, `WARNING`, `ERROR`, `CRITICAL`, and `FATAL` as severity labels. It normalizes labels to uppercase and counts `WARNING` under `WARN`. Its default pattern is `ERROR|CRITICAL|FATAL`, matched case-insensitively. It counts a line as matching if that regular expression finds a match anywhere on the line.

The severity arithmetic is direct: there are two `INFO` lines, one `WARN`, one `ERROR`, and one `CRITICAL`. The total is $2 + 1 + 1 + 1 = 5$, equal to the number of input lines because each example line contains one recognized severity label. The default pattern matches the `ERROR` line and the `CRITICAL` line, so the matching-line count is $2$. A `CRITICAL` line contributes to both its severity count and the matching-line count. These are distinct summaries, not mutually exclusive categories.

Run the tool with:

```bash
bash log-summary.sh --file application.log
```

For this input, its JSON fields report `lines` as `5`, `severity_counts` as counts for the recognized labels, and `matching_lines` as `2`. The output contains aggregate counts, not the original log messages. That makes it useful for a quick overview, but it does not prove that every event was captured or that a failure was caused by a particular condition.

## Where this design approach helps

The same responsibility-first approach applies beyond logs. In a scientific workflow, one component can validate a file's structure, another can calculate a summary, and another can serialize results. In a monitoring system, collection, classification, and alert delivery may have different owners because they have different failure modes. In data-processing automation, separating parsing from transformation makes it easier to test each step against known inputs.

This approach is especially helpful when a tool crosses a boundary: from private data to a report, from an instrument to a shared store, or from a script to a scheduled job. Making responsibilities explicit reveals where validation, access control, redaction, and error handling belong. It also helps a team decide what a utility deliberately does not do. A counter that emits totals is not automatically a log viewer, alerting system, or privacy guarantee.

## Assumptions, limits, and common traps

Small components help keep a program manageable, but splitting every line of code into a separate unit can add needless indirection. Combine tightly related work when separate ownership would make the design harder to follow. Conversely, avoid a single “do everything” component whose responsibilities and collaborators keep growing.

For log summaries, a severity count depends on the format. The utility recognizes the first severity label found on each line, not every label. A line with multiple severity words contributes only the first recognized one to `severity_counts`. Pattern matching is independent and counts each matching line once, even if the expression could match more than once within that line. The tool does not retain or print the messages, but reading a sensitive file still requires appropriate local access controls.

There is another easy mistake: treating a count as an explanation. Counts describe what the chosen rules found in the chosen input. They do not establish statistical significance, causation, or completeness. A line that uses an unrecognized label will not appear in the severity totals. If the input format changes, the summary rules may need to change too.

## Exercises with short answers

1. A component parses command-line options and also formats JSON. What design question should you ask? **Answer:** Whether these responsibilities have a clear reason to change together. If not, give them distinct owners or functions.
2. A five-line file has one recognized severity on every line, with counts of two `INFO`, one `WARN`, and two `ERROR`. How many lines match the default pattern? **Answer:** Two, since both `ERROR` lines match.
3. Why is a pseudocode sketch useful before implementation? **Answer:** It exposes the intended sequence and policies, making ambiguities easier to find before language details obscure them.

A useful design is not the one with the most cards, classes, or diagrams. It is the one whose responsibilities can be explained, whose boundaries can be tested, and whose output answers the intended question without pretending to answer a larger one.

---

Published 2026-10-14.

**Source:** Data Structures and Algorithms in C++ 2e, section **56 Chapter 1. A C++ Primer**.

This is an original explanatory note; the source book is not redistributed.
