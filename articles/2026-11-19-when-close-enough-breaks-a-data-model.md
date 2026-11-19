# When “Close Enough” Breaks a Data Model

Imagine a search system that treats “Java developer” and “C++ developer” as interchangeable. A recruiter gets more results, perhaps, but also loses the distinction the search was meant to preserve. Now imagine a data integration job that declares two records identical because their names look alike. That single shortcut can spread incorrect properties through a knowledge graph.

These are versions of one engineering problem: a relationship in a model says more than its author intended. Chapter 7, “Bad Semantics,” of *Semantic Modeling for Data: Avoiding Pitfalls and Breaking* examines this problem through identity, synonymy, mappings, and class hierarchies. Its central warning is practical: a label or link is not harmless metadata. The formal meaning attached to it can shape search results and logical inferences.

## Similar words are not necessarily synonyms

A semantic model connects concepts to words people use for them. If several words are recorded as synonyms of one concept, a search system may treat them as interchangeable. That can be useful when the equivalence is sound. It is risky when the terms are merely related.

The distinction is easy to blur. Someone looking for a Java specialist might also consider a C++ specialist, but the languages are not synonyms. One term does not mean the other, even if the skills overlap. Similarly, two job titles may occur in related contexts without denoting the same role. A search expansion can intentionally include related concepts, but that is a different relation from synonymy and should be represented or handled separately.

The chapter describes several routes to bad synonyms. A team may avoid creating a separate concept because writing definitions, translations, and relationships takes effort. It may reuse a search synonym ring, whose purpose is to improve retrieval, as if it expressed strict equivalence. It may mistake lookalike words across languages for equivalents, or overlook context and specialist usage. Even familiar terms change meaning between communities: “holiday,” for example, does not have identical usage in every variety of English.

A useful test is substitution. If a user or system can replace term A with term B in the relevant contexts without changing the intended meaning, synonymy may be appropriate. If the replacement works only for one task, audience, or context, record that limitation rather than implying universal equivalence. When the distinction is uncertain, a broader relation such as “related to” may be more honest than “same meaning.”

## Worked example: reviewing a search vocabulary

Suppose a research library indexes software methods and receives a proposal to treat “graph database,” “knowledge graph,” and “RDF store” as one synonym group. The goal is to help researchers find material when they use different terminology. Before accepting the group, reviewers inspect the intended meaning and the search behavior.

First, define the target concept. The library decides that its concept is “software for storing and querying data represented as a graph.” It then checks each proposed label against that definition. “Graph database” is a plausible label. “RDF store” is a more specific kind of graph-oriented data store, so it may be a narrower concept rather than an interchangeable name. “Knowledge graph” can describe a data structure or a broader representation, not necessarily the storage software itself.

Next, test the consequences. If a search for “RDF store” returns every document tagged only as a general graph database, is that acceptable? If a query for “knowledge graph” is meant to find papers about graph-shaped data regardless of storage technology, collapsing the terms could hide useful distinctions. The reviewers therefore keep “graph database” as a label for the defined concept, and create or retain separate concepts for RDF stores and knowledge graphs, linked by an explicitly weaker relation where appropriate.

The arithmetic of the review is simple but helps make the decision operational. Imagine there are 120 documents tagged with the broad graph-database concept, of which 35 are also tagged specifically as RDF stores. If the model treats the narrower term as a synonym, a query for “RDF store” may retrieve the 120 broad results, not just the 35 specific ones. That is 85 additional documents, calculated as $120 - 35 = 85$. Those results are not necessarily wrong for exploratory search, but the expansion must be a search policy, not a claim that the terms have identical meaning. The counts describe this imagined collection only; they do not prove that one vocabulary design is universally best.

Finally, the team records its criterion, intended search behavior, and known boundary cases. A second reviewer checks the full set of terms together, rather than judging each pair in isolation. This can reveal inconsistencies: perhaps A seems equivalent to B and B to C, while A and C clearly differ. Set-level review helps expose that tension before it becomes a persistent model defect.

## Identity links carry strong consequences

Mappings connect concepts across independently developed models. The chapter emphasizes that not all mapping relations mean the same thing. In particular, `owl:sameAs` is a strong identity claim: the linked resources are the same, and statements about them can be treated as statements about one another. That can produce useful integration, but it can also import assertions that do not fit the local model.

This issue is sometimes called referential opacity. Two resources may point to the same real-world subject while carrying properties or relations that one model does not accept or intend. Treating them as identical can therefore make the local graph inherit surprising statements. Similarity is also not identity, and identity may be context-dependent. Each case calls for checking what the relation means and what reasoning follows from it.

