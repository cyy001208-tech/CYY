---
name: research-space-article-ingest
title: Research Space Article Ingest and Argument Reconstruction
version: 2.0.0
schema_version: 2.0.0
language: en
default_output: markdown+jsonl
citation_default: GB/T 7714—2015
status: draft_for_human_review
---

# Research Space Article Ingest and Argument Reconstruction

This compatibility entry preserves the original v2 specification path. The installable, fully English Codex skill now uses the standard package layout:

- [Skill instructions](./research-space-article-ingest/SKILL.md)
- [Operational object model](./research-space-article-ingest/references/object-model.md)
- [Normative data contract](./research-space-article-ingest/references/data-contract.md)
- [Governance and validation](./research-space-article-ingest/references/governance-and-validation.md)

Together, these files replace the earlier monolithic Chinese specification without removing its core contract. They preserve:

- the untrusted-source instruction boundary;
- stable passage-level traceability;
- distinct evidence, evidence-use, claim, argument, reasoning, and warrant objects;
- version, time, fieldwork, experiment, memo, and bibliography records;
- P0-P5 staged processing;
- `00-09` machine-readable output plus a human-readable map;
- ambiguity and review queues;
- long-document coverage and resumability;
- corpus-bounded cross-record linking; and
- validation and human-approval boundaries.

## Installable package

Install or copy the `research-space-article-ingest/` directory as one skill. Keep its `references/` directory beside `SKILL.md`; the skill loads those references as one versioned contract.

## Purpose

Use the skill to transform papers, book chapters, historical editions, archival documents, research reports, field notes, interviews, experiment logs, memos, and mixed sources into a persistent research package that an AI system can revisit and a researcher can audit.

Do not use it as a detached summarizer. Its purpose is to preserve where a statement occurs, who speaks, what counts as evidence, how an argument uses that evidence, which reasoning step connects an input to an output, which version or time boundary applies, and what remains uncertain.

## Execution model

Run the six fixed stages:

1. **P0 — Intake and safety**: register files, hashes, page coverage, OCR state, damage, and instruction-boundary flags.
2. **P1 — Original, structure, and bibliography**: preserve the source, build passages, and retain original and normalized citations.
3. **P2 — Explicit objects**: extract entities, concepts, materials, processes, versions, times, evidence, observations, experiments, fieldwork, and memos.
4. **P3 — Argument reconstruction**: identify issues and claims; build arguments, reasoning steps, and warrants; preserve counterarguments, limitations, and open questions.
5. **P4 — Relations and reuse**: create reviewable relation assertions, internal links, evidence uses, proposition candidates, and deduplication candidates.
6. **P5 — Merge, coverage, and validation**: merge chunks, compare runs, report ambiguity and unresolved items, and emit evidence-backed validation results.

The P0-P5 stages describe execution order. They do not map one-to-one to the `00-09` output directories.

## Review boundary

A model may extract, reconstruct, review, dispute, or reject records within the legal state machine. A model must not assign `human_approved` or `published`. Publication requires a separate process that verifies hashes, schema conformance, citation integrity, and human review records.

## Version status

- Specification version: `2.0.0`
- Schema version: `2.0.0`
- Language: English
- Default citation style: GB/T 7714—2015
- Review state: `draft_for_human_review`
