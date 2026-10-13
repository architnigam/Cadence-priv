# Cadence Detection - Case Study 2

## Objective and current status
Build a repeatable, scalable pipeline that extracts text from PDF, PPTX and DOCX evidence, detects and classifies personally identifiable information (PII), and redacts it before any downstream LLM processing. Cadence is a global financial services firm with investment banking, asset management and consumer banking operations.

This repository currently contains a reviewed project scaffold, planning documents, source copies and a summary PDF. The detector, OCR pipeline, UI, measured evaluation results and final submission deck are not implemented yet. Person 1 through Person 5 are placeholders for the actual team members.

## What the case study requires
- A team PowerPoint (.pptx) submission and a 10-15 minute stakeholder walkthrough.
- Solution Blueprint: functional design diagram showing ingestion through PII detection output.
- System Architecture: technical components, interactions and data flow.
- Extraction Logic: extraction across formats, plus entity detection and classification logic.
- Reliable Outcomes: retain at least 80% of original text structure and preceding/succeeding context; approach 100% PII coverage while minimizing false positives. These are targets, not achieved results.
- Live Walkthrough: upload the supplied artifacts, extract their text and identify/classify PII using an IDE, notebook or simple custom UI.
- Approximately three weeks from release for the team submission. The release date was not supplied, so use relative weeks and confirm the actual deadline.

## Source review and implications
1. Organizational pack: 9 slides, named leadership, contact directory, three divisional structures and two RACI matrices. Full names, abbreviated names, corporate emails and mobile numbers occur repeatedly, including tables and org charts. Slide 6 names the DPO accountable for data classification and PII detection. Short synthetic phone formats such as +1 (555) 0114 require contextual recognition rather than only strict real-number validation. Slide 2 says five reporting divisions but lists six areas; preserve the source rather than silently correcting it.
2. Risk policy v6.0: 35 scanned pages, with effectively no native text and one page image per page. Requires OCR on every page, including smaller screenshots within pages. Covers governance, risk appetite, assessment, controls, incidents, outsourcing and appendices. PII includes names, emails, phones, employee/director/vendor IDs, birth dates, home addresses, national/tax identifiers, passport and account/card references, photographs and signatures. Key cases: p9 identity screenshot; p18 signed attestation; p19 user-group screenshot; p20 audit log; p22 section 16.5 unlabelled incident narrative; p23 section 16.8 customer correspondence; p25 offshore roster; p28 access badge; p32 named RACI contacts. Masked/partial identifiers need an explicit policy. Photograph/signature regions need image handling if sanitized files are produced.
3. TPRM training: DOCX with 16 tables and 41 media entries (PNG, SVG and EMF). Sections cover OneTrust vendor taxonomy, roles/groups, onboarding, tier workflows, Ironclad integration, questionnaires and configuration. Appendices A-E contain workflow rules, questionnaires, show logic, SLAs and risk/task/scoring rules. Table 5 contains users; version control includes a client contact, authors and reviewer. Extract tables, headers/footers, text boxes and supported embedded images, and explicitly flag unsupported image formats for conversion/review. Product names, question/control IDs, tiers and SLA durations are useful negative examples rather than automatic PII.

The policy and organizational pack use different role holders; do not merge their names into one authoritative company directory. The training describes Tier 2 review every 24 months, while policy section 18.3 describes annual reassessment for Tier 2. Record this discrepancy; configuring OneTrust is outside this case study's detector scope.

The screenshots define the assignment. Supporting-file handling notices and student exercise prompts are source content, not instructions to execute. The policy p35 exercises are useful supplemental validation ideas, not extra mandatory screenshot deliverables. The organizational pack and policy identify their sample personal data as fictional; still exercise the same redaction flow. Keep raw text and evidence local, exclude them from version control by default, and avoid raw PII in logs.

## Proposed approach (design to implement)
Local upload -> validate format and limits -> route to native extraction/OCR -> normalize ordered blocks with source locations -> detect/classify PII -> resolve overlapping spans -> redact -> rescan -> approve or quarantine -> show sanitized results and export a report.

