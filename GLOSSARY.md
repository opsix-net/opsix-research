# Glossary

New concepts are appended as research notes are published.


<!-- publication:2026-10-02 -->

## 2026-10-02

Source note: [2026-10-02](articles/2026-10-02-trading-applications-data-protocols-safe-boundaries.md).

### FIX

Financial Information eXchange, a messaging protocol used to communicate financial information and instructions. It provides conventions for encoding messages, but implementations can differ by protocol version and venue-specific requirements, so support for FIX alone does not guarantee drop-in compatibility.

### Tag

A numeric field identifier in a FIX message, paired with a value in `tag=value` form. Tags identify items such as message type or instrument. Applications should use a clear internal mapping and validate required fields against the relevant protocol and venue rules.

### SOH

Start of Header, the ASCII control byte `0x01` used to separate FIX fields. A printable character such as a vertical bar may represent SOH in documentation, but must not replace the actual byte in a transmitted message.

### BodyLength

The value in FIX tag 9, giving the byte length of the message body: the bytes after the delimiter ending tag 9 and before the start of tag 10. It must be calculated from the serialized byte sequence, including field delimiters.

### Checksum

The value in FIX tag 10, calculated from the message bytes preceding that tag according to the protocol’s checksum rule. It is a transmission-format check, not proof that the message is valid for a venue or that an order is safe or accepted.

### Session

An established exchange of messages between identified sender and target systems. A session includes connection and lifecycle behavior such as logon and logout; exact authentication, recovery, and required message fields depend on the counterparty’s specification.

### Venue adapter

A software boundary that translates internal application concepts into a particular broker’s or trading venue’s interface. It isolates differences in fields, supported order types, and connection conventions from strategy and domain logic.

### Syntactic validity

Conformance to a message’s structural rules, such as field encoding and required formatting. A syntactically valid request may still be semantically inappropriate, unauthorized, unsupported, or outside risk limits.

### Ragged row

A CSV data record whose number of fields differs from the number of header columns. In this utility, such rows are reported by file line number and excluded from per-column statistics because their values cannot be safely aligned to the header.

### Finite numeric value

A value successfully parsed as a number that is neither positive infinity, negative infinity, nor NaN. The CSV profiler includes finite numeric values in counts and summaries, while excluding nonfinite values from those calculations.


<!-- publication:2026-10-05 -->

## 2026-10-05

Source note: [2026-10-05](articles/2026-10-05-flexible-json-reliable-data-workflows.md).

### JSON

JavaScript Object Notation, a text format for representing values such as objects, arrays, strings, numbers, booleans, and null. Its flexibility makes it useful for exchanging data, but records in one file may have different keys or nested shapes, so successful parsing does not guarantee consistent meaning.

### Semi-structured data

Data that has some organization, such as named fields or nested objects, without requiring every record to follow one fixed table schema. JSON and HTML-derived records are common examples. Consumers usually need to inspect and normalize the structure before treating values as consistent analytical variables.

### Data validation

Checks that data and configuration meet declared expectations, such as required fields, types, permitted ranges, or a valid graph shape. Validation can detect specified problems, but it cannot establish that the expectations are scientifically appropriate or that untested data properties are correct.

### Dependency graph

A representation of tasks as nodes and prerequisite relationships as directed edges. An edge from task A to task B means B requires A to finish first. Making these relationships explicit helps teams review execution order and spot missing or contradictory prerequisites.

### Directed acyclic graph (DAG)

A directed graph with no directed path that loops back to its starting node. A DAG admits at least one topological ordering, making it useful for workflows where tasks have prerequisites but no iterative cycles.

### Topological order

A sequence of a graph’s nodes in which each node appears after all of its prerequisites. Such an order exists for a directed acyclic graph and may not be unique. The order satisfies declared constraints but does not by itself optimize runtime or validate the tasks.

### Cycle

A directed path that returns to its starting node. In a dependency graph, a cycle means that tasks in the loop cannot all be scheduled in a one-pass prerequisite order. It may indicate an erroneous dependency or an iterative process that needs a different workflow model.

### Kahn’s algorithm

A topological-sorting method that repeatedly removes nodes with no remaining prerequisites and releases their dependents. If nodes remain when no more nodes are ready, the graph contains a cycle or nodes blocked downstream from one.

