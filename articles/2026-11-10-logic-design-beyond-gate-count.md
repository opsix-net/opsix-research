# Logic Design Beyond the Gate Count

A Boolean expression with fewer terms looks like a clear win. Then the parts list arrives: the “simpler” circuit needs an extra chip, more board space, and a tangle of wires. That mismatch is the practical lesson at the center of integrated-circuit logic design: a low gate count is not automatically a low-cost implementation.

In section 5.8 of *Digital Principles and Logic Design*, the focus shifts from minimizing Boolean expressions to choosing and using medium-scale and large-scale integration (MSI and LSI) devices. The central idea is straightforward but easy to overlook: optimize the circuit that can actually be built from available packages, not just the abstract diagram made of logic symbols.

## The objective changes when gates come in packages

A combinational circuit maps current input values to output values without storing a history of earlier inputs. One familiar design route begins with a truth table, derives Boolean functions, and simplifies them. Fewer literals and logic gates can mean fewer components, so this is a useful method and often a good starting point.

But an integrated circuit does not necessarily contain exactly the gates a designer needs. A package may contain several copies of a gate type. If a design uses one gate from a package, the unused gates still occupy that package. A second design with more total gates might fit into fewer packages by using spare gates already included in those packages.

That changes the cost model. Instead of counting only logic gates, a practical comparison can include:

- The number and type of IC packages.
- Whether the required gate types are available in those packages.
- How many package pins must connect externally.
- The number and complexity of wiring connections.
- Board area, power, timing, availability, and other application constraints.

These considerations are related, but they are not interchangeable. Fewer packages do not automatically mean lower power or better timing. Those properties depend on the particular devices and circuit. The section’s core point is narrower: gate count alone cannot establish which integrated-circuit implementation is more economical.

## Why a larger gate count can use fewer chips

The section illustrates its argument with two implementations of an Excess-3-to-BCD code converter. It says both implementations use 13 logic gates. Under the package capacities listed in the passage, one implementation requires six ICs: one 3-input OR package, one 3-input AND package, two 2-input AND packages, one 2-input OR package, and one inverter package. The other requires four: two 2-input AND packages, one 2-input OR package, and one inverter package.

The comparison is not that one design has fewer logic gates. Both have 13. The difference is how the gates map onto the available packages and how the implementations arrange their connections. The passage concludes that the four-package implementation is more economical in this comparison. It also emphasizes internal connections within ICs, which can reduce the wiring needed between package pins.

The package capacities in the example matter. A package with multiple gates can leave unused gates behind, and the designer may be able to use those gates elsewhere in the same circuit. But this is not a universal guarantee: gate type, input count, pin availability, and circuit connectivity determine whether those spare gates are useful. A spare inverter cannot replace a needed AND gate merely because it is available.

A careful designer therefore asks two questions, not one. First, what Boolean function must the circuit implement? Second, what arrangement of actual devices implements it with acceptable package count and interconnection burden?

## Start with the function, then inspect the device landscape

The section recommends checking whether the required function is already available as an IC. Commercial devices may implement particular functions directly. If a device exactly matches the requirement, it may avoid building the same behavior from smaller gates. If no exact match exists, a designer can consider combining devices that are close to the target function.

This is a useful design habit well beyond digital logic: look for a suitable building block before constructing one from primitives. The comparison still needs engineering judgment. A dedicated device may have constraints that make it unsuitable, such as incompatible logic levels, timing, package, or availability. “Functionally similar” is not the same as “safe to substitute.” Confirm the full requirements before selecting a part.

The passage groups integration scales by the number of logic gates in a package:

- SSI, or small-scale integration: up to ten gates in one package.
- MSI, or medium-scale integration: eleven to one hundred gates in one package.
- LSI, or large-scale integration: more than one hundred gates in one package.

It also mentions VLSI devices, which can provide dedicated functions and substantial reductions in circuit space and interconnection. These categories describe integration scale, not a promise that a particular package is the best choice. The precise device still matters.

## A worked package comparison

Consider a hypothetical controller made from a logic network of 18 gates. These numbers are invented for this worked example, not taken from the section. Suppose implementation A uses 18 individual single-gate packages. Implementation B uses two packages, each containing ten gates of the needed type, and its connections allow all 18 required gates to be used across those packages.