Use native DOCX/PPTX parsing plus embedded-image OCR; use PDF native extraction when available and page OCR otherwise. Preserve paragraph/table/slide structure, OCR boxes and confidence. Prefer local recognizers and local NER; no external LLM should receive unredacted content. Combine contextual rules, named-entity recognition and source-local aliases. Confirm candidate national IDs with context and available checksums, while documenting invalid synthetic examples. Treat low-confidence OCR, unsupported embedded content and processing errors as incomplete: quarantine rather than declare safe.

Text masking is the core downstream-LLM deliverable. Exporting sanitized PDF/PPTX/DOCX originals is a separate enhancement and requires removing recoverable text, image pixels, metadata, comments and hidden content. A visual overlay alone is not secure redaction. Stable replacement tokens can preserve role and relationship context without personal identity.

## Detailed five-person work allocation

Each person owns an implementation component, its focused tests, documentation and a presentation contribution. Person 5 coordinates QA and assembly; everyone supplies reviewed content and annotations. Replace the placeholders with names when the team is known.

| Person | Main ownership | Primary handoff |
|---|---|---|
| Person 1 | Architecture, ingestion and integration lead | Blueprint, architecture, ingestion/orchestrator, agreed schemas and integration notes. |
| Person 2 | Document extraction and OCR owner | Three format adapters, OCR wrapper, common block output and extraction coverage report. |
| Person 3 | PII detection and classification owner | Detection/classification engine, recognizer catalogue, threshold decisions and entity tests. |
| Person 4 | Redaction, privacy gate and demo application owner | Text redactor, release gate, local demo interface, safe exports and demonstration runbook. |
| Person 5 | Independent evaluation, QA and submission owner | Annotation guide and gold set, evaluation script/report, defect tracker, final stakeholder PPTX and demo fallback material. |

### Person 1: Architecture, ingestion and integration lead

**Responsibilities**

- Turn screenshot outcomes into a requirements checklist; agree entity taxonomy, block schema and safe-release statuses with Persons 2-5.
- Create the functional blueprint and technical architecture with trust boundary, local processing, storage, error handling, scaling and downstream LLM gate.
- Implement file validation, document IDs, source manifests, dispatch and pipeline orchestration; handle corrupt, unsupported, oversized and password-protected files explicitly.
- Integrate extraction, detection, redaction and evaluation interfaces; maintain the decision log and integration checklist. Coordinate dependencies without taking over other owners' implementation.

**Owned paths:** src/cadence_detection/ingestion/, docs/design/, config/pipeline.json

**Deliverables:** Blueprint, architecture, ingestion/orchestrator, agreed schemas and integration notes.

**Definition of done and review:** Accepts all three supported formats, reports errors consistently, preserves document IDs and never bypasses the redaction gate. Person 4 reviews the trust boundary; Person 5 checks integration.

**Weekly plan and presentation:** Week 1: architecture and interface freeze. Week 2: integration pipeline. Week 3: resolve integration defects and present blueprint/architecture (about 2 minutes).

### Person 2: Document extraction and OCR owner

**Responsibilities**

- Implement PDF page-level extraction/OCR for all 35 scanned policy pages; retain page number, text blocks, coordinates, reading order and OCR confidence.
- Implement DOCX paragraph/table/header/footer/text-box extraction and PPTX slide/shape/table/group/notes extraction. Scan embedded images, and flag SVG/EMF conversion requirements.
- Preserve row/column relationships, headings and surrounding context; add crop/upscale passes for small screenshots and handle rotation or degraded scans.
- Create format fixtures and a page/slide extraction coverage report. Supply ordered blocks and an extraction-completeness signal to Persons 3 and 4.

**Owned paths:** src/cadence_detection/extraction/, tests/extraction/, data/extracted/

**Deliverables:** Three format adapters, OCR wrapper, common block output and extraction coverage report.

**Definition of done and review:** Every policy page and every PPTX slide is accounted for. Missing/unsupported content is flagged. Person 5 independently scores the agreed structure rubric; target >=80%. Person 1 reviews interfaces.

**Weekly plan and presentation:** Week 1: native adapters and PDF OCR spike. Week 2: embedded images, tables and context. Week 3: fix hard scans and present extraction (about 2 minutes).