The chapter also distinguishes common OWL and SKOS choices. `owl:sameAs` is not interchangeable with `owl:equivalentClass`: the latter states that two classes have equivalent extensions, while the former asserts identity between resources. Nor is `owl:sameAs` the same as `skos:exactMatch`; the chapter notes that `skos:exactMatch` does not require or imply the statement merging associated with identity. Choosing a familiar-looking predicate without checking its formal semantics is a way to make an unintended inference part of the system.

A sensible integration workflow is selective. Inspect the external model, confirm that its meaning fits the local use, and import only the assertions the project accepts. A link is not an obligation to adopt every label or statement from the linked source. If the evidence supports only a weaker relationship, do not promote it to identity just to make the mapping look tidy.

## Class hierarchies must distinguish kinds from instances

A subclass relation makes a logical commitment. If class A is a subclass of class B, every instance of A is also an instance of B. An individual entity, such as a particular city, is not ordinarily a subclass of the class “City.” It is an instance of that class. Confusing these relations can distort a hierarchy and generate nonsensical inferences.

The phrase “is a” contributes to the confusion. “Mira is a biologist” describes an individual as an instance of a type. “A biologist is a scientist” describes a type as a subclass of another type. The same words can express different relationships, so modelers and automated extraction systems need context, not just a phrase-matching rule.

The excerpt also notes that some modeling approaches restrict second-order classes, where instances of a class are themselves classes. That constraint can affect how people represent types, but it does not make an individual-to-class mistake correct. The right relation depends on what the nodes represent and on the modeling language’s rules.

## Make review part of the modeling process

The chapter’s guidance can be translated into a practical review routine:

1. **Write the intended meaning.** State what each concept and relation means, including the context in which a label applies.
2. **Separate equivalence from usefulness.** A term that improves recall may be related, narrower, or broader without being synonymous.
3. **Review with multiple people.** Synonymy can be vague at the edges. More than one informed judge can reveal disagreement; agreement measures can help identify borderline cases, though they do not decide meaning automatically.
4. **Evaluate sets, not only pairs.** Check whether a group of labels remains coherent as a whole.
5. **Inspect imported semantics.** Check external definitions, predicates, and inferences, then select what to reuse.
6. **Document decisions.** Record assumptions and domain-specific conventions, and apply them consistently.

This work is not a demand for maximal detail. A project may deliberately merge distinctions that its users do not need. The important point is to make that choice explicit and consistent, rather than letting an accidental synonym or mapping silently dictate behavior.

## A practical link to asset pipelines

The same habit of checking what a relationship actually asserts applies outside knowledge graphs. In an asset pipeline, two files with matching names are not necessarily duplicates, and two byte-identical files are not necessarily safe to remove. A duplicate report should state evidence, not take an irreversible action on someone’s behalf.

The accompanying `duplicate-report` utility follows that conservative pattern. It groups files only when their byte lengths and SHA-256 digests match, reports the paths and estimated redundant bytes, and never deletes anything. This is not a semantic-modeling feature, and the chapter does not prescribe it. Rather, it is a practical engineering workflow inspired by the broader lesson: distinguish observation from interpretation, make assumptions visible, and leave consequential decisions to a reviewed process.

## Exercises with short answers

**1.** A model links “pediatrician” and “doctor.” Should they be synonyms?  
**Answer:** No. A pediatrician is a kind of doctor, so this is a narrower-to-broader relationship, not equivalence.

**2.** Two datasets describe the same city, but one includes a local administrative category the other does not use. Is `owl:sameAs` automatically safe?  
**Answer:** No. Check whether identity-based statement sharing would import claims that are invalid or unsuitable in the receiving model.

**3.** A search team expands queries from “Python” to “Java” because both are programming languages. What is the modeling risk?  
**Answer:** Search usefulness is being confused with synonymy. Keep the concepts distinct and implement expansion as an explicit retrieval policy if desired.

Good semantic models do not make every distinction imaginable. They make the distinctions their users need, state what their relations mean, and resist turning convenience into false certainty. That discipline keeps search useful and inference trustworthy.

---

Published 2026-11-19.

**Source:** Semantic modeling for data - avoiding pitfalls and breaking -- Panos Alexopoulos; Safari, an OReilly Media Company, section **Chapter 7. Bad Semantics**.

This is an original explanatory note; the source book is not redistributed.
