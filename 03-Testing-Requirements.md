# Exam-Eval — Testing Requirements

**Document type:** Test requirements & quality plan  
**Version:** 1.0  
**Date:** 13 October 2026  

*Related: [01-Product-Features.md](./01-Product-Features.md) · [02-Functional-and-NonFunctional-Requirements.md](./02-Functional-and-NonFunctional-Requirements.md)*

---

## 1. Purpose and scope

This document defines **what must be tested**, **test types**, **entry/exit criteria**, and **traceability** from functional requirements (FR-*) to test cases. It does not list every test script; those live in the test repository per release.

**In scope:** API, web UI, teacher mobile, grading worker, analytics, campus isolation, install smoke tests.  
**Out of scope:** Penetration testing by external firm (separate engagement); load testing at national scale (phase 4).

---

## 2. Test principles

1. **No real student PII** in automated CI — use synthetic rosters and fixture scans only.
2. **Campus isolation** is a first-class test theme (security regression).
3. **Grading golden files** for MCQ; handwritten tests mock Bedrock/Textract in CI.
4. **Requirements traceability:** every Must FR has at least one automated or manual test case ID.
5. **Install smoke** runs against a disposable AWS account or LocalStack subset before release tags.

---

## 3. Test levels

| Level | Owner | When |
|-------|--------|------|
| Unit | Engineering | Every commit |
| Integration (API + DB) | Engineering | Every PR |
| Worker pipeline | Engineering | Every PR (mocked AI) |
| E2E (web) | Engineering | Nightly + release |
| Mobile E2E | Engineering | Release candidate |
| Manual exploratory | QA / product | Per sprint |
| Install smoke | DevOps | Per release |
| UAT | School pilot | Pre-GA |

---

## 4. Test requirements by area

### 4.1 Authentication and authorization (FR-AUTH-*, FR-ORG-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-AUTH-01 | FR-AUTH-03 | Reject API calls without token | Integration |
| T-AUTH-02 | FR-AUTH-03 | Reject expired token | Integration |
| T-AUTH-03 | FR-ORG-02 | User A cannot read campus B students | Integration + RLS |
| T-AUTH-04 | FR-ORG-03 | viewer cannot PATCH grading results | Integration |
| T-AUTH-05 | FR-ORG-04 | Override mark creates audit_log row | Integration |
| T-AUTH-06 | FR-AUTH-01 | Mobile login completes PKCE flow | E2E mobile |

### 4.2 Student database (FR-STU-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-STU-01 | FR-STU-02 | CSV import succeeds for valid file | Integration |
| T-STU-02 | FR-STU-02 | Duplicate roll_no in class reports error | Integration |
| T-STU-03 | FR-STU-03 | Bulk match fails gracefully if roster empty | Integration |
| T-STU-04 | FR-STU-04 | Profile shows exam totals after finalize | E2E |
| T-STU-05 | FR-STU-05 | Profile shows topic weak flags after insights | E2E |

### 4.3 Exams and topics (FR-EX-*, FR-TOP-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-EX-01 | FR-EX-05 | Cannot finalize from draft without publish | Integration |
| T-EX-02 | FR-TOP-04 | Strict mode blocks publish without topic on Q | Integration |
| T-EX-03 | FR-EX-03 | CSV answer key import round-trips | Unit + integration |
| T-EX-04 | FR-EX-04 | Template JSON stored and loaded for worker | Integration |

### 4.4 Upload — bulk and mobile (FR-SUB-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-SUB-01 | FR-SUB-01 | Bulk ZIP upload presign + complete enqueues jobs | Integration |
| T-SUB-02 | FR-SUB-02 | Per-student upload sets student_id immediately | Integration |
| T-SUB-03 | FR-SUB-03 | Mobile camera flow uploads to S3 (mock or device farm) | E2E mobile |
| T-SUB-04 | FR-SUB-04 | Roll OCR fixture maps to correct student | Worker unit |
| T-SUB-05 | FR-SUB-05 | Manual match UI resolves unmatched | E2E |
| T-SUB-06 | FR-SUB-06 | Second upload replaces draft submission | Integration |
| T-SUB-07 | FR-SUB-07 | Objects stored with encryption headers | Integration (S3) |

### 4.5 Grading (FR-GRD-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-GRD-01 | FR-GRD-01 | MCQ golden sheet scores 100% expected | Worker unit |
| T-GRD-02 | FR-GRD-01 | Wrong MCQ option scores zero | Worker unit |
| T-GRD-03 | FR-GRD-02 | Handwritten path with mocked Bedrock returns schema | Worker unit |
| T-GRD-04 | FR-GRD-04 | Low confidence → needs_review status | Worker integration |
| T-GRD-05 | FR-GRD-05 | Teacher override updates final_score | Integration |
| T-GRD-06 | FR-GRD-06 | Finalize triggers insights job | Integration |
| T-GRD-07 | FR-GRD-03 | Job status visible in UI | E2E |
| T-GRD-08 | FR-GRD-07 | Hindi exam uses language flag in worker config | Worker unit |