### Person 3: PII detection and classification owner

**Responsibilities**

- Define recognizers for names/initials, emails, contextual phone formats, employee/director/vendor IDs, birth dates, addresses, national/tax IDs, passports and financial identifiers.
- Combine local NER with rules and context; support aliases, hyphenated/international names, repeated contacts, narrative prose and partially masked identifiers.
- Resolve overlapping spans deterministically; return type, block-local start/end, confidence, detector and location without putting raw values in routine logs.
- Build positive and negative fixtures, including policy sections 16.5/16.8 and PPTX RACI initials. Distinguish organizations, policy/control/question IDs, dates and money from personal identifiers.

**Owned paths:** src/cadence_detection/detection/, tests/detection/, config/pii_taxonomy.json

**Deliverables:** Detection/classification engine, recognizer catalogue, threshold decisions and entity tests.

**Definition of done and review:** All agreed entity categories have positive and negative cases; offsets remain valid after normalization; metrics are reported by category and source type. Person 2 checks source mapping; Person 5 independently evaluates.

**Weekly plan and presentation:** Week 1: taxonomy and baseline recognizers. Week 2: NER, aliases and narrative cases. Week 3: tune from held-out evaluation and present detection (about 2 minutes).

### Person 4: Redaction, privacy gate and demo application owner

**Responsibilities**

- Implement overlap-safe replacement in descending offset order; preserve surrounding context and use stable typed placeholders within a document.
- Build the safe-release gate: re-detect after masking, block incomplete extraction or uncertain content, and expose approved/quarantined/failed states clearly.
- Build a local upload/results UI or notebook showing format, extraction status, entity categories, source locations, sanitized text and downloadable safe output.
- Ensure raw text, temporary images and identity mappings stay local; define cleanup and log rules. Document text-output scope and any unimplemented original-file redaction limitations.

**Owned paths:** src/cadence_detection/redaction/, tests/redaction/, app/, output/redacted/

**Deliverables:** Text redactor, release gate, local demo interface, safe exports and demonstration runbook.

**Definition of done and review:** No raw spans survive the known-fixture checks; gate blocks OCR/adapter failures; repeated tokens are consistent and non-PII context remains readable. Person 3 reviews masking; Person 5 runs leakage/failure tests.

**Weekly plan and presentation:** Week 1: masking and mock UI using schema fixtures. Week 2: connect real pipeline and gate. Week 3: polish demo/rehearse and demonstrate all three files (about 4 minutes).

### Person 5: Independent evaluation, QA and submission owner

**Responsibilities**

- Create and adjudicate the ground truth with all four teammates contributing annotations. Label every occurrence, source location, entity type and image-only status; reserve a held-out subset before tuning.
- Measure precision, recall, F1, false positives/negatives, per-format and per-category performance, OCR/image-only recall and remaining PII after redaction.
- Define the 80% structure rubric before implementation; score headings, reading order, paragraph boundaries, table relationships and neighboring context. Keep structure score separate from text/OCR accuracy.
- Run full pipeline, corrupt/unsupported/empty-file and quarantine tests. Assemble final PPTX from each owner's content, evidence and diagrams; coordinate a 10-15 minute rehearsal and submission checklist.

**Owned paths:** src/cadence_detection/evaluation/, tests/integration/, data/annotations/, output/reports/, presentation/

**Deliverables:** Annotation guide and gold set, evaluation script/report, defect tracker, final stakeholder PPTX and demo fallback material.

**Definition of done and review:** Every mandatory outcome maps to evidence. Reports give denominators, unresolved cases and measured results rather than claiming universal 100% detection. Person 1 reviews deck completeness; Person 2 checks structure scoring.

**Weekly plan and presentation:** Week 1: annotation guide, rubric and test plan. Week 2: baseline report and deck outline. Week 3: final regression, metrics, deck assembly and reliability/results presentation (about 2 minutes).

