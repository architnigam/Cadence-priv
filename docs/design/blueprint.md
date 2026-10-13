# Proposed functional blueprint

```mermaid
flowchart LR
 U[Local upload] --> V[Validate and route]
 V --> E[Native extraction and local OCR]
 E --> B[Ordered blocks and source locations]
 B --> D[Local rules and NER]
 D --> R[Classify and redact]
 R --> G[Rescan and completeness gate]
 G -->|pass| S[Sanitized text and report]
 G -->|incomplete or uncertain| Q[Quarantine and review]
 S --> L[Downstream LLM boundary]
```

Raw evidence remains inside the local processing boundary. Future scale: job queue, bounded OCR workers and document-level isolation. Final technical architecture must specify actual selected libraries, storage, deployment and failure recovery; these are not implemented.