### Deterministic ordering

A rule that produces the same sequence from the same graph and implementation choices. Choosing the lexicographically smallest currently ready node makes this utility’s output repeatable, but does not make task results deterministic if inputs or computations vary.

### Workflow orchestration

The coordination of tasks and their dependencies, often including when tasks run and how failures are handled. A topological ordering provides a possible sequence for an acyclic workflow, while production orchestration may additionally require scheduling, retries, monitoring, and provenance.


<!-- publication:2026-10-08 -->

## 2026-10-08

Source note: [2026-10-08](articles/2026-10-08-machine-signals-manufacturing-decisions.md).

### Smart manufacturing

A manufacturing approach that connects production operations with data and complementary technologies to improve visibility, coordination, and decision-making. It can include IoT, analytics, automation, cloud services, and enterprise-system integration, rather than referring to one specific machine or platform.

### Digital thread

An integrated view that links information across manufacturing processes and stages of a product's life cycle, such as design, production, use, service, repair, and decommissioning. Useful links require consistent identity and context across systems; sensor installation alone does not guarantee a functioning thread.

### Overall Equipment Effectiveness (OEE)

A metric expressed as the product of availability, performance, and quality. Its value depends on clearly defined time boundaries, speed baselines, and quality denominators. It describes performance under those definitions but does not, by itself, explain causes or prove an intervention worked.

### Operational Technology (OT)

Technology used to monitor or control physical industrial processes. Examples include programmable logic controllers, SCADA, and CNC systems. OT environments can have different availability, network, and security requirements from business-focused IT systems.

### Information Technology (IT)

Technology commonly used to process, store, and exchange organizational information. In manufacturing integrations, IT may consume or contextualize data from OT, but the two environments should not be treated as interchangeable in their operational and security needs.

### Discrete manufacturing

Production of distinct, countable products or parts, such as furniture, machine components, or phones. Individual units or lots may be tracked through production and later life-cycle stages when identifiers and connected records are available.

### Process manufacturing

Production that transforms ingredients or raw materials using recipes and physical or chemical processes. Because inputs and outputs may not be individually countable like discrete items, useful records may focus on batches, recipes, quantities, and process conditions.

### Manufacturing Execution System (MES)

Software that monitors, tracks, controls, or synchronizes production execution, including tasks and schedules. MES capabilities can overlap with broader smart-manufacturing systems, so integration or replacement work should first clarify existing system responsibilities.

### SCADA

Supervisory Control and Data Acquisition systems monitor and control industrial processes, often providing operators with real-time or near-real-time visualization and control interfaces. SCADA may contribute to a wider smart-manufacturing environment but is not equivalent to every capability in that environment.

### Industry 5.0

A human-centric framing of industrial development that emphasizes collaboration between people and machines, sustainability, responsibility, and attention to societal needs. It broadens the goals beyond optimization and efficiency alone.


<!-- publication:2026-10-09 -->

## 2026-10-09

Source note: [2026-10-09](articles/2026-10-09-raw-records-useful-visual-story.md).

### Data visualization

A purposeful visual representation of data, such as a chart, map, or graph, intended to help people inspect or communicate information. A visualization can reveal patterns, trends, and outliers, but its usefulness depends on the underlying data and the choices made in constructing and interpreting the display.

### DUSSSS

A six-part visualization framework named in the chapter: Data, User, Strategy, Structure, Style, and Story. It prompts a designer or engineering team to consider evidence, audience, purpose, organization, appearance, and intended takeaway as connected decisions rather than treating chart styling as the whole task.

### Nominal data

Categorical data whose labels have no inherent ordering, such as service names or colors. Assigning numeric codes to nominal categories does not make them quantities. Charts and calculations should preserve the distinction between category identity and measured magnitude.

### Ordinal data

Categorical data with a meaningful order, such as low, medium, and high. The ordering is informative, but the gaps between levels are not necessarily equal. Treating ordinal codes as measurements with equal intervals requires additional justification.

### Data strategy

Planning for how data are captured, extracted, cleaned, integrated, and maintained for a visualization or analysis. It includes considering source quality and update behavior, so the displayed information is fit for the question and does not imply more freshness or completeness than the pipeline provides.

### Structure

