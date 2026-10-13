# Draft component contract

Agree before implementation. All statuses below are proposals.

- Document: document_id, filename, detected_format, sha256, processing_status, errors.
- Block: document_id, block_id, page_or_slide (1-based), source_kind, reading_order, text, table_row/table_column (optional), bbox (optional), OCR_confidence (optional), extraction_complete.
- Bounding boxes: specify coordinate system and units per adapter; retain conversion to original image/slide/page. Never mix normalized and pixel coordinates silently.
- Finding: document_id, block_id, entity_type, start, end, confidence, detector, source_location. Python Unicode code-point offsets; start inclusive/end exclusive into the exact normalized block text. Raw values stay in memory/local restricted evidence only.
- Redaction: resolve overlaps, replace from right to left, generate stable document-local typed tokens and preserve block structure. Mapping storage requires explicit local controls.
- Result: approved, quarantined or failed; sanitized_blocks, category_counts, extraction_coverage, remaining_candidate_count, warnings. Approval requires completed extraction and a passed verification gate; gate criteria to be agreed.
- Implement adapters independently of UI. Person 1 owns contract changes with all consumers reviewing; fixtures should cover errors and success.
