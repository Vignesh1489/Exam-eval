# Exam-Eval — Product Development Plan

**Product name:** Exam-Eval  
**Document version:** 1.0  
**Date:** 13 October 2026  
**Repository:** `/Users/vdhana/local-workspace/exam-eval`  
**Primary region:** AWS ap-south-1 (India)

**Related:** [Exam-Eval-HLD-and-LLD.md](./Exam-Eval-HLD-and-LLD.md) (high-level + per-component low-level design) — [PDF](./Exam-Eval-HLD-and-LLD.pdf)

---

## Table of contents

1. [Executive summary](#1-executive-summary)
2. [Vision and goals](#2-vision-and-goals)
3. [Target users and personas](#3-target-users-and-personas)
4. [Deployment and commercial model](#4-deployment-and-commercial-model)
5. [Competitive positioning](#5-competitive-positioning)
6. [Functional requirements](#6-functional-requirements)
7. [Learning insights and analytics](#7-learning-insights-and-analytics)
8. [Non-functional requirements](#8-non-functional-requirements)
9. [System architecture](#9-system-architecture)
10. [Data model](#10-data-model)
11. [API specification (overview)](#11-api-specification-overview)
12. [User interface map](#12-user-interface-map)
13. [AI grading pipeline](#13-ai-grading-pipeline)
14. [Security, privacy, and compliance](#14-security-privacy-and-compliance)
15. [Deployment and operations](#15-deployment-and-operations)
16. [Implementation roadmap](#16-implementation-roadmap)
17. [Testing and quality](#17-testing-and-quality)
18. [Risks and mitigations](#18-risks-and-mitigations)
19. [Success metrics](#19-success-metrics)
20. [Open decisions and defaults](#20-open-decisions-and-defaults)
21. [Appendix A — Answer key CSV template](#appendix-a--answer-key-csv-template)
22. [Appendix B — Repository structure](#appendix-b--repository-structure)

---

## 1. Executive summary

Exam-Eval is a **school-owned AWS** platform for **mixed-format exam evaluation** (MCQ/OMR + handwritten answers) and **topic-level learning insights**. Schools install the product in **their own AWS account** so student scans, marks, and AI processing (Amazon Textract and Amazon Bedrock) **never leave their environment**.

**Core value:**

- **Automate grading** with AI plus teacher review for low-confidence items.
- **Store marks** per student, subject, and exam with audit trails.
- **Ingest textbooks and answer keys** to improve grading and teaching.
- **Show class and student insights** — which topics are weak across the class and per child — so teachers know what to re-teach.

**Install model:** `exam-eval` CLI or AWS CloudFormation **Launch Stack** (no Kubernetes, Helm, or GitOps for school IT).

---

## 2. Vision and goals

### Vision

Every school group can run accurate, transparent exam evaluation and data-driven remediation **without sending student data to a vendor’s shared cloud**.

### Product goals (v1)

| Goal | Description |
|------|-------------|
| G1 | Grade mixed exams from scanned answer sheets |
| G2 | Match sheets to students via **roll-number OCR** (manual match fallback) |
| G3 | Support **English** and **regional language** exam papers |
| G4 | Multi-**campus** isolation within one customer AWS account |
| G5 | **Topic-tagged** questions → strength/weakness insights at student and class level |
| G6 | One-command (or Launch Stack) **install** with observability and backups included |
| G7 | **Teacher mobile app** for per-student scan/upload in class; web for bulk, setup, and analytics |
| G8 | **Student database** with class, section, roll number, and **marks history** (per exam/subject/topic) |

### Out of scope (v1)

- **Student** or parent mobile app (no student logins)
- Central vendor-hosted production datastore
- Student-facing AI tutor chat
- Online proctoring
- Payment/billing engine (license file optional in Phase 3)

---

## 3. Target users and personas

| Persona | Needs |
|---------|--------|
| **School admin** | Campuses, users, policies, retention, exports |
| **Teacher** | Web: exams, bulk upload, review, insights; **Mobile:** roster → tap student → camera upload sheet |
| **Viewer** (principal/coordinator) | Read-only dashboards and reports |
| **School IT** | Install, upgrade, DNS, Bedrock enablement, backups |
| **Vendor support** (optional) | Time-bound IAM role; no standing PII access |

---

## 4. Deployment and commercial model

### Primary: self-host in school AWS account

| Aspect | Choice |
|--------|--------|
| Data residency | School AWS account, ap-south-1 |
| Isolation between customers | Separate AWS accounts |
| Isolation between campuses | PostgreSQL RLS + `campus_id` |
| Auth | Amazon Cognito in school account |
| Revenue (suggested) | Per-campus license + implementation + annual support; school pays AWS usage |

### Secondary (optional later)

- Short **vendor-cloud pilot** with strict data deletion (not production default).
- **AWS Marketplace** → CloudFormation install.
- **Dedicated account** operated by vendor for schools with zero IT (same isolation story).

### Confirmed product decisions

| # | Decision |
|---|----------|
| 1 | Repo: `exam-eval` under `local-workspace` |
| 2 | Sheet identity: roll-number OCR only (v1); manual match UI |
| 3 | Languages: English + per-exam language for language papers |
| 4 | License: none in Phase 1–2; offline signed license in Phase 3 (no student PII) |
| 5 | Images: vendor ECR + cross-account mirror to school ECR |

---

## 5. Competitive positioning

**Positioning:** *DeepGrading/EvalAI-class AI grading, but data and AI never leave the school’s AWS account.*

| Competitor type | Examples | Gap Exam-Eval fills |
|-----------------|----------|---------------------|
| India AI grading SaaS | DeepGrading, EvalAI, PaperArch | School-owned AWS + install kit |
| University grading | Gradescope (Turnitin) | K–12 India, full auto-score + insights, BYOC |
| OMR-only | TestPlus, ScoreExam, Alpha OMR | Handwritten descriptive + topic insights |
| Self-hosted exams | QST, LAN exam tools | Paper scan + AI rubric pipeline |

**Differentiators:** data sovereignty, easy install, mixed exams, regional languages, textbook RAG, multi-campus, **re-teach analytics**.

---

## 6. Functional requirements

### 6.1 Organization and access

- FR-ORG-1: Single organization per installation; multiple **campuses**.
- FR-ORG-2: Roles: `school_admin`, `teacher`, `viewer`.
- FR-ORG-3: Campus switcher for admins; teachers scoped to assigned campus(es)/classes.
- FR-ORG-4: Audit log for mark overrides and key uploads.

### 6.2 Students and subjects (roster required)

**Student database is mandatory** before answer sheets can be matched and marks attributed. The system does not invent students from scans alone (except a controlled **add-from-unmatched** flow after bulk upload).

- FR-STU-1: CRUD students; fields: roll_no, name, class, section, campus_id.
- FR-STU-2: CSV import with duplicate detection report (primary onboarding path).
- FR-STU-3: Exam wizard selects **class + section(s)**; only students in that roster are eligible for match.
- FR-SUB-1: Subjects per campus (name, code).

**Typical order:** import roster → create exam for that class → upload sheets → match by roll number → grade.

### 6.2b Answer sheet upload modes

| Mode | Priority | Description |
|------|----------|-------------|
| **Bulk (class batch)** | **Primary (v1)** | Teacher uploads one **ZIP**, multi-page **PDF**, or many images in one action for the whole class (e.g. after scanning the stack at the office). System splits pages, OCRs roll numbers, links each sheet set to a roster student. |
| **Per-student** | **Secondary (v1)** | Teacher uploads one PDF/images for **one student** (late submission, absentee makeup, re-scan). Student chosen from dropdown **or** roll number on sheet; no need to re-upload the whole class. |
| **Add missing student** | Exception | If bulk scan reveals unknown roll no: **manual match** to existing student or **add student to roster** then match (admin/teacher permission). |

**Bulk upload expectations (documented for schools):**

- Preferred: **one answer sheet per student** (all pages for that student contiguous), roll number in template header.
- Acceptable: **single PDF** with all students’ pages in scan order — worker splits on blank page or sheet boundary heuristics (configurable; template helps).
- ZIP layout: flat images `rollno_*.pdf` or `section_rollno.pdf` optional naming to aid match when OCR fails.

**Not in v1:** students uploading their own papers from home (no student portal).

### 6.2c Client applications (web + teacher mobile)

| Client | Users | Primary jobs |
|--------|-------|----------------|
| **Web app** (Next.js) | Admin, teacher | Roster import, exam wizard, **bulk** scan upload, review queue, marks tables, **class/student insights**, settings |
| **Teacher mobile app** (iOS/Android) | Teacher | **Per-student** capture: class/section list → select student → **camera** or gallery → upload; see upload status; optional quick view of total marks (not full analytics) |

**Strategy:** One **API** (FastAPI); web and mobile are separate clients. Mobile is **not** a full replacement at first — it optimizes **in-class per-student upload**. Long term, mobile can gain more web parity (review on tablet, etc.).

**Tech (planned):** React Native (Expo) + shared TypeScript API types with web; Cognito PKCE login on device; presigned S3 upload from phone.

**Phasing:** Web MVP in Phase 1–2; **teacher mobile beta in Phase 2** (after per-student API stable).

### 6.3 Syllabus and topics

- FR-TOP-1: Topic tree per subject (unit/chapter → topic).
- FR-TOP-2: CSV import of syllabus; optional CBSE-style templates.
- FR-TOP-3: Link topics to textbook chunks (page references).

### 6.4 Textbooks

- FR-BOOK-1: Upload PDF per subject; async ingest job.
- FR-BOOK-2: Chunk text; embed with Bedrock; store in pgvector.
- FR-BOOK-3: Status: processing / ready / failed.

### 6.5 Exams and answer keys

- FR-EX-1: Exam lifecycle: draft → published → grading → finalized → closed.
- FR-EX-2: Metadata: campus, subject, class, section(s), term, **language**, total marks.
- FR-EX-3: Questions: number, type (mcq | short | long | numeric), max_marks, **topic_code**.
- FR-EX-4: Answer key import via CSV (preferred) or PDF + AI extract + teacher confirm.
- FR-EX-5: Sheet **template JSON** (roll-number region, per-question crop boxes for OMR/handwritten).
- FR-EX-6: Block publish if questions lack topic tags (configurable strict mode).

### 6.6 Submissions and identity

- FR-SUB-1: **Bulk** upload PDF/images/ZIP for an exam (primary).
- FR-SUB-1b: **Per-student** upload linked to `student_id` or roll number (secondary).
- FR-SUB-2: Split pages; OCR roll number from template region (bulk path).
- FR-SUB-3: Match to **pre-imported roster** for exam class/section; ambiguous → manual match queue.
- FR-SUB-4: Block finalize while unmatched submissions exist.
- FR-SUB-5: One **submission** per student per exam (re-upload replaces or versions per policy; default: replace draft, block if finalized).

### 6.7 Grading

- FR-GRD-1: MCQ/OMR: deterministic scoring from template + Textract.
- FR-GRD-2: Handwritten: Textract + Bedrock JSON score (rubric, partial credit).
- FR-GRD-3: Optional RAG: retrieve textbook chunks per question.
- FR-GRD-4: Confidence threshold → **review queue**.
- FR-GRD-5: Teacher override with comment; recompute topic stats on save.
- FR-GRD-6: Finalize exam → publish marks; export CSV/PDF.

### 6.8 Learning insights

- FR-INS-1: Per-student topic score % per exam and per term.
- FR-INS-2: Class view: count and % of students **weak** per topic.
- FR-INS-3: Configurable weak threshold (default 40%) and optional bottom-quartile rule.
- FR-INS-4: Re-teach priority list; drill-down to student names (authorized teachers only).
- FR-INS-5: Printable class diagnostic summary.

### 6.9 Reports and exports

- FR-RPT-1: Marks matrix (student × subject/exam).
- FR-RPT-2: Excel/CSV export; PDF report cards (phase 2+).

---

## 7. Learning insights and analytics

### 7.1 Data dependency

Insights require **every question mapped to at least one topic** at exam setup.

### 7.2 Metrics

| Metric | Formula / rule |
|--------|----------------|
| Topic score % | Σ(marks obtained on Q for topic) / Σ(max marks on those Q) |
| Weak | Score % &lt; threshold OR bottom quartile in class (configurable) |
| Class weak count | # students weak on topic in section/class |
| Strength | Top topics by score % above median (configurable) |

### 7.3 Rollup scope

- **Exam-level:** computed on finalize (v1 minimum).
- **Term-level:** same subject + class + date range (Phase 2).

### 7.4 Optional AI (Phase 2+)

- Suggest topic tags from question paper PDF.
- Natural-language **class gap summary** (aggregated counts only; no student names in LLM prompt).

---

## 8. Non-functional requirements

| ID | Requirement |
|----|-------------|
| NFR-1 | Availability target 99.5% during school hours (customer-operated) |
| NFR-2 | P95 API &lt; 500 ms excluding uploads |
| NFR-3 | Grading async; progress visible in UI |
| NFR-4 | Encrypt all data at rest (KMS) and in transit (TLS 1.2+) |
| NFR-5 | Campus RLS enforced in DB and API |
| NFR-6 | No answer text in application logs |
| NFR-7 | Support ~2,000 students per campus; ~500 sheets per exam batch |
| NFR-8 | Retention policy configurable per campus (delete scans after N months) |

---

## 9. System architecture

### 9.1 Components (per school AWS account)

- **Next.js** web app (ECS Fargate)
- **FastAPI** API (ECS Fargate)
- **Grading worker** (ECS Fargate, scales on SQS depth)
- **RDS PostgreSQL 16** (+ pgvector)
- **S3** (scans, books, keys) — SSE-KMS
- **SQS** (+ DLQ)
- **Cognito** user pool
- **Textract**, **Bedrock** (same region)
- **ALB**, **ACM**, **Route 53** (customer domain)

### 9.2 Data flow

1. Upload → S3 (presigned URL).
2. API enqueues grading job → SQS.
3. Worker: S3 → Textract → route by question type → Bedrock (if needed) → RDS.
4. On finalize: analytics rollup job → insight tables.
5. No student payloads sent to vendor systems.

### 9.3 Technology stack

| Layer | Technology |
|-------|------------|
| Web | Next.js 15, TypeScript, Tailwind, shadcn/ui |
| Teacher mobile | React Native (Expo), TypeScript, shared API client package |
| Backend | FastAPI, Python 3.12, SQLAlchemy, Alembic |
| Worker | Same Python codebase as API |
| DB | PostgreSQL 16, RLS policies |
| IaC | Terraform root module (+ CloudFormation parity) |
| CLI | Python or Go: `exam-eval` install/upgrade/doctor |

---

## 10. Data model

### 10.1 Core tables (campus-scoped unless noted)

| Table | Purpose |
|-------|---------|
| `organizations` | Customer metadata |
| `campuses` | Branch/campus |
| `users` | Staff accounts; link to Cognito sub |
| `user_campus_roles` | campus_id, role |
| `students` | Roster: roll_no, name, class, section, campus_id, status (active/left), optional external_id |
| `student_enrollments` | Optional: student in class/section per academic year |
| `student_exam_results` | Per exam: total marks, max, percent, finalized_at (denormalized for fast student profile) |
| `student_subject_summaries` | Per term: subject-wise totals/averages (materialized on finalize) |
| `subjects` | Per campus |
| `topics` | Syllabus tree |
| `textbooks` | PDF metadata, S3 key |
| `textbook_chunks` | Text + embedding vector |
| `exams` | Exam metadata, language, status |
| `exam_questions` | Q types, marks, topic_id, template regions |
| `answer_keys` | Canonical answers, rubrics, MCQ key |
| `submissions` | Per student attempt, match status |
| `submission_pages` | S3 keys per page |
| `submission_answers` | OCR text, selected option |
| `grading_results` | auto_score, final_score, confidence, feedback |
| `exam_topic_stats` | Materialized per student per topic per exam |
| `class_topic_summary` | Weak counts, medians per topic |
| `audit_log` | Immutable events |
| `jobs` | Async job status |

### 10.2 RLS policy pattern

`campus_id = current_setting('app.current_campus_ids')` (session set post-auth for allowed campuses).

---

## 11. API specification (overview)

Base path: `/api/v1` — Bearer JWT from Cognito (or dev token).

| Area | Methods |
|------|---------|
| Auth | `GET /me`, `POST /auth/refresh` |
| Campuses | `GET`, `POST`, `PATCH` (admin) |
| Users | CRUD (admin) |
| Students | `GET`, `POST`, `POST /import` |
| Subjects | CRUD |
| Topics | CRUD, `POST /import` |
| Textbooks | `POST /upload`, `GET`, `GET /{id}/status` |
| Exams | CRUD, `POST /{id}/questions`, `POST /{id}/answer-key/import`, `POST /{id}/publish`, `POST /{id}/finalize` |
| Submissions | `POST /exams/{id}/upload`, `GET /{id}`, `POST /{id}/match` |
| Grading | `GET /exams/{id}/review-queue`, `PATCH /grading-results/{id}` |
| Insights | `GET /exams/{id}/insights/class`, `GET /exams/{id}/insights/students/{id}`, `GET /insights/term` |
| Reports | `GET /exams/{id}/export.csv`, `GET /reports/marks` |
| Jobs | `GET /jobs/{id}` |

---

## 12. User interface map

| Screen | Primary user |
|--------|----------------|
| Login / Cognito hosted UI | All |
| Dashboard (jobs, review count, recent exams) | Teacher |
| Campuses & users | Admin |
| Students (list, import, **profile: marks by subject/exam/topic**) | Teacher, admin |
| **Teacher mobile:** class roster → student → camera upload | Teacher |
| Subjects & syllabus topics | Teacher, admin |
| Textbooks (upload, status) | Teacher |
| Exam wizard (metadata → questions/key → template → publish) | Teacher |
| Upload scans (progress, errors, manual match) | Teacher |
| Review grading (crop + score + override) | Teacher |
| Marks table | Teacher, viewer |
| **Class insights** (weak topics heatmap) | Teacher, viewer |
| **Student insights** (profile) | Teacher |
| Re-teach planner | Teacher |
| Settings (thresholds, retention) | Admin |
| Post-install setup wizard | Admin, IT |

---

## 13. AI grading pipeline

### 13.1 Stages

1. **Ingest scan** — validate format, store S3.
2. **Identify student** — OCR roll no → roster match.
3. **Layout** — apply exam template; extract regions.
4. **MCQ path** — bubble detection / mark detection → compare to key.
5. **Handwritten path** — Textract text → Bedrock with rubric + optional RAG → JSON `{ score, confidence, rationale }`.
6. **Aggregate** — total marks; flag review if confidence low or unmatched.
7. **Finalize** — lock marks; run topic analytics.

### 13.2 Bedrock prompt constraints

- Structured JSON output only.
- Exam language matches `exams.language`.
- Minimize PII in prompt (roll number only where needed).

### 13.3 Human in the loop

Teachers must review queue before finalize (configurable: require zero pending review items).

---

## 14. Security, privacy, and compliance

- KMS encryption for S3 and RDS.
- Private subnets for RDS; ECS tasks without public IPs.
- IAM least privilege per service.
- CloudTrail in customer account.
- DPDP-aware practices: purpose limitation, retention, access control.
- Optional `VendorSupportRole` with external ID and time limit.

---

## 15. Deployment and operations

### 15.1 Path A — `exam-eval` CLI (primary)

```bash
exam-eval doctor
exam-eval install --region ap-south-1 --domain exams.school.edu --admin-email admin@school.edu
exam-eval upgrade --version 1.0.0
```

### 15.2 Path B — CloudFormation Launch Stack

Browser wizard; same resources as Path A.

### 15.3 Included operations assets

- CloudWatch dashboards and alarms (5xx, DLQ, RDS, queue age).
- AWS Backup for RDS.
- Container Insights.

### 15.4 School IT checklist

1. AWS account + installer IAM policy  
2. DNS CNAME to ALB  
3. Enable Bedrock models  
4. Run install (~30–45 min)  
5. Annual upgrades via CLI  

**Explicitly excluded from default:** Helm, GitOps, EKS.

---

## 16. Implementation roadmap

### Phase 1 — Foundation (8–10 weeks indicative)

| Deliverable | Details |
|-------------|---------|
| Monorepo scaffold | `apps/web`, `apps/api`, `workers/grader`, `docker-compose` |
| Auth + RLS | Cognito integration stub; campus scoping |
| Domain CRUD | Campuses, users, students, subjects, topics |
| Exams | Wizard, CSV key import with topic_code |
| Storage | S3 presigned uploads (local MinIO in dev) |
| MCQ grading | Deterministic path + marks UI |
| Basic topic rollup | Table only; simple report |

**Exit criteria:** End-to-end demo: import students → create exam → upload MCQ scans (dev fixtures) → see marks.

### Phase 2 — AI + insights + teacher mobile (10–14 weeks)

| Deliverable | Details |
|-------------|---------|
| SQS worker | Production grading pipeline |
| Textract + Bedrock | Handwritten + review UI |
| Roll-number OCR | Template + manual match |
| Textbook RAG | pgvector ingest |
| Insight dashboards | Class + student + term rollup |
| Multilingual | Per-exam language in prompts/OCR |
| Student profile API | Class, section, roll no, marks by exam/subject/topic |
| Teacher mobile (Expo) | Per-student camera/gallery upload, roster by class |

**Exit criteria:** Mixed exam graded with teacher review; class weak-topic dashboard; teacher uploads one student sheet from mobile.

### Phase 3 — School install kit (6–8 weeks)

| Deliverable | Details |
|-------------|---------|
| Terraform root module | Full AWS stack |
| `exam-eval` CLI | install / upgrade / doctor |
| CloudFormation | Launch Stack parity |
| ECR mirror | Automated in install |
| Docs | School IT one-pager, backup/restore |
| License | Optional offline signed file |

**Exit criteria:** Fresh AWS account → install → smoke test in &lt; 1 day.

### Phase 4 — Enterprise (ongoing)

- Cognito SAML / AD federation  
- Marketplace listing  
- Advanced observability toggle  
- Cost tagging dashboards  

---

## 17. Testing and quality

| Layer | Approach |
|-------|----------|
| Unit | Scoring, CSV parsers, topic rollup math |
| Integration | RLS: campus A cannot read campus B |
| Worker | Golden scan fixtures; mock Bedrock in CI |
| E2E | Playwright: login → exam → upload → marks |
| Security | OWASP API checks; dependency scanning |
| Load | 500-sheet batch job completion time |

No production student data in vendor CI.

---

## 18. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| School IT skill gap | CLI + Launch Stack + partner install |
| Bedrock not enabled | `doctor` command + wizard |
| Wrong student match | Manual match; block finalize |
| LLM inconsistency | Review queue; rubric JSON schema |
| OCR on regional scripts | Per-exam language; re-OCR option |
| Upgrade drift | Semver, compatibility matrix |
| Cost surprise | AWS cost guide per 1k pages |
| Missing topic tags | Block publish in strict mode |

---

## 19. Success metrics

| Metric | Target (pilot schools) |
|--------|------------------------|
| Grading time vs manual | ≥ 70% reduction |
| Teacher override rate | &lt; 15% of questions after tuning |
| Roll-number auto-match rate | ≥ 90% |
| Install time (IT) | &lt; 4 hours first time |
| Insight adoption | ≥ 80% teachers open class insights post-exam |
| NPS (teachers) | ≥ 40 after term |

---

## 20. Open decisions and defaults

| Topic | Default |
|-------|---------|
| License | Offline signed file in Phase 3 |
| Container registry | Vendor ECR mirror |
| Weak topic threshold | 40% |
| Strict topic tagging on publish | On for insights-dependent schools |

---

## Appendix A — Answer key CSV template

```csv
question_no,type,max_marks,topic_code,correct_option,correct_answer,rubric,keywords
1,mcq,1,T01-BIO-CELLS,B,,,
2,short,3,T02-BIO-MITO,,Mitochondria is the powerhouse of the cell,Full marks for nucleus+function mention,mitochondria|ATP
3,long,5,T03-BIO-PHOTOSYN,,Describe light-dependent reactions,See rubric JSON in system,
```

---

## Appendix B — Repository structure

```
exam-eval/
├── apps/
│   ├── web/                 # Next.js UI
│   ├── mobile/              # Expo teacher app
│   └── api/                 # FastAPI
├── packages/
│   └── api-client/          # Shared TS types + fetch for web + mobile
├── workers/
│   └── grader/              # SQS consumer (may share api package)
├── packages/
│   └── shared/              # Optional shared types
├── infra/
│   ├── terraform/
│   │   └── school-install/
│   └── cloudformation/
├── cli/
│   └── exam-eval/           # Install CLI
├── docs/
│   ├── Exam-Eval-Product-Development-Plan.md
│   └── Exam-Eval-Product-Development-Plan.pdf
├── docker-compose.yml
└── README.md
```

---

## Document control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-10-13 | Product plan | Initial consolidated PRD |

---

*End of document*
