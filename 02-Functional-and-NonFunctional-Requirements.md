# Exam-Eval — Functional & Non-Functional Requirements

**Document type:** Requirements specification  
**Version:** 1.0  
**Date:** 13 October 2026  

*Related: [01-Product-Features.md](./01-Product-Features.md) · [03-Testing-Requirements.md](./03-Testing-Requirements.md)*

---

## 1. Introduction

### 1.1 Purpose

This document specifies **functional requirements (FR)** and **non-functional requirements (NFR)** for Exam-Eval. Each FR is testable and mapped to product features in document 01.

### 1.2 Definitions

| Term | Definition |
|------|------------|
| Campus | A branch of the school group within one AWS installation |
| Roster | Imported list of students for a class/section |
| Finalize | Irreversible (or policy-gated) lock of exam marks for reporting |
| Weak (topic) | Student topic score below campus threshold or bottom quartile rule |

---

## 2. Functional requirements

### 2.1 Organization, campus, and security

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-ORG-01 | The system SHALL support multiple campuses per installation. | Must |
| FR-ORG-02 | The system SHALL enforce that a user only accesses data for campuses they are assigned. | Must |
| FR-ORG-03 | The system SHALL support roles: school_admin, teacher, viewer. | Must |
| FR-ORG-04 | The system SHALL record an audit log entry for mark overrides and answer key uploads. | Must |
| FR-ORG-05 | school_admin SHALL manage users and campus assignments. | Must |

### 2.2 Authentication

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-AUTH-01 | Staff SHALL authenticate before using web or mobile clients. | Must |
| FR-AUTH-02 | The system SHALL integrate with Cognito in the school AWS account. | Must |
| FR-AUTH-03 | API requests SHALL reject missing or invalid tokens. | Must |

### 2.3 Student database

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-STU-01 | The system SHALL store student roll_no, name, class, section, campus_id. | Must |
| FR-STU-02 | The system SHALL allow CSV import of students with a validation report. | Must |
| FR-STU-03 | The system SHALL require a roster for the exam’s class/section before matching submissions. | Must |
| FR-STU-04 | The system SHALL expose a student profile with marks per exam and subject. | Must |
| FR-STU-05 | The system SHALL expose topic-level strength/weakness on the student profile after insights run. | Must |
| FR-STU-06 | Teachers SHALL add or edit students when permitted by policy. | Should |

### 2.4 Subjects and topics

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-TOP-01 | The system SHALL maintain subjects per campus. | Must |
| FR-TOP-02 | The system SHALL maintain a hierarchical topic list per subject. | Must |
| FR-TOP-03 | The system SHALL import topics from CSV. | Should |
| FR-TOP-04 | Each exam question SHALL reference at least one topic before publish (configurable strict mode). | Must |

### 2.5 Textbooks

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-BOOK-01 | Teachers SHALL upload PDF textbooks per subject. | Should |
| FR-BOOK-02 | The system SHALL process textbooks asynchronously and report status. | Should |
| FR-BOOK-03 | The system SHALL store embeddings for optional retrieval during grading. | Could |

### 2.6 Exams and answer keys

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-EX-01 | Teachers SHALL create exams with subject, class, section(s), and language. | Must |
| FR-EX-02 | Teachers SHALL define questions with type and max_marks. | Must |
| FR-EX-03 | Teachers SHALL import or enter answer keys and rubrics. | Must |
| FR-EX-04 | Teachers SHALL define a sheet template including roll-number region. | Must |
| FR-EX-05 | The system SHALL transition exams through draft → published → finalized states. | Must |
| FR-EX-06 | The system SHALL prevent finalize if any submission is unmatched (configurable). | Must |

### 2.7 Submissions and upload

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-SUB-01 | Teachers SHALL bulk-upload scans for an exam via web (ZIP/PDF/images). | Must |
| FR-SUB-02 | Teachers SHALL upload scans for a single student via web or mobile. | Must |
| FR-SUB-03 | Mobile upload SHALL support camera capture and gallery selection. | Must |
| FR-SUB-04 | The system SHALL match bulk uploads to students via roll-number OCR when possible. | Must |
| FR-SUB-05 | The system SHALL provide manual match UI for failed or ambiguous matches. | Must |
| FR-SUB-06 | The system SHALL allow at most one active submission per student per exam (re-upload policy defined). | Must |
| FR-SUB-07 | Uploads SHALL use encrypted object storage in the school account. | Must |

