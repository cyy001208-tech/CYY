# Data Contract

This reference defines the minimum closed schemas for Research Space Article Ingest v2.0. Add extension fields only under an explicit namespace and only after human approval.

## Contents

1. [Object envelope](#object-envelope)
2. [Intake and integrity](#intake-and-integrity)
3. [Long-source runs](#long-source-runs)
4. [Bibliography and citations](#bibliography-and-citations)
5. [Document structure and passages](#document-structure-and-passages)
6. [Semantic objects](#semantic-objects)
7. [Time](#time)
8. [Arguments](#arguments)
9. [Evidence](#evidence)
10. [Relations and unresolved reasoning](#relations-and-unresolved-reasoning)
11. [Practice records](#practice-records)
12. [Links and spatial preparation](#links-and-spatial-preparation)
13. [Quality and review](#quality-and-review)

## Object envelope

Apply this envelope to every formal object except `manifest` and `validation_result`.

```yaml
object_envelope:
  object_uuid: "UUIDv7 | null"
  provisional_key: "sha256:<source_revision + record_type + locator>"
  local_id: "string | null"
  record_uuid: "UUIDv7 | null"
  record_type: "string"
  schema_version: "2.0.0"

  provenance:
    source_revision: "string"
    extraction_run_ref: "string"
    created_by: "source_extraction | system_reconstruction | human"
    model_name: "string | null"
    prompt_version: "string | null"

  review_status: "machine_extracted | machine_reconstructed | machine_reviewed | needs_human_review | human_approved | disputed | rejected | published"
  legacy_ids: []
  field_states: []
```

## Intake and integrity

```yaml
manifest:
  schema_version: "2.0.0"
  record_uuid: "UUIDv7 | null"
  accession_id: "string | null"
  source_revision: "sha256"
  package_created_at: "ISO-8601"
  coverage_status: "complete | partial | failed"
  object_counts: {}
  output_files: []
  unresolved_count: 0
  review_status: "machine_extracted | machine_reviewed | needs_human_review"

source_record:
  identity: {}
  original_filename: "string"
  title: "string | null"
  creator_refs: []
  document_mode: "academic_article | book_chapter | monograph | historical_text | archival_document | thesis | research_report | field_note | experiment_log | memo | interview | image_based_source | mixed"
  mime_type: "string"
  file_size: "integer | null"
  page_count: "integer | null"
  language: []
  text_layer_status: "native | OCR | mixed | unavailable"
  source_revision: "sha256"
  citation_style: "GB_T_7714_2015 | Chicago_NB | Chicago_AD | MLA_9 | APA_7 | original_only"
  corpus_context:
    available: false
    registry_available: false
    searchable_record_count: 0

integrity_record:
  original_file_hash: "sha256"
  full_text_hash: "sha256 | null"
  archive_status: "complete | partial | failed"
  missing_pages: []
  unreadable_regions: []
  OCR_used: false
  OCR_engine: "string | null"
  notes: []

page_index:
  page_index_id: "string"
  physical_page: "integer"
  pdf_page: "integer | null"
  printed_page: "string | null"
  image_ref: "string | null"
  text_start_offset: "integer | null"
  text_end_offset: "integer | null"
  state: "readable | partial | unreadable | missing"
```

## Long-source runs

```yaml
extraction_run:
  run_id: "string"
  schema_version: "2.0.0"
  source_revision: "sha256"
  stage: "P0_integrity | P1_structure_bibliography | P2_passage_explicit_content | P3_argument_reconstruction | P4_semantic_linking | P5_validation"
  page_range_processed:
    start: "integer | null"
    end: "integer | null"
  section_refs: []
  total_pages: "integer | null"
  coverage_status: "complete | partial | failed"
  continuation_required: "boolean"
  previous_run_ref: "string | null"
  next_page: "integer | null"
  idempotency_key: "sha256:<source_revision + schema_version + stage + range>"
  model_name: "string | null"
  prompt_version: "string"
  started_at: "ISO-8601"
  finished_at: "ISO-8601 | null"
  errors: []

extraction_log:
  log_id: "string"
  run_ref: "string"
  event_type: "created | updated | skipped_duplicate | ambiguity_created | validation_failed | resumed"
  object_ref: "string | null"
  timestamp: "ISO-8601"
  message: "string"
```

Long-source invariants:

1. Never use `complete` before every required page or section is processed.
2. Never silently omit later content because of an output limit.
3. Record a concrete page range or section set for every chunk.
4. Use the idempotency and provisional keys to prevent duplicate objects.
5. Reconstruct whole-document arguments only after chunk-level source objects exist.
6. Link continuation runs and resume at the recorded breakpoint.
7. Produce deduplication candidates and a coverage audit during merge.

## Bibliography and citations

```yaml
reference:
  reference_id: "string"
  citation_original: "string"
  citation_normalized: "string | null"
  citation_style: "GB_T_7714_2015 | Chicago_NB | Chicago_AD | MLA_9 | APA_7 | original_only"
  reference_type: "journal_article | book | book_chapter | thesis | conference_paper | newspaper | archival_record | historical_edition | web_resource | interview | unpublished_material | unknown"
  bibliographic_fields:
    author: []
    title: "string | null"
    container_title: "string | null"
    editor: []
    translator: []
    edition: "string | null"
    place: "string | null"
    publisher: "string | null"
    year: "string | null"
    volume: "string | null"
    issue: "string | null"
    pages: "string | null"
    doi: "string | null"
    url: "string | null"
    access_date: "string | null"
  completeness: "complete | partial | fragmentary"
  verification_status: "from_document | cross_checked | partially_checked | unverified | disputed"
  missing_fields: []

citation_occurrence:
  citation_id: "string"
  passage_ref: "string"
  reference_ref: "string | null"
  note_number: "string | null"
  cited_page: "string | null"
  quotation_type: "direct | indirect | paraphrase | mention | secondary_citation"
  function: "evidence | authority | background | contrast | criticism | definition | method_source | data_source | unknown"
  link_status: "linked | unresolved | ambiguous | bibliography_missing"
```

Preserve the original citation before normalization. Never invent missing bibliographic fields.

## Document structure and passages

```yaml
section:
  section_id: "string"
  parent_section_ref: "string | null"
  level: "integer"
  heading_original: "string | null"
  heading_normalized: "string | null"
  section_function: "abstract | introduction | literature_review | source_criticism | research_question | method | historical_background | argument | case_analysis | experiment | fieldwork | discussion | conclusion | appendix | references | note | unknown"
  page_range: "string | null"
  passage_refs: []

passage:
  passage_id: "string"
  source_ref: "string"
  section_path: []
  locator:
    printed_page: "string | null"
    pdf_page: "integer | null"
    image_page: "integer | null"
    paragraph_index: "integer | null"
    sentence_start: "integer | null"
    sentence_end: "integer | null"
    char_start: "integer | null"
    char_end: "integer | null"
    footnote_number: "string | null"
    table_figure_number: "string | null"
    region_coordinates: "array | null"
  content_original: "string"
  content_normalized: "string | null"
  translation: "string | null"
  quote_hash: "sha256"
  transcription_status: "raw | checked | reviewed"
  omissions_or_damage: "string | null"
```

Keep true source wording only in `passage.content_original`. Make every claim, evidence item, and interpretation traceable through passage references.

## Semantic objects

```yaml
entity:
  entity_id: "string"
  entity_type: "person | organization | place | work | event | physical_object | institution | group | other"
  canonical_name: "string"
  aliases: []
  original_forms: []
  description: "string | null"
  identity_status: "confirmed | probable_match | possible_match | ambiguous | unresolved"
  passage_refs: []

concept:
  concept_id: "string"
  term_original: "string"
  normalized_term: "string"
  term_layer: "source_term | author_defined_term | historical_term | disciplinary_term | researcher_defined_term | system_schema_term"
  definition_passage_refs: []
  usage_passage_refs: []
  semantic_status: "explicitly_defined | implicitly_used | reconstructed | disputed"
  broader_refs: []
  narrower_refs: []
  related_refs: []
  contrasting_refs: []

material:
  material_id: "string"
  canonical_name: "string"
  aliases: []
  material_class: "fiber | pigment | dye | binder | sizing | mineral | plant | animal | synthetic | paper | ink | other"
  properties_stated: []
  passage_refs: []

process:
  process_id: "string"
  name: "string"
  process_type: "historical_technique | experimental_protocol | craft_process | analytical_method | fieldwork_procedure | conservation_process | other"
  inputs: []
  materials: []
  tools: []
  ordered_steps: []
  outputs: []
  variables: []
  failure_modes: []
  passage_refs: []

version_record:
  version_id: "string"
  work_ref: "string"
  version_type: "manuscript | block_print | movable_type | facsimile | modern_edition | translation | revised_edition | anthology_inclusion | excerpt | unknown"
  title_statement: "string"
  editor_compiler_refs: []
  publisher_or_carver: "string | null"
  place_ref: "string | null"
  time_ref: "string | null"
  holdings_or_location: "string | null"
  contents_profile: "string | null"
  passage_refs: []
  verification_status: "from_document | cross_checked | partially_checked | unverified | disputed"
```

Do not create a second unrelated entity for a concept, process, material, or version already represented by its dedicated schema.

## Time

```yaml
time_profile:
  time_id: "string"
  time_type: "composition_time | writing_time | first_print_time | first_publication_time | earliest_extant_edition_time | edition_time | copy_time | revision_time | inclusion_time | event_time | observation_time | experiment_time | recorded_time | ingested_at | reviewed_at"
  original_expression: "string"
  normalized:
    start: "YYYY-MM-DD | YYYY | null"
    end: "YYYY-MM-DD | YYYY | null"
    calendar: "gregorian | regnal | lunar | mixed | unknown"
  granularity: "day | month | year | decade | reign_period | unknown"
  qualifier: "exact | approximate | before | after | not_earlier_than | not_later_than | range"
  derivation_status: "explicit | reported | inferred | disputed | unknown"
  certainty: "certain | highly_probable | probable | possible | uncertain | unknown"
  basis_refs: []
  applies_to_refs: []
  limitations: []
```

Never equate `earliest_extant_edition_time` with `composition_time`.

## Arguments

```yaml
issue:
  issue_id: "string"
  question: "string"
  issue_type: "authorship | chronology | dating | edition_history | textual_relationship | terminology | interpretation | causation | mechanism | classification | comparison | attribution | historical_context | methodology | practice | evaluation | open | other"
  origin: "explicit | reconstructed"
  scope: "string | null"
  passage_refs: []
  parent_issue_ref: "string | null"
  principal: "boolean"

claim:
  claim_id: "string"
  quotation_refs: []
  statement_reconstructed: "string"
  representation_status: "explicit_paraphrase | reconstructed | inferred"
  claim_level: "thesis | section | local | intermediate"
  claim_type: "factual_assertion | authorship_attribution | chronology_claim | edition_claim | textual_claim | causal_claim | mechanism_claim | interpretive_claim | evaluative_claim | methodological_claim | classification_claim | hypothesis | recommendation | other"
  speaker: "paper_author | quoted_author | editor | translator | field_observer | experimenter | researcher | system_reconstruction"
  claim_status: "asserted | hypothesized | speculative | disputed"
  certainty: "certain | highly_probable | probable | possible | uncertain | unknown"
  scope: "string | null"
  modality_original: "string | null"
  principal_issue_ref: "string | null"
  parent_claim_ref: "string | null"
  produced_by_reasoning_ref: "string | null"
  used_as_premise_by: []
  verification_status: "from_document | cross_checked | partially_checked | unverified | disputed"

argument:
  argument_id: "string"
  title: "string"
  argument_type: "deductive | inductive | abductive | analogy | causal | chronological | version_history | textual_dependency | comparison | definition | classification | mechanism | authority | experiment_based | field_observation_based | cumulative_case | rebuttal | other"
  conclusion_ref: "string"
  stance: "supports | opposes | qualifies | limits | reopens | explains"
  premise_use_refs: []
  reasoning_step_refs: []
  strength: "strong | moderate | weak | indeterminate"
  limitation_refs: []
  passage_refs: []

reasoning_step:
  reasoning_step_id: "string"
  argument_ref: "string"
  input_refs: []
  output_ref: "string"
  inference_type: "deduction | induction | abductive_inference | temporal_precedence | terminus_post_quem | terminus_ante_quem | version_transmission | exclusion_by_absence | textual_dependency | causal_inference | mechanism_inference | analogy | comparison | synthesis | elimination | generalization | specification | reinterpretation | other"
  explanation: "string"
  warrant:
    statement: "string | null"
    origin: "explicit | reconstructed | inferred | not_recoverable"
    passage_refs: []
    confidence: "high | medium | low | indeterminate"
  assumptions: []
  hidden_premises: []
  confidence: "high | medium | low | indeterminate"
```

Classify an intermediate claim by its graph position, not its perceived importance.

## Evidence

```yaml
evidence_item:
  evidence_id: "string"
  evidence_type: "primary_text | direct_quotation | paraphrase | bibliographic_date | composition_date | publication_date | edition_record | edition_absence | later_inclusion | textual_parallel | textual_variant | authorship_record | historical_record | archival_record | material_object | image | table | statistic | experiment_result | field_observation | interview_statement | secondary_scholarship | author_experience | other"
  statement: "string"
  passage_refs: []
  reference_refs: []
  observation_ref: "string | null"
  experiment_ref: "string | null"
  subject_refs: []
  representation_status: "quotation | explicit_paraphrase | normalized_record | aggregated_record"
  directness: "direct | reported | indirect"
  source_level: "primary | secondary | tertiary | personal_observation"
  verification_status: "from_document | cross_checked | partially_checked | unverified | disputed"
  reliability: "high | medium | low | indeterminate"
  reliability_basis: []
  time_profile_refs: []
  version_scope: "string | null"
  limitations: []

evidence_use:
  evidence_use_id: "string"
  argument_ref: "string"
  reference_type: "evidence_item | relation_assertion | passage | claim | observation | experiment"
  reference_ref: "string"
  role: "major_premise | minor_premise | chronological_premise | version_premise | textual_premise | causal_premise | mechanism_premise | comparative_premise | corroborating_evidence | counterevidence | contextual_background | example | authority_support | aggregated_premise | other"
  interpretation: "string"
  weight: "core | supporting | contextual | marginal"
  local_confidence: "high | medium | low | indeterminate"
```

Never place system-generated content in `evidence_item`.

## Relations and unresolved reasoning

```yaml
relation_assertion:
  relation_assertion_id: "string"
  subject_ref: "string"
  predicate: "authored_by | attributed_to | compiled_by | edited_by | included_in | absent_from | copied_from | derived_from | textually_depends_on | textually_parallel_to | variant_of | earlier_than | later_than | contemporary_with | overlaps_with | first_attested_in | included_later_in | caused_by | enables | inhibits | composed_of | uses_material | located_at | observed_at | supports | opposes | qualifies | refines | reinterprets | other"
  object_ref: "string"
  statement: "string"
  basis_refs: []
  directness: "explicit | reconstructed | inferred"
  confidence: "high | medium | low | indeterminate"
  scope: "string | null"
  limitations: []

counterargument:
  counterargument_id: "string"
  target_ref: "string"
  statement: "string"
  origin: "paper_author | cited_scholar | editor | system_reconstruction"
  stance: "opposes | weakens | qualifies | alternative_explanation"
  premise_refs: []
  passage_refs: []
  response_refs: []
  status: "unanswered | acknowledged | partially_answered | answered | unresolved"

limitation:
  limitation_id: "string"
  target_ref: "string"
  limitation_type: "missing_source | incomplete_corpus | edition_uncertainty | dating_uncertainty | terminology_gap | scope_limit | method_limit | sample_limit | observational_bias | causal_gap | inferential_gap | alternative_explanation | source_reliability | time_type_mismatch | other"
  statement: "string"
  effect: "reduces_certainty | limits_scope | blocks_conclusion | requires_verification | creates_alternative"
  resulting_certainty: "certain | highly_probable | probable | possible | uncertain | unknown | null"
  resolution_status: "open | partially_resolved | resolved | unresolvable"
  passage_refs: []

open_question:
  open_question_id: "string"
  question: "string"
  arises_from_refs: []
  missing_information: []
  suggested_source_types: []
  priority: "high | medium | low"
  status: "open | researching | partially_answered | closed"
```

## Practice records

```yaml
fieldwork_session:
  fieldwork_id: "string"
  title: "string"
  date_refs: []
  place_refs: []
  participant_refs: []
  purpose: "string | null"
  source_refs: []
  observation_refs: []
  interview_refs: []
  limitations: []

experiment:
  experiment_id: "string"
  title: "string"
  purpose: "string | null"
  hypothesis_refs: []
  date_refs: []
  place_refs: []
  operator_refs: []
  materials: []
  tools: []
  process_ref: "string | null"
  variables:
    independent: []
    dependent: []
    controlled: []
    uncontrolled: []
  measurements: []
  result_summary: "string | null"
  author_evaluation: "string | null"
  anomalies: []
  limitations: []
  follow_up: []
  passage_refs: []

observation:
  observation_id: "string"
  observation_type: "field_observation | workshop_observation | laboratory_observation | object_observation | reading_observation | market_observation | interview_observation | other"
  content_original_ref: "string"
  content_normalized: "string"
  observer_refs: []
  time_refs: []
  place_refs: []
  object_refs: []
  conditions: "string | null"
  evidential_status: "direct_observation | recollection | hearsay | interpretation"
  limitations: []

memo:
  memo_id: "string"
  memo_type: "reading_note | field_note | lab_note | project_note | idea_note | conversation_note | task_note | mixed"
  title: "string | null"
  original_text_ref: "string"
  recorded_time_ref: "string | null"
  author_ref: "string | null"
  extracted_observation_refs: []
  extracted_inspiration_refs: []
  extracted_task_refs: []
  tags: []

inspiration:
  inspiration_id: "string"
  text_original_ref: "string"
  text_normalized: "string"
  status: "seed | emerging_hypothesis | hypothesis | under_test | integrated_into_claim | archived"
  origin_refs: []
  related_issue_refs: []
  related_concept_refs: []
  possible_next_steps: []
  certainty: "speculative | possible | developing | unknown"

task:
  task_id: "string"
  action: "string"
  arises_from_refs: []
  priority: "high | medium | low"
  due_time_ref: "string | null"
  status: "open | in_progress | blocked | completed | cancelled"
```

## Links and spatial preparation

```yaml
proposition_candidate:
  candidate_id: "string"
  normalized_statement: "string"
  source_claim_refs: []
  proposition_type: "string"
  scope: "string | null"
  merge_confidence: "high | medium | low"
  differences: []
  corpus_status: "local_only | awaiting_corpus_comparison | compared"
  candidate_status: "pending | approved | rejected"

internal_link:
  internal_link_id: "string"
  from_ref: "string"
  relation: "contains | part_of | follows | cites | mentions | has_argument | has_reasoning_step | uses_evidence | produced_by | responds_to | located_in | other"
  to_ref: "string"
  structural: true
  explanation: "string | null"

cross_record_candidate:
  link_id: "string"
  from_ref: "string"
  relation: "same_as_candidate | supports | opposes | qualifies | refines | extends | reinterprets | uses_same_evidence | uses_same_relation | discusses_same_issue | shares_entity | shares_concept | shares_process | possible_duplicate"
  to_ref: "string"
  explanation: "string"
  basis_refs: []
  confidence: "high | medium | low"
  candidate_status: "pending | approved | rejected"

deduplication_candidate:
  deduplication_id: "string"
  object_refs: []
  match_basis: "same_passage | same_quote_hash | same_normalized_statement | same_entity_identity | same_version_identity | other"
  similarity_score: "number | null"
  differences: []
  action: "merge_candidate | keep_separate | needs_review"
  candidate_status: "pending | approved | rejected"

spatial_ready:
  object_ref: "string"
  object_family: "literature | argument | evidence | practice | fieldwork | memo | inspiration | entity | concept | time | place"
  salience: "core | major | secondary | peripheral"
  abstraction_level: "raw_source | observation | evidence | intermediate | argument | claim | proposition"
  connectivity_hint: "hub | bridge | cluster_member | leaf | isolated_candidate"
  tags: []
```

Do not emit `x`, `y`, `z`, angle, radius, or height values in this version.

## Quality and review

```yaml
review_item:
  review_item_id: "string"
  target_ref: "string | null"
  field_path: "string | null"
  reason_type: "ambiguity | missing_source | unreadable | schema_conflict | low_confidence | unresolved_citation | duplicate_candidate | coverage_gap | security_flag | other"
  description: "string"
  priority: "high | medium | low"
  status: "open | assigned | resolved | rejected"
  related_refs: []

validation_result:
  check_id: "string"
  status: "pass | fail | partial | not_applicable"
  checked_count: "integer"
  passed_count: "integer"
  failed_refs: []
  validation_method: "schema_validation | reference_integrity_check | rule_based_check | model_semantic_review | human_review"
  explanation: "string"
  run_ref: "string"

ambiguity_record:
  ambiguity_id: "string"
  source_ref: "string | null"
  field_path: "string"
  candidate_values: []
  selected_value: "any | null"
  resolution_status: "unresolved | needs_human_review | human_resolved"
  impact: "low | medium | high"
  resolution: {}
```