The organization of the data and the presentation, including what appears, in what order, at what level of detail, and how often it refreshes. A clear structure helps users navigate information and understand the relationship between the displayed evidence and the question being asked.

### Outlier

An observation that differs noticeably from others in a dataset or context. An outlier may signal a real unusual event, a measurement problem, or a data-processing issue. Its appearance on a chart is a reason to investigate, not automatic proof that it should be removed.

### JSON Lines

A text format in which each nonblank line contains a separate JSON value, commonly one record per line. Because records can be read independently, JSON Lines is practical for streamed exports and pipeline handoffs. A particular consumer may impose further requirements, such as requiring every record to be an object.

### Field type consistency

The degree to which a named field appears with the same decoded data type across records. A field that alternates between a number and a string may complicate downstream processing. An observed type count is a diagnostic, not a declaration of which type is correct.

### Missing-field count

A count of valid object records that do not contain a particular field. It differs from a field whose value is explicitly null: the field is present in the latter case. Missingness can affect analysis and should be understood before summaries or visualizations are interpreted.


<!-- publication:2026-10-13 -->

## 2026-10-13

Source note: [2026-10-13](articles/2026-10-13-clusters-are-evidence-not-answers.md).

### Clustering

An unsupervised learning approach that assigns observations to groups according to a chosen representation and similarity rule. The resulting groups depend on decisions such as feature selection, scaling, distance, algorithm, and cluster count, so they are not automatically intrinsic or scientifically verified categories.

### Feature scaling

A transformation that changes the numerical scale of a feature, often by centering it and dividing by its standard deviation. Scaling can prevent large-unit measurements from dominating distance calculations, but it also changes the relative influence of features and should reflect the analysis goal.

### Euclidean distance

The straight-line distance between observations represented as numeric vectors. It combines squared differences across features, making it sensitive to feature units and scale. It is common in methods such as K-means, but its appropriateness depends on the measurement context.

### Linkage

A rule for defining the distance between two groups during hierarchical clustering. Single, complete, and average linkage use different summaries of pairwise distances, so the same observations can produce different trees under different linkage choices.

### K-means

A partitioning algorithm that assigns observations to a specified number of groups and seeks to minimize the sum of squared distances from points to their assigned group means. Its result can depend on initial assignments, the chosen value of K, feature scaling, and outliers.

### Local optimum

A solution that cannot be improved by the algorithm's local updates, even though a better solution may exist elsewhere in the search space. K-means can converge to different local optima from different initial assignments, which motivates running multiple starts.

### Principal components analysis (PCA)

A method that constructs orthogonal directions capturing successive amounts of variation in numeric data. Its directions depend on the variables' scales unless the variables are standardized. A plot using only a few components is a projection and may omit structure in remaining dimensions.

### Robustness check

An analysis that examines whether a pattern persists under reasonable changes, such as alternative scaling, linkage, initialization, or data subsets. Stability increases confidence that a finding is not tied to one narrow setup, but does not by itself prove that the pattern is meaningful.

### Independent validation

Assessment of a hypothesis or proposed pattern using data not used to develop it. Independent evidence helps distinguish a promising exploratory observation from a result that depends on the original sample or analysis choices.

### SHA-256

A cryptographic hash function that maps input bytes to a fixed-length digest. A file inventory can use the digest to compare content across copies or runs. A digest does not identify authorship, certify safety, or establish the correctness of a file's contents.


<!-- publication:2026-10-14 -->

## 2026-10-14

Source note: [2026-10-14](articles/2026-10-14-responsibilities-to-reliable-tools.md).

### CRC cards

A lightweight design aid in which each card represents a component and records its responsibilities and collaborators. The technique encourages designers to assign actions to suitable actors before implementation, while keeping component duties and dependencies small enough to reason about.

### Responsibility

A defined piece of work or obligation assigned to a component. Clear responsibilities help readers locate behavior, identify suitable tests, and notice when one component has accumulated unrelated duties.

### Collaborator

Another component that a component interacts with to carry out its responsibilities. Naming collaborators makes dependencies visible and can reveal unnecessary coupling or unclear ownership.

### Action/actor cycle

A design process that starts by identifying an action, then selects the component best suited to perform it. Repeating this process assigns required work to components and exposes responsibilities that lack a clear owner.

### UML