### 2.8 Grading

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-GRD-01 | The system SHALL score MCQ items deterministically from the answer key. | Must |
| FR-GRD-02 | The system SHALL score handwritten items using OCR and AI against rubrics. | Must |
| FR-GRD-03 | The system SHALL run grading asynchronously with visible job status. | Must |
| FR-GRD-04 | The system SHALL flag items below confidence threshold for teacher review. | Must |
| FR-GRD-05 | Teachers SHALL override AI scores with comment. | Must |
| FR-GRD-06 | Teachers SHALL finalize exams to publish marks and trigger analytics. | Must |
| FR-GRD-07 | Grading for language papers SHALL use the exam’s configured language. | Must |

### 2.9 Marks and exports

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-RPT-01 | The system SHALL display marks per student and per question after grading. | Must |
| FR-RPT-02 | The system SHALL export exam marks to CSV. | Must |
| FR-RPT-03 | viewer role SHALL read marks and insights without edit rights. | Must |

### 2.10 Learning insights

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-INS-01 | On finalize, the system SHALL compute per-student topic score percentages. | Must |
| FR-INS-02 | The system SHALL show per-class count of students weak per topic. | Must |
| FR-INS-03 | Campus admins SHALL configure weak-topic threshold. | Should |
| FR-INS-04 | The system SHALL support term rollups for one subject and class over a date range. | Should |
| FR-INS-05 | Class insight views SHALL be sortable by re-teach priority. | Should |

### 2.11 Installation (school IT)

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-DEP-01 | The product SHALL be installable in a school AWS account without Kubernetes. | Must |
| FR-DEP-02 | Install SHALL provision monitoring alarms and database backup. | Must |
| FR-DEP-03 | Upgrade SHALL be supported via documented CLI or stack update. | Must |

---

## 3. Non-functional requirements

### 3.1 Performance and scalability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-PERF-01 | API read latency (excluding uploads) | P95 &lt; 500 ms under normal load |
| NFR-PERF-02 | Bulk upload initiation | Presign response &lt; 2 s |
| NFR-PERF-03 | Students per campus | ≥ 2,000 |
| NFR-PERF-04 | Sheets per exam batch | ≥ 500 |
| NFR-PERF-05 | Grading throughput | Scale workers with queue depth |

### 3.2 Availability and reliability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-AVAIL-01 | Production stack (customer-operated) | Target 99.5% during school hours |
| NFR-AVAIL-02 | Failed grading jobs | Retry with DLQ and alerting |
| NFR-AVAIL-03 | Database | Multi-AZ in production deployments |

### 3.3 Security and privacy

| ID | Requirement |
|----|-------------|
| NFR-SEC-01 | All data at rest encrypted with KMS. |
| NFR-SEC-02 | All traffic TLS 1.2+. |
| NFR-SEC-03 | Campus isolation enforced in API and database RLS. |
| NFR-SEC-04 | Answer text SHALL NOT appear in application logs. |
| NFR-SEC-05 | Presigned URLs SHALL be time-limited and campus-scoped keys. |
| NFR-SEC-06 | Student production data SHALL NOT be used in vendor CI. |

### 3.4 Compliance and data residency

| ID | Requirement |
|----|-------------|
| NFR-COMP-01 | Production data SHALL reside in the school AWS account (default region ap-south-1). |
| NFR-COMP-02 | Configurable retention and deletion of scan images. |
| NFR-COMP-03 | Audit log retained per campus policy (minimum 1 year recommended). |

### 3.5 Usability

| ID | Requirement |
|----|-------------|
| NFR-UX-01 | Web UI SHALL be usable on desktop browsers (latest Chrome, Edge, Safari). |
| NFR-UX-02 | Teacher mobile SHALL support current iOS and Android versions supported by Expo. |
| NFR-UX-03 | Bulk upload flow SHALL show matched vs unmatched counts before grading completes. |
| NFR-UX-04 | Staff UI language v1: English (exam content may be regional). |

### 3.6 Maintainability and operability

| ID | Requirement |
|----|-------------|
| NFR-OPS-01 | Structured JSON logs with correlation IDs. |
| NFR-OPS-02 | CloudWatch dashboards for API errors, queue depth, grading failures. |
| NFR-OPS-03 | Database schema changes via versioned migrations. |
| NFR-OPS-04 | API versioned under `/api/v1`. |

### 3.7 Compatibility

| ID | Requirement |
|----|-------------|
| NFR-COMPAT-01 | Accept scan formats: PDF, JPEG, PNG in bulk and mobile upload. |
| NFR-COMPAT-02 | Answer key CSV template documented and stable across minor releases. |

---

## 4. Requirements traceability (summary)

| Feature area (Doc 01) | FR range |
|------------------------|----------|
| Student database | FR-STU-* |
| Upload bulk / mobile | FR-SUB-* |
| Grading | FR-GRD-* |
| Insights | FR-INS-* |
| Deployment | FR-DEP-* |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-13 | Initial FR/NFR set |
