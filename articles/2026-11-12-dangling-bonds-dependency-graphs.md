# From Dangling Bonds to Dependency Graphs: How Local Constraints Shape Systems

A crystal can look orderly and still contain a bond that ends abruptly. A software build can look like a tidy list and still contain a task that cannot run until itself, directly or indirectly, has finished. In both cases, local relationships determine what the larger system can do. The physics and the software are not the same problem, but comparing them is a useful way to sharpen how we reason about structure, constraints, and emergent behavior.

## A missing bond leaves a local electronic state

In an ideal tetrahedral carbon crystal, each carbon atom forms four covalent bonds. If a bond is broken, one atom may be left with an unsatisfied bond, called a dangling bond. The source section describes this defect as producing a localized electronic level in the forbidden energy gap between the valence and conduction bands.

“Localized” matters. In an ideal periodic crystal, electronic states are commonly described as extending through the lattice as bands. A dangling bond is associated with a state concentrated near the defect instead. It is not simply another ordinary state at the edge of a band. The defect changes the local environment, and the available electronic state reflects that change.

The passage also identifies vacancies and damage from irradiation or ion implantation as other structural faults that can create similar in-gap levels. These are examples of how departures from ideal order alter the electronic structure. The precise energy and character of a defect state depend on its surroundings and on how the atoms relax after the defect forms.

One dangling bond is a local feature. If many defects are present, their associated states can interact and spread across a range of energies, forming a defect band. Disorder, including variations in bond angles, can affect how broad that band becomes. Its position is not fixed by a simple universal rule: the passage relates it to structural relaxation and to the origin of the electronic states involved.

## Charge state turns a defect into an interaction problem

The section distinguishes two charge states associated with the dangling-bond level. A neutral dangling bond has one electron. In the passage’s terminology, the corresponding lower defect band is donor-type and neutral when occupied. If the defect receives another electron, it becomes a different charge state, represented by an upper acceptor-type band that is neutral when empty.

This distinction introduces electron-electron repulsion. Putting two electrons on the same localized site costs energy. The energy separation between the two charge-state levels is identified in the passage with the Hubbard correlation energy, $U$. In the usual physical interpretation, this is an average Coulomb cost for double occupancy. For two electrons separated by distance $r_{12}$ in a simple dielectric description, the pair interaction has the form

$$
\frac{e^2}{4\pi\varepsilon_0\varepsilon_r r_{12}},
$$

where $e$ is the elementary charge, $\varepsilon_0$ is vacuum permittivity, and $\varepsilon_r$ is relative permittivity. An effective on-site $U$ averages an interaction over the relevant electronic configurations and wavefunctions, rather than treating the electrons as point charges at one fixed separation. The supplied passage’s printed formula has corrupted symbols, so the expression above clarifies the standard Coulomb form and should not be mistaken for a transcription of its typography.

A useful bookkeeping model gives a site an energy of zero when empty, an energy $\epsilon$ when occupied by one electron, and $2\epsilon+U$ when occupied by two. The extra $U$ is the cost that cannot be captured by simply assigning each electron the same independent energy. This model is a simplification, but it makes the role of repulsion concrete.

## Why independent-electron bands often work, and when they do not

The excerpt contrasts two ways to think about electrons. In a basic band picture, each electron moves in an average potential created by the lattice and the other electrons. The individual interaction between a particular pair is not explicitly followed. This approach can be effective because mobile electrons screen electric fields, and the Pauli exclusion principle limits which states electrons can occupy. The passage also invokes quasiparticles, in which the behavior of an electron is understood together with the response of its surroundings.

Yet averaging can miss important behavior when interaction energy is comparable to the kinetic energy that favors motion. Delocalization lets electrons occupy states spread through the crystal and can lower kinetic energy. Localizing charge can reduce that motion, but it can also avoid the energetic penalty of placing two electrons on the same site. The outcome depends on the competition between these tendencies, not on either one in isolation.

This competition motivates the Mott metal-insulator transition: changing conditions such as electronic concentration can shift a material between conducting and insulating behavior. In some cases, a simple band calculation predicts metallic behavior, while strong electron repulsion instead favors an insulating state. The passage gives a qualitative account, not a numerical transition criterion. Any specific threshold requires a model and material parameters that are not supplied in this section.