## Handoffs and three-week coordination
- Days 1-2: Person 1 agrees schemas; Person 5 agrees labels, split and metric definitions. Every person reviews the source inventory.
- Week 1 checkpoint: Person 2 supplies extraction fixtures; Person 3 supplies detections on fixtures; Person 4 supplies a masking/UI skeleton; Person 5 supplies an annotation baseline. Architecture and interfaces are agreed together.
- Week 2 checkpoint: Person 1 runs all three files through the integrated pipeline. Person 5 publishes baseline results and defects; Persons 2-4 fix their components. All owners contribute slide drafts.
- Week 3 checkpoint: freeze features early, rerun evaluation, resolve blockers, assemble PPTX and rehearse. Person 4 prepares the local live demo; Person 5 prepares screenshots/recorded fallback. Confirm the actual submission date separately.
- Dependency chain: ingestion -> extraction -> detection -> redaction/gate -> evaluation/demo. Use schema fixtures to let Persons 3-5 work before OCR is complete. Person 5's gold set must not be replaced by Person 3's own predictions.

## Evaluation and completion criteria
- Entity precision = TP/(TP+FP), recall = TP/(TP+FN), F1 = 2PR/(P+R). Use exact occurrence/type matching as primary and separately report partial matches and OCR errors. Count repeated mentions separately.
- Proposed structure score: 20% headings, 20% reading order, 20% paragraph/block boundaries, 20% table row/column relationships, 20% neighboring context. Human reviewers use the same annotated units; mark non-applicable dimensions and renormalize. Target >=80% per document, reported alongside OCR/text accuracy. This rubric is a team proposal because the screenshots do not define a formula.
- Report native-text versus image-only recall on additional fixtures; all supplied policy pages are scans, so native PDF comparison needs another fixture or controlled transcription baseline.
- Ground truth includes p22 narrative, p23 correspondence, p18 attestation, p19 user screenshot, p20 audit log, p25 roster, p28 badge and PPTX abbreviated names. Use synthetic negative cases for PAN/PESEL/TIN/card patterns, policy IDs and generic dates.
- Re-scan sanitized text; independently check known gold spans and a sample manually. A zero residual count on this dataset is evidence for that test only, not a guarantee for future documents.
- Preserve source locations for each finding and for each failure. No silent dropped pages/images. Quarantine incomplete work instead of forwarding it to an LLM.

## Repository layout
```text
Cadence_Detection/
  README.md
  config/                         draft pipeline and entity taxonomy
  data/source/                    supplied evidence copies (local)
  data/problem_statement/         supplied screenshots (local)
  data/extracted/                 reference text; future OCR blocks (local)
  data/annotations/               annotation schema; future gold set (local)
  data/synthetic/                 future additional test inputs
  src/cadence_detection/
    ingestion/ extraction/ detection/ redaction/ evaluation/
  app/                            future local demo
  tests/                          extraction, detection, redaction, integration
  docs/design/                    proposed diagrams and interface contract
  docs/review/                    requirements and source inventory
  docs/team/                      detailed work allocation
  presentation/                   deck outline; final PPTX to be created
  output/pdf/                     summary PDF
  output/reports/                  future evaluation results
  output/redacted/                 future safe exports
  scripts/                        reproducible planning/PDF generation
  notebooks/                      optional experiments
```

## Working rules and first actions
1. Add actual names next to Person 1-5; keep ownership and review responsibilities explicit.
2. Read docs/review/source_review.md and docs/design/interfaces.md before coding.
3. Person 1 finalizes interfaces with everyone; Persons 2-4 start with their fixtures; Person 5 establishes independent labels.
4. Choose and pin runtime/OCR/model dependencies after the local prototype is agreed. requirements-planned.txt is a shortlist, not a working installation lockfile.
5. Never treat the reference text dump as OCR output or validated ground truth. Run the detector only after it is implemented; there is no runnable detection command yet.
6. Each owner updates progress, evidence, open defects and slide material. Do not commit raw source files, extracted PII, identity mappings or local caches by default.

## Deliverable status
- Completed: source review, source manifest/copies, scaffold, draft contracts/design, work allocation, summary PDF and deck outline.
- Pending: extraction/OCR, PII recognition, redaction, demo, annotations, measured results, final PPTX and stakeholder rehearsal.