Unified Modeling Language, a standard visual notation used to express software designs, including object-oriented structures. UML diagrams can help communicate organization, but they do not by themselves specify every runtime behavior or validate an implementation.

### Pseudocode

A language-independent description of an algorithm's steps. It can make ordering, branching, and policies visible before implementation details such as syntax or library choices are introduced.

### Named constant

A meaningful identifier associated with a fixed value, used instead of repeatedly embedding an unexplained literal. Named constants can make code easier to understand and change, provided their names accurately describe the values.

### Severity label

A log token that communicates an event's category or urgency, such as `INFO`, `WARN`, or `ERROR`. A counter recognizes only labels defined by its rules, so the resulting totals depend on the application's logging format.

### Regular expression

A compact pattern language for describing text to search for. Its syntax and matching behavior depend on the implementation and options in use; an expression that matches a line does not necessarily identify a unique event or explain its cause.

### Descriptive summary

A compact account of observed data, such as a count of lines or labels. It describes what the chosen procedure found in the supplied input, but alone does not establish causation, statistical significance, or completeness.


<!-- publication:2026-10-15 -->

## 2026-10-15

Source note: [2026-10-15](articles/2026-10-15-from-spring-physics-to-reliable-simulation.md).

### Ordinary differential equation

An equation relating an unknown function of one independent variable to its derivatives. In the spring model, the stretch depends on time, and its first and second time derivatives describe velocity and acceleration.

### State variable

A quantity that represents the system at a given time and is advanced by a computational model. The spring stretch and its velocity together determine the next state in a time-stepping simulation.

### Damping

A force or effect that removes energy from motion. In the passage's model, damping is proportional to the stretch velocity and opposes the motion under the specified sign convention.

### Finite difference

A numerical approximation to a derivative formed from function values at discrete points. Finite-difference formulas let a program approximate continuous motion by advancing values on a time grid.

### Time step

The interval between successive simulation times, commonly written as $\Delta t$. Smaller steps provide more frequent computed values, but require more work and do not by themselves guarantee accuracy or stability.

### Explicit Euler method

A time-stepping method that advances a state using derivative information evaluated at the current time. It is simple to implement and useful for demonstrations, but can be inaccurate or unstable for unsuitable step sizes or systems.

### JSON object

A JSON value consisting of named fields and associated values, represented with braces. The audited utility counts only successfully parsed objects as valid records; other JSON values are reported as non-object lines.

### Type consistency

A property of a data field whose values follow an expected type convention across records. jsonl-audit reports observed type counts, but does not decide which types are acceptable or enforce a schema.

### Descriptive diagnostic

A summary of observed data properties, such as counts or categories, that does not establish cause, significance, or correctness against an external standard. Audit output should guide further validation rather than substitute for it.


<!-- publication:2026-10-16 -->

## 2026-10-16

Source note: [2026-10-16](articles/2026-10-16-trustworthy-market-data-public-apis.md).

### Public API

A service interface that an external application can call under defined access and usage rules. For market data, its documentation determines request parameters, authentication, response shape, limits, and permitted use. Public availability does not mean unlimited access or unrestricted redistribution.

### HTTP status

A response code that communicates the result of an HTTP request at the protocol level. A success status does not guarantee that a provider returned the requested market observation, because application-level errors or quota messages can appear in the response body.

### Serde

A Rust framework for serializing and deserializing data structures. With suitable derives and attributes, it can map JSON fields to Rust types, including keys that do not match Rust naming conventions. Successful deserialization checks shape and type compatibility, not market-data meaning.

### Asynchronous I/O

A programming approach that allows a task waiting on network input or output to yield execution so other work can proceed. Rust applications commonly use an async runtime such as Tokio with an HTTP client. It improves coordination of waiting work, but does not make a remote service faster or guarantee a response.

### Rate limit

A provider-imposed constraint on how frequently or how many requests a client may make. Limits may be per time window, account, endpoint, or plan, and may include burst or daily caps. The current provider policy and returned throttling signals should guide scheduling.

### Backoff

A retry strategy that waits before making another attempt after a recoverable failure, often increasing the delay across repeated failures. Backoff can reduce repeated pressure during outages or throttling. It must be bounded and cannot override a provider’s request limits.

### Deterministic manifest

