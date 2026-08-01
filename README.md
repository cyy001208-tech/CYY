# Research Space Article Ingest

> Reconstruct research material as traceable evidence and argument—not as a
> detached summary.

`research-space-article-ingest` is an installable Codex skill and specification for turning papers, book
chapters, historical editions, research reports, field notes, interviews,
experiment logs, and memoranda into a persistent research package that an AI
system can revisit and a researcher can audit.

The repository publishes a **v2.0.0 specification**, not a compiled application
or a complete parsing engine.

## The Problem

A conventional summary often preserves what a source appears to say while
losing:

- where the statement appears in the original;
- who is speaking;
- how a piece of evidence is used;
- which reasoning step connects evidence to a claim;
- what remains uncertain;
- which edition, version, or time boundary applies;
- whether a record has been machine-reviewed or human-approved.

This specification keeps those layers separate and traceable.

## Core Objects

| Object | Function |
|---|---|
| `Passage` | Stores a stable, retrievable span of source text or image region. |
| `EvidenceItem` | Registers reusable evidence independently of a particular argument. |
| `EvidenceUse` | Records how an argument uses an EvidenceItem. |
| `Observation` | Preserves observer, time, place, object, and conditions. |
| `Claim` | Stores a statement that can be supported, challenged, or qualified. |
| `Argument` | Organizes premises, reasoning, conclusion, and position. |
| `ReasoningStep` | Represents one transformation from explicit inputs to an output. |
| `RelationAssertion` | Stores a reviewable content relation between registered objects. |

## P0–P5 Processing Pipeline

1. **P0 — Intake and safety**
   Register the file, hash, page count, OCR state, and missing regions. Treat
   source documents as untrusted data.
2. **P1 — Original, structure, and bibliography**
   Preserve the source, create stable Passages, and separate original citation
   text from normalized citation.
3. **P2 — Explicit objects**
   Extract people, organizations, places, works, concepts, materials,
   processes, editions, time records, evidence, observations, and experiments.
4. **P3 — Argument reconstruction**
   Identify Issues and Claims; build Arguments, ReasoningSteps, and Warrants;
   retain counterarguments, limitations, and open questions.
5. **P4 — Relations and reuse**
   Create RelationAssertions, InternalLinks, and EvidenceUses. Do not invent
   cross-record targets when no searchable corpus exists.
6. **P5 — Merge, coverage, and validation**
   Merge chunked runs, compare repeated extraction, report ambiguity, review
   queues, unresolved items, and validation results.

The P0–P5 stages describe execution order. They do not map one-to-one to the
final output directories.

## Output Package

```text
00_manifest.*          intake, versions, and coverage
02_original_archive/   original source, integrity, and page index
03_bibliography/       original/normalized citations and occurrences
04_structure/          document structure and Passages
05_argument/           Issues, Claims, Arguments, reasoning, and evidence use
06_semantics/          entities, concepts, materials, processes, editions, time
07_practice/           fieldwork, experiments, observations, memos, tasks
08_links/              proposition, link, and deduplication candidates
09_quality/            run logs, ambiguity, review queues, validation results
10_human_readable_map.md
```

The `00–09` package supports machine reading and programmatic validation.
`10_human_readable_map.md` provides a researcher-facing overview.

## Safety and Review Boundaries

- Uploaded files, OCR, web archives, images, footnotes, and attachments are
  untrusted research data.
- A model reconstruction cannot become its own evidence.
- When no registry exists, use provisional keys rather than fabricating UUIDs.
- When no global corpus exists, do not fabricate cross-record links.
- A model cannot assign `human_approved` or `published` status.
- Privacy, access control, copyright, quotation length, and fieldwork consent
  require separate policies.

## Current Status

- Specification version: `2.0.0`
- Schema version: `2.0.0`
- Default citation format: GB/T 7714—2015
- Review state: `draft_for_human_review`
- Skill and specification language: English

## Related Skills

This repository provides source and argument infrastructure. It can support,
but is independent from:

- [`personal-practice-curatorial-research`](https://github.com/cyy001208-tech/personal-practice-curatorial-research)
- [`portfolio-exhibition-editor`](https://github.com/cyy001208-tech/portfolio-exhibition-editor)

## Repository Contents

```text
README.md
research-space-article-ingest.skill.v2.md
research-space-article-ingest/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── object-model.md
    ├── data-contract.md
    └── governance-and-validation.md
CITATION.cff
CITATION.bib
docs/index.html
.github/workflows/pages.yml
```

## License Status

No open-source license has been granted yet. Public visibility does not by
itself grant permission to copy, modify, or redistribute this work.
