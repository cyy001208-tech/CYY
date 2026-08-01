# Object Model

This reference defines the classification boundaries for Research Space Article Ingest v2.0. Treat every example as fictional training material.

## Contents

1. [Classification principles](#classification-principles)
2. [Core object types](#core-object-types)
3. [Representation and provenance values](#representation-and-provenance-values)
4. [Operational tests](#operational-tests)
5. [Ambiguity handling](#ambiguity-handling)

## Classification principles

- Classify by operational role, not by the noun a source happens to use.
- Preserve the exact source in `Passage`; put interpretation in typed objects that reference it.
- Store reusable material once as `EvidenceItem`; describe each argument-specific use with `EvidenceUse`.
- Attribute every claim to a speaker.
- Represent a complete argument as a conclusion plus premise uses and reasoning steps.
- Reify a content relation only when its basis can be reviewed.
- Keep system reconstruction visibly separate from source expression.
- Create an ambiguity record whenever two or more classifications remain reasonable.

## Core object types

| Schema name | Independent object | Operational definition |
|---|---:|---|
| `Passage` | Yes | A continuous, stably locatable span of text or image region that can be retrieved from a registered source without interpretive rewriting. |
| `EvidenceItem` | Yes | Reusable material, data, quotation, version information, observation result, record, or object information that can be located and described independently of one argument. |
| `EvidenceUse` | Yes | One argument's specific use, interpretation, role, and weight for a registered evidence item, relation, observation, passage, or intermediate claim. |
| `Observation` | Yes | A phenomenon, measurement, perception, or change recorded by an identifiable observer under identifiable time, place, object, or condition constraints. |
| `Claim` | Yes | A statement by an identifiable speaker that can be supported, opposed, qualified, compared, or investigated further. |
| `IntermediateClaim` | No | Represent it as `Claim` with `claim_level=intermediate`. |
| `ThesisClaim` | No | Represent it as `Claim` with `claim_level=thesis`; it directly answers the principal issue and organizes two or more important arguments. |
| `Argument` | Yes | A complete reasoning unit organized around one conclusion, with one or more premise uses, at least one reasoning step, a stance, and relevant limitations. |
| `ReasoningStep` | Yes | One inferential transformation from an explicit set of inputs to one explicit output inside an argument. |
| `Warrant` | No | Embed it in `ReasoningStep`; it states why the inputs license the output. |
| `RelationAssertion` | Yes | A reviewable assertion that two registered objects stand in a content-bearing relation, with basis, scope, directness, and limits. |
| `Counterargument` | Yes | A reasoned unit that opposes, weakens, qualifies, or offers an alternative explanation to a target claim or argument. |
| `Limitation` | Yes | A boundary on the scope, certainty, method, sample, version, source quality, or verifiability of a claim, argument, evidence item, or relation. |
| `OpenQuestion` | Yes | A precise unanswered question arising from missing, conflicting, unverified, or incomplete material. |
| `PropositionCandidate` | Yes | A normalized abstract proposition derived from one or more claims and awaiting corpus-level comparison or approval. |
| `Fact` | No | Treat factuality as a bounded property of a statement with provenance, speaker, scope, version, and verification state. |
| `Context` | No | Treat context as an information role. It becomes an argument input only through an explicit evidence use. |

## Representation and provenance values

### Representation status

| Value | Use it when |
|---|---|
| `explicit` | The speaker directly states the target content in one passage or adjacent passages, and the record adds no substantive premise, cause, or conclusion. |
| `reconstructed` | The content is not present as one complete sentence but can be recovered from local wording, references, and structure with minimal supplementation. |
| `inferred` | One or more unstated but explainable reasoning steps are required to reach the target content. |
| `speculative` | The source or system presents a possibility, hypothesis, or exploratory explanation without enough evidence for a stable inference. |

Never label a paraphrase `explicit` merely because it is plausible. Preserve quotation references and represent the paraphrase as a claim.

### Evidential directness

| Value | Use it when |
|---|---|
| `direct` | The current source directly concerns the target object or property and does not depend on another source's report. |
| `reported` | The current source quotes, paraphrases, or reports another source, observer, or researcher that has not been directly checked in this run. |
| `indirect` | Support or weakening proceeds through circumstantial evidence, a proxy, analogy, background association, or multi-step reasoning. |

### Source level

| Value | Use it when |
|---|---|
| `primary` | The source was produced by a participant, maker, institution, original edition, material object, original dataset, or direct observation process from the relevant time. |
| `secondary` | The source analyzes, interprets, compares, organizes, or evaluates primary material or prior research. |
| `tertiary` | The source compiles or summarizes secondary accounts without adding direct engagement with the primary material. |
| `personal_observation` | The evidence comes from a registered direct observation made in the current research process. |

Never use `system_generated` as a source level.

### Evaluation dimensions

| Dimension | Question answered | Typical values |
|---|---|---|
| `certainty` | How strongly does the identified speaker present the conclusion? | `certain`, `highly_probable`, `probable`, `possible`, `uncertain`, `unknown` |
| `reliability` | How strong is the bounded quality of this source or evidence under identifiable criteria? | `high`, `medium`, `low`, `indeterminate` |
| `confidence` | How sure is the classifier or reviewer that this classification, link, reconstruction, or merge is correct? | `high`, `medium`, `low`, `indeterminate` |
| `verification_status` | Which verification operation has occurred? | `from_document`, `cross_checked`, `partially_checked`, `unverified`, `disputed` |

Do not treat confidence as an objective probability or as evidence that the content is true.

## Operational tests

### Passage

Require:

- a registered source record;
- at least one stable locator such as printed page, PDF page, paragraph, character range, note number, figure/table region, or image coordinates;
- original content or a reliable transcription; and
- no model-authored explanation inside the original-content field.

Exclude:

- summaries, interpretations, and conclusions produced by the model;
- remembered wording that cannot be located; and
- a synthetic paragraph stitched from non-contiguous locations.

Example: preserve the exact paragraph on PDF page 7 that describes the 1874 Green Court edition, along with page and character offsets.

Non-example: store “the author thinks Shen Heng proposed it first” without an exact source span.

If the precise range is unstable, preserve the best available locator and create a review item. Never fabricate precision.

### EvidenceItem

Require:

- at least one passage, observation, experiment result, reference, or reviewed relation assertion;
- a description that remains meaningful outside the current argument;
- provenance from source material or a registered observation or experiment; and
- source level, directness, verification state, and scope.

Exclude:

- the author's or system's final conclusion;
- an unstated premise supplied by the model;
- an argument-specific role, which belongs in `EvidenceUse`; and
- background added only from model memory.

Example: register the absence of “Rose Grafting Methods” from the table of contents of the earliest surviving *Northern Garden Manual* edition, linked to the edition record and passage.

Non-example: “therefore the chapter was added later.” That is a claim produced by reasoning.

If a sentence could be either material or author judgment, preserve the passage and create competing `EvidenceItem` and `Claim` candidates in an ambiguity record.

### EvidenceUse

Require:

- a target argument;
- a reference to one callable registered object;
- a role, interpretation, weight, and local confidence; and
- a reference rather than a duplicated copy of the evidence text.

Example: use an 1874 edition record as a `chronological_premise` in one argument.

If it plausibly serves two roles, record role candidates and require review.

### Observation

Require:

- an identifiable observer or recording agent;
- the best available time, place, object, and conditions;
- a distinction among direct observation, recollection, hearsay, and interpretation; and
- a passage, memo, field note, or experiment record.

Exclude:

- universal conclusions detached from the observation conditions;
- common-knowledge assertions with no observer or record; and
- details invented from an image or partial text.

Example: record that rose cuttings produced more roots after willow extract was applied in April 2021 at the fictional Green Garden, including the observer and experimental conditions.

Non-example: “willow extract always improves rooting in all plant cuttings.” That is an overgeneralized claim.

Split mixed observation and interpretation into separate observation and claim records. If the split is not defensible, preserve the wording and register ambiguity.

### Claim

Require:

- an identifiable speaker;
- quotation references or an explicit `system_reconstruction` marker;
- preserved modality, scope, version, and qualification; and
- a statement that can meaningfully be asked, “What supports this?” or “How could this be opposed?”

Exclude:

- bare quotations, data points, dates, and table-of-contents entries;
- labels that only describe a section topic;
- conclusions added to complete a graph; and
- floating statements with no identifiable speaker.

Use `explicit_paraphrase` only when the paraphrase is directly supported by linked wording. Use `reconstructed` when locally distributed wording must be combined. Use `inferred` only when a reasoning step is required.

### Intermediate claim

Use `claim_level=intermediate` only when both conditions hold:

1. A reasoning step produces the claim.
2. Another argument then uses the claim as an input.

Do not use importance, paragraph position, or rhetorical prominence as the criterion.

### Thesis claim

Use `claim_level=thesis` only when the claim:

- directly answers the principal issue;
- governs two or more substantial arguments; and
- has a scope that matches the whole source rather than a local section.

Do not force a thesis onto a source that remains exploratory, fragmentary, or multi-voiced.

### Argument

Require:

- exactly one conclusion reference;
- one or more premise-use references;
- at least one reasoning step;
- a stance toward the conclusion or target; and
- relevant limitations.

Exclude:

- a list of evidence with no inferential connection;
- a single claim with no supporting structure;
- a whole paper treated as one undifferentiated argument; and
- a topic cluster created from word similarity.

If the warrant cannot be recovered, record `not_recoverable`; do not invent one to make the argument appear complete.

### ReasoningStep

Require:

- one parent argument;
- explicit input references;
- one explicit output reference;
- a named inference type;
- a transparent explanation; and
- a warrant with origin and confidence.

Split a chain with multiple outputs into multiple reasoning steps.

### Warrant

Store the warrant inside the reasoning step. Label its origin:

- `explicit`: the source states the bridge;
- `reconstructed`: local structure supports a minimal recovery;
- `inferred`: the bridge is an analyst inference;
- `not_recoverable`: no defensible bridge can be recovered.

Do not present a reconstructed warrant as author wording.

### RelationAssertion

Require:

- registered subject and object references;
- a specific predicate;
- a reviewable statement;
- basis references;
- directness, confidence, scope, and limitations.

Use it for relations such as attribution, inclusion, absence, derivation, textual dependence, chronology, causation, material use, support, opposition, qualification, or reinterpretation.

Use `InternalLink` instead for purely structural containment, navigation, or file organization.

Never use `related_to` as an escape hatch. If no specific relation can be defended, create an ambiguity or open question.

### Counterargument

Require a target and at least one premise or reason. Record whether it opposes, weakens, qualifies, or supplies an alternative explanation. Keep its response and resolution state separate.

Do not label a bare contradiction a counterargument unless it contains an identifiable reason.

### Limitation

Use a limitation to state how missing sources, incomplete corpus coverage, version uncertainty, dating uncertainty, terminology gaps, scope, method, sample, observation bias, causal gaps, inferential gaps, alternative explanations, or source quality change what may be concluded.

Record whether the limitation reduces certainty, limits scope, blocks a conclusion, requires verification, or creates an alternative.

### OpenQuestion

Write an explicit question that arises from registered objects. List missing information, useful source types, priority, and status.

Do not use an open question as a vague container for “more research is needed.”

### PropositionCandidate

Normalize a proposition only when one or more source claims support the abstraction. Preserve source claim references, scope, differences, and merge confidence.

Keep it `awaiting_corpus_comparison` when no searchable corpus is available. Never invent a global proposition identifier.

### Fact

Do not create a standalone `Fact` object. Represent the statement with:

- source and passage;
- speaker;
- scope and version;
- directness and source level;
- verification status; and
- any limitation or dispute.

“Fact” in the source's prose does not exempt the statement from provenance and review.

### Context

Use context for information that helps interpret a period, object, concept, or argument but does not serve as a direct premise in the current argument. When an argument calls it, create an `EvidenceUse` with `role=contextual_background`.

## Ambiguity handling

Create an `ambiguity_record` when:

- two object types remain plausible;
- speaker identity is unresolved;
- a version or date has competing readings;
- a relation predicate cannot be selected confidently;
- multiple reconstructions fit the same passages; or
- a classification depends on an unverified assumption.

Record:

- the target reference;
- competing interpretations;
- supporting and opposing passage references for each;
- why the ambiguity cannot yet be resolved;
- the next verification action; and
- review priority.

Do not resolve ambiguity merely to satisfy schema completeness.