### 4.6 Insights (FR-INS-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-INS-01 | FR-INS-01 | Topic % computed correctly for fixture exam | Unit |
| T-INS-02 | FR-INS-02 | Class weak count matches fixture roster | Unit |
| T-INS-03 | FR-INS-03 | Threshold change recalculates weak flags | Integration |
| T-INS-04 | FR-INS-04 | Term rollup aggregates two finalized exams | Integration |
| T-INS-05 | FR-INS-05 | Class insights sort by weak % descending | E2E |

### 4.7 Reports (FR-RPT-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-RPT-01 | FR-RPT-02 | CSV export columns match schema | Integration |
| T-RPT-02 | FR-RPT-03 | viewer receives 403 on finalize | Integration |

### 4.8 Non-functional

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-NFR-01 | NFR-SEC-03 | RLS policy blocks cross-campus SELECT | Integration |
| T-NFR-02 | NFR-SEC-04 | Log sink contains no answer text substring | Automated log scan |
| T-NFR-03 | NFR-PERF-01 | k6 smoke: 50 RPS read API P95 &lt; 500 ms (staging) | Load (release) |
| T-NFR-04 | NFR-AVAIL-02 | DLQ receives poison message; alarm fires | Manual / staging |
| T-NFR-05 | NFR-UX-01 | Playwright smoke on Chromium | E2E |

### 4.9 Deployment (FR-DEP-*)

| Test ID | Requirement | Description | Type |
|---------|-------------|-------------|------|
| T-DEP-01 | FR-DEP-01 | `exam-eval doctor` passes on configured account | Install smoke |
| T-DEP-02 | FR-DEP-01 | Install completes; `/health` returns 200 | Install smoke |
| T-DEP-03 | FR-DEP-02 | CloudWatch alarm resources exist post-install | Install smoke |
| T-DEP-04 | FR-DEP-03 | Upgrade bumps image tag; migration succeeds | Install smoke |

---

## 5. Test data requirements

| Asset | Description | Storage |
|-------|-------------|---------|
| Roster CSV fixtures | 30 students, 2 sections, duplicate edge case | `tests/fixtures/roster/` |
| MCQ scan images | 5 sheets, known roll numbers | `tests/fixtures/scans/mcq/` |
| Handwritten fixtures | 2 sheets, mocked OCR text | `tests/fixtures/scans/hw/` |
| Answer key CSV | 20 questions, all types, topic codes | `tests/fixtures/keys/` |
| Template JSON | Roll region + 5 question boxes | `tests/fixtures/templates/` |
| Expected insights JSON | Golden output for T-INS-01/02 | `tests/fixtures/insights/` |

---

## 6. Environments

| Environment | Purpose | Data |
|-------------|---------|------|
| Local (Docker Compose) | Dev unit/integration | Synthetic |
| CI | PR gates | Synthetic; mocked AWS |
| Staging (school-like AWS) | E2E, load, install rehearsal | Synthetic only |
| Pilot | UAT with one school | Real (school-controlled) |

---

## 7. Entry and exit criteria

### 7.1 Release candidate entry

- All Must FRs have passing tests T-* mapped in traceability matrix.
- No open Critical/High defects.
- Install smoke T-DEP-01..03 passed on staging.

### 7.2 General availability exit

- Pilot UAT sign-off (minimum one campus, one full exam cycle).
- T-DEP-04 upgrade test passed.
- Mobile T-AUTH-06, T-SUB-03 passed on RC build.
- Security: T-NFR-01, T-NFR-02 passed.

---

## 8. Traceability matrix (Must FR → tests)

| FR IDs | Test IDs |
|--------|----------|
| FR-AUTH-01..03 | T-AUTH-01, 02, 06 |
| FR-ORG-02..04 | T-AUTH-03, 04, 05 |
| FR-STU-01..05 | T-STU-01..05 |
| FR-SUB-01..07 | T-SUB-01..07 |
| FR-GRD-01..07 | T-GRD-01..08 |
| FR-INS-01..02 | T-INS-01, 02 |
| FR-EX-05, FR-EX-06 | T-EX-01, T-GRD-06 |
| FR-DEP-01..03 | T-DEP-01..03 |

*(Should/Could FRs: add tests in backlog per sprint.)*

---

## 9. Defect severity

| Severity | Definition |
|----------|------------|
| Critical | Data leak across campuses; wrong marks at scale; install broken |
| High | Cannot upload, grade, or finalize; mobile login fails |
| Medium | Insights wrong but marks correct; UI workaround exists |
| Low | Cosmetic, non-blocking |

---

## 10. Responsibilities

| Role | Responsibility |
|------|----------------|
| Engineering | Unit, integration, worker tests; CI maintenance |
| QA | E2E scenarios, UAT coordination, exploratory |
| DevOps | Install smoke, staging environment |
| Product | UAT acceptance against Doc 01 features |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-13 | Initial testing requirements |