At the package level, implementation A uses 18 packages. Implementation B uses two. The arithmetic is simply $18 - 2 = 16$ fewer packages, or $16 / 18 = 8/9$, about $88.9\%$ fewer packages. This does not prove that B is cheaper overall. The comparison has not assigned prices, counted wiring, checked unused gate types, or considered timing, power, or package availability. It only demonstrates why gate count and package count can tell different stories.

Now add a constraint: the only available ten-gate package has gates with a different input count from the controller’s required gates. The assumed mapping may fail. Likewise, if a design needs 11 gates of one kind and the package contains ten, it cannot fit into one package just because the total gate count is small. A realistic estimate groups required gates by type and checks that each available package can implement the needed functions and connections.

A useful comparison table would therefore record, for each candidate design, its gate types and counts, package types and quantities, unused gates, inter-package signal connections, and relevant electrical constraints. That turns “this diagram looks smaller” into a testable implementation choice.

## When classical simplification helps, and when to look wider

The section does not reject Boolean simplification. It describes the classical method as general, understandable, and capable of producing a result. It also notes its limits: for many input variables, truth tables and simplification can become cumbersome, and the resulting circuit may require many ICs and interconnecting wires. An alternative design approach can sometimes produce a better practical circuit, depending on the application and the designer’s experience.

That is a call to compare methods, not a claim that one method always wins. The most suitable design might come from simplification, a dedicated MSI or LSI device, a decomposition into functional blocks, or a combination. The application determines what “better” means. A low package count may be valuable in a space-constrained product, while a design that is easier to test or source may be preferable in a research prototype.

The general workflow is:

1. Specify the input-output behavior and constraints.
2. Check whether an available IC directly provides the needed function.
3. If not, identify near-match devices and verify whether they can be combined safely.
4. Develop one or more candidate logic implementations.
5. Map each candidate to actual package types and capacities.
6. Compare package quantity, external connections, and application-specific requirements.
7. Verify the selected design against its functional and electrical specifications.

This workflow is not a substitute for device datasheets or circuit validation. It is a way to avoid optimizing a drawing while ignoring the parts that will go on the board.

## Common traps in practical comparisons

**Treating every gate as equally available.** Gate functions and input counts differ. Package capacity is useful only when the enclosed gates fit the design.

**Counting unused gates as a design failure.** A package may have spare gates because that is how the device is sold. The relevant question is whether the package-level implementation is acceptable, not whether every gate is used.

**Equating fewer packages with lower total cost.** Package count is an important factor in the passage’s comparison, but total cost and suitability can depend on wiring, board area, power, speed, availability, and other requirements.

**Assuming a near-match device is interchangeable.** Check behavior, pinout, logic levels, timing, and operating conditions. Similar names do not guarantee compatible circuits.

**Applying a method mechanically to a large problem.** A procedure that works well for a small truth table may become unwieldy as the number of inputs grows. Compare alternatives rather than assuming the first valid solution is the best implementation.

## Exercises with short answers

**1. A design uses 12 gates and fits into two available six-gate packages. Does that establish that it is cheaper than a design using 14 gates in one package?**

No. It establishes a package-count difference under the stated assumptions, but price, gate types, wiring, and other constraints still need comparison.

**2. Why can an unused gate in a package still be economical?**

The package is purchased as a unit. If using its other gates avoids buying another package, the unused gate may not make the overall implementation less economical.

**3. What should a designer do if no commercial IC exactly matches the required function?**

Investigate devices that are close to the function and determine whether they can be combined to meet the full functional and electrical requirements.

## The practical takeaway

Boolean simplification is a valuable tool, but an integrated-circuit design is not priced by its algebra alone. Package contents, gate types, internal connections, and external wiring all affect the implementation. Section 5.8’s lasting lesson is to keep both views in focus: derive a correct logic function, then inspect how real devices can realize it. The best circuit is not automatically the one with the fewest symbols. It is the one that meets the application’s requirements with a sensible, verified implementation.

---

Published 2026-11-10.

**Source:** Digital Principles and Logic Design, section **5.8 COMBINATIONAL LOGIC WITH MSI AND LSI**.

This is an original explanatory note; the source book is not redistributed.