The text also uses uncertainty-principle language to describe the kinetic cost of localization. Confining a particle to a smaller region increases the spread in momentum, which is a useful qualitative guide to kinetic energy. It is not, by itself, a complete calculation of a solid’s energy balance. Likewise, lower permittivity can strengthen Coulomb interactions in a simple screened-interaction picture, but real materials have more detailed screening and electronic structure. These are physical intuitions, not substitutes for a material-specific model.

Finally, terminology distinguishes a Fermi gas, where electron-electron interactions are neglected, from a Landau liquid, where interactions are accounted for in a quasiparticle description. The distinction is a modeling choice about how interactions enter the description, not a claim that one kind of material contains interactions and the other does not.

## Worked example: a four-site occupancy model

Consider four equivalent localized sites, each with one available orbital in this simplified model, and four electrons. Let each singly occupied site contribute energy $\epsilon$, and let each doubly occupied site add a repulsion cost $U>0$. Compare two arrangements:

1. **One electron per site:** four singly occupied sites. The total energy is $4\epsilon$.
2. **One doubly occupied site, two singly occupied sites, and one empty site:** the electron count is still $2+1+1+0=4$. The total energy is $(2\epsilon+U)+2\epsilon=4\epsilon+U$

Care is needed: there is one doubly occupied site, not two. Its energy is $2\epsilon+U$, each of the two singly occupied sites contributes $\epsilon$, and the empty site contributes zero. Therefore the total is $4\epsilon+U$.

The second arrangement costs $U$ more than the first in this atomic-limit model. This checked arithmetic isolates the local repulsion. It does not show that electrons always distribute one per site in a real material. Hopping between sites can lower kinetic energy, and the balance between hopping and $U$ is central to many lattice models. The example simply demonstrates what the on-site interaction term counts.

## From physical constraints to workflow constraints

The analogy to software is limited but practical. In a build pipeline, a task may require generated files from other tasks. A dependency edge encodes that requirement, just as a defect’s local environment constrains which electronic states are available. A topological ordering gives one valid sequence of tasks when the graph has no cycle. A cycle represents incompatible ordering requirements, much as competing physical effects require more than a simple independent-particle description.

A deterministic ordering is useful for reproducible logs and reviews. It does not mean the chosen ordering is the only valid one, or that it optimizes runtime. It is one consistent schedule under the specified dependencies. A small command-line checker can catch cycles before an expensive build, data-processing run, or simulation begins.

That is the purpose of the companion `dependency-order` utility. It accepts a JSON mapping of task names to dependency lists, produces a deterministic topological order when possible, and reports blocked nodes otherwise. A researcher might use it to validate stages such as raw-data validation, calibration, fitting, and report generation. The utility checks order constraints only. It does not execute tasks, inspect scientific validity, or decide whether one analysis method is appropriate.

## Assumptions and common traps

Several cautions keep this discussion honest. A localized defect level is not automatically a narrow, isolated band when defects are numerous; interactions and disorder matter. A donor or acceptor label describes charge behavior in context and should not be reduced to a universal energy position. The simple $U$ model isolates double-occupancy cost, but neglects many details, including orbital structure and long-range interactions.

Likewise, a dependency graph says nothing about whether tasks are correctly implemented. A successful topological sort verifies that the stated graph is acyclic, not that every dependency was declared. If a task reads a file produced by another task but the edge is missing, the schedule may be valid and still fail in practice. Graph validation is most useful when paired with clear task definitions and tests.

## Exercises with short answers

1. **If a dangling-bond site changes from one electron to two, what new energy term appears in the simple model?** The additional on-site repulsion $U$.
2. **Does a topological order prove that a scientific workflow is scientifically sound?** No. It checks declared dependency consistency, not scientific correctness.
3. **Why can a defect band differ from a single localized level?** Multiple defect states can interact and spread in energy, with disorder and structural relaxation affecting their distribution.

The broad lesson is that local structure can have system-wide consequences. In a crystal, a broken bond can introduce states that reshape how electrons occupy and move through the material. In a workflow, a missing or cyclic dependency can derail an entire run. Understanding the local constraints is the first step toward reasoning reliably about the whole system.

---

Published 2026-11-12.

**Source:** Solid-State Physics for Electronics, section **302 Solid-State Physics for Electronics**.

This is an original explanatory note; the source book is not redistributed.