A structured inventory whose ordering and representation remain stable for the same eligible files and contents. Stable output makes comparisons and review easier. Determinism does not imply that the scanned directory was an atomic snapshot if files changed during traversal.

### Artifact

A file or other output produced or retained by a software, data, or research workflow, such as a build package, normalized dataset, or report. Recording paths, sizes, and hashes can support artifact comparison, but does not replace provenance records or quality checks.


<!-- publication:2026-10-19 -->

## 2026-10-19

Source note: [2026-10-19](articles/2026-10-19-synchronous-machine-transient-phenomena.md).

### Subtransient reactance

A synchronous-machine reactance used to represent the earliest response after a sudden disturbance. In the simplified physical picture, induced currents in damper and excitation windings momentarily maintain their coupled fluxes, leaving stator current to affect them through leakage paths.

### Transient reactance

A reactance used for the interval after the subtransient effects have diminished but before the machine reaches steady state. The associated approximation retains the excitation-winding flux response while representing the damper-winding contribution differently.

### Damper winding

A rotor winding or conducting structure that supports induced currents when machine flux changes. Those currents oppose rapid changes in coupled flux and influence the initial, subtransient response.

### Steady-state emf adjustment

A fictitious internal emf defined so a selected reactance, such as a subtransient reactance, can reproduce a specified steady operating point. It is an algebraic consistency device, not evidence of a physical step change in excitation.

### Zero sequence

A symmetrical-component set in which the three phase quantities are equal in magnitude and phase. In the cited machine model, its voltage relation includes stator resistance and a leakage-related inductance.

### Negative sequence

A balanced set of three-phase quantities rotating opposite to the positive-sequence direction. Relative to a synchronously rotating rotor frame, its components vary at twice the supply frequency.

### Saliency

A difference between a synchronous machine's direct-axis and quadrature-axis magnetic behavior, commonly represented by unequal axis reactances. In negative-sequence analysis it can produce waveform distortion and small harmonic components.

### Reference frame

A coordinate system used to express electrical quantities, such as stationary phase coordinates or axes rotating with the rotor. A waveform's apparent frequency can change with the frame, so the frame must be stated when interpreting transient equations.

### Deterministic inventory

A repeatable, ordered record of selected files and their attributes, such as relative paths, byte counts, and content hashes. Determinism concerns stable output for the same scanned state and rules; it does not freeze a changing filesystem during the scan.


<!-- publication:2026-10-20 -->

## 2026-10-20

Source note: [2026-10-20](articles/2026-10-20-make-financial-charts-earn-their-ink.md).

### Clarity

A chart is clear when its intended comparison or trend can be understood without guessing at the meaning of its marks, labels, or scales. Clarity comes from choosing an appropriate visual form and supplying enough context for readers to interpret it.

### Accuracy

Accuracy in visualization includes plotting the correct observations and representing them with truthful scales, units, dates, and annotations. A correct dataset can still produce a misleading chart if the visual frame distorts or obscures its meaning.

### Simplicity

Simplicity means removing elements that do not help answer the chart’s question while retaining essential definitions and context. It is purposeful reduction, not omission of information readers need to interpret the data responsibly.

### Time series

A time series is a sequence of observations indexed by time, such as monthly revenue or daily prices. A line chart often helps reveal its direction and fluctuations, but gaps, irregular timing, and reporting changes need explicit attention.

### Baseline

A baseline is the reference value from which an axis or visual mark begins. Bar charts commonly use zero because bar length is interpreted proportionally; a truncated baseline can exaggerate differences. Line charts may use a narrower range, provided it is clear.

### Logarithmic scale

A logarithmic scale places equal multiplicative changes at equal visual distances. It can help compare relative growth across magnitudes, but changes the interpretation of spacing, cannot display zero, and should be identified clearly for readers.

### Annotation

An annotation is explanatory text or a marker placed alongside a chart’s data. It can identify a reporting change, notable value, or relevant event. Annotations should distinguish documented context from causal interpretations that the chart alone cannot establish.

### Color encoding

Color encoding uses color to represent categories, highlight observations, or distinguish series. It works best when applied consistently and with adequate contrast. Pairing color with labels or other visual cues helps readers who cannot distinguish particular colors.

### Directed cycle

A directed cycle is a path of dependency edges that eventually returns to its starting node. In a task graph, it creates circular prerequisites, so no valid topological order can include all affected nodes.


<!-- publication:2026-10-21 -->

## 2026-10-21

Source note: [2026-10-21](articles/2026-10-21-reading-images-by-their-frequencies.md).

### Spatial domain

A representation of an image by values at pixel locations. Each array position corresponds to a location in the image, and its value may represent grayscale intensity or a channel value. Spatial operations act directly on these pixels or their neighborhoods.

### Frequency domain

A representation that describes how strongly different spatial patterns contribute to an image. Frequencies indicate rates of variation and directions across the pixel grid. This representation can make periodic structures and filter design easier to inspect.

### Discrete Fourier transform

A finite-array transform that expresses sampled data as discrete frequency components. Its coefficients generally contain magnitude and phase information. A matching inverse transform reconstructs the input, subject to the library's normalization convention and numerical precision.

### Fast Fourier transform

An algorithmic family for computing a discrete Fourier transform more efficiently than a direct summation for many input sizes. FFT describes how the transform is computed, not a different mathematical representation or a filtering operation by itself.

### DC component

The zero-frequency coefficient of a transform. For an image, it is related to the sum or mean pixel intensity, depending on transform normalization. Changing the image's constant brightness offset changes this component.

### Magnitude spectrum

A display or analysis of the magnitudes of Fourier coefficients, often with logarithmic scaling to show both strong and weak components. It omits phase, so it cannot alone specify the original image or its reconstruction.

### Low-pass filter

A filter that retains or emphasizes low spatial frequencies while attenuating higher ones. Its effect commonly includes smoothing, but it can also remove fine details that are meaningful, such as narrow edges or small structures.

### High-pass filter

A filter that attenuates low spatial frequencies and retains higher ones. It can emphasize rapid intensity changes such as edges, but it may also emphasize high-frequency noise and produce artifacts.

### Convolution theorem

A Fourier relationship stating that convolution in the spatial domain corresponds to multiplication in the frequency domain, with the reverse relationship applying to multiplication. This connects spatial filters with transform-based filtering and can guide implementation choices.

### Deconvolution

An attempt to infer an image before blur by reversing a convolution model. Direct inversion can amplify noise where the blur response is small, so practical methods often need regularization, a noise model, or other assumptions.


<!-- publication:2026-10-23 -->

## 2026-10-23

Source note: [2026-10-23](articles/2026-10-23-tuples-data-boundary-csv-records.md).

### Tuple

An ordered Python collection whose slots cannot be reassigned after creation. Tuples commonly represent small, fixed-shape groups, such as coordinates or a record with a known set of fields. A tuple can still contain mutable objects, so immutability of the outer grouping does not imply every nested value is immutable.

### Unpacking

Assignment that distributes elements from an iterable to multiple target names by position. For example, `a, b = (3, 4)` assigns the first element to `a` and the second to `b`. Unless starred targets are used, the number of elements must match the number of targets.

### Iterable

An object that can provide its elements one at a time, such as a list, tuple, or string. Unpacking works with iterables, not only tuples. The iterable must yield an appropriate number of values for the assignment pattern, or Python raises an error.

### CSV

Comma-separated values is a text format for tabular records, often with a header row. Real CSV syntax includes quoting and escaping rules, so splitting each line on a comma is not a reliable general parser. Python's `csv` module is designed to handle those conventions.

### Schema

A description of a dataset's expected fields, their order or names, and often their types and constraints. A stable schema lets code interpret positions consistently. Shape checks catch some schema violations, but correct field counts alone do not prove that values have the intended meaning.

### Missing value

A value absent according to a chosen data policy. In this utility, an empty field or a field containing only whitespace is counted as missing. A nonnumeric string or a nonfinite numeric token is not counted as missing, even though it is excluded from the numeric summaries.

### Finite number

A numeric value that is neither positive infinity, negative infinity, nor NaN. The profiler includes values that parse as Python floats and pass a finiteness check. This prevents nonfinite values from contaminating reported counts, sums, minima, maxima, and means.

### Descriptive statistic

A numerical summary that describes observed data, such as a mean, minimum, or maximum. Such summaries do not by themselves establish statistical significance, causality, representativeness, or scientific validity. Interpretation requires context about sampling, measurement, and the question being studied.
