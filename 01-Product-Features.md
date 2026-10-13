# Exam-Eval — Product Features

**Document type:** Product feature catalog  
**Version:** 1.0  
**Date:** 13 October 2026  
**Audience:** School leadership, teachers, product stakeholders  

*Related: [02-Functional-and-NonFunctional-Requirements.md](./02-Functional-and-NonFunctional-Requirements.md) · [03-Testing-Requirements.md](./03-Testing-Requirements.md)*

---

## 1. Product summary

Exam-Eval helps schools **grade mixed-format exams** (MCQ and handwritten answers) from scanned papers, **store marks** against a **student database**, and surface **topic-level insights** so teachers know what to re-teach. The system runs in the **school’s own AWS account**; student data does not sit on a vendor shared cloud.

**Clients:**

- **Web application** — setup, bulk upload, review, analytics, administration.
- **Teacher mobile application** — per-student photo upload in the classroom (teachers only; no student app).

---

## 2. Feature areas

### 2.1 Organization and campuses

- Support one school group per installation with **multiple campuses**.
- Campus switcher for administrators; teachers see only assigned campuses.
- Campus-level settings (e.g. weak-topic threshold, scan retention).

### 2.2 Users and access

- Staff sign-in with username/password (Amazon Cognito in the school account).
- Roles: **school admin**, **teacher**, **viewer** (read-only).
- Audit trail for sensitive actions (mark overrides, answer key changes).

### 2.3 Student database

- Maintain student records: **roll number**, name, **class**, **section**, campus.
- **CSV import** and in-app edit; duplicate detection on import.
- **Student profile** showing:
  - Identity and class/section
  - **Marks** per exam and subject
  - **Topic strengths and weaknesses** (after insights are computed)
- Roster is **required** before sheets can be matched to students.

### 2.4 Subjects and syllabus topics

- Define subjects per campus.
- Maintain a **topic tree** (unit/chapter → topic) per subject.
- Import syllabus via CSV or templates.
- Link topics to textbook content where books are ingested.

### 2.5 Textbooks (class books)

- Upload PDF textbooks per subject.
- Automatic text extraction and chunking for search during grading (optional per question).
- Ingestion status visible to teachers (processing / ready / failed).

### 2.6 Exams and answer keys

- Create exams: subject, class, section(s), term, **exam language** (English or regional for language papers).
- Define questions: number, type (MCQ, short answer, long answer, numeric), max marks.
- **Tag each question to a syllabus topic** (required for learning insights).
- Import **answer keys** via structured CSV or PDF with teacher confirmation.
- Define **answer sheet template** (roll-number region, question regions for OMR/handwriting).
- Exam lifecycle: draft → published → grading → finalized.

### 2.7 Answer sheet upload (teacher only)

**Bulk upload (web, primary)**

- Upload ZIP, multi-page PDF, or multiple images for a whole class in one session.
- System splits pages, reads **roll numbers** from the template, matches the roster.
- Manual match for unmatched or ambiguous sheets.

**Per-student upload (web and mobile)**

- Teacher selects exam → class/section → **student** → upload file or **capture with camera** (mobile).
- Intended for late submissions, absentees, or re-scans without re-uploading the full class.

### 2.8 AI-assisted grading

- **MCQ / OMR:** automatic scoring from scanned marks.
- **Handwritten answers:** OCR plus AI scoring against model answers and rubrics.
- Partial credit and short rationale for teachers.
- **Human review queue** for low-confidence results; teachers can override scores.
- **Finalize** exam to lock marks and trigger analytics.

### 2.9 Marks and reports

- Question-wise and total marks per student.
- Class marks table; filters by section and subject.
- Export to **CSV/Excel**; PDF summaries for staff meetings (where implemented).

### 2.10 Learning insights (topic analytics)

- **Per student:** topic score %; labeled strengths and weaknesses.
- **Per class:** how many (and what %) of students are weak on each topic.
- **Re-teach priority** list sorted by class need.
- Drill-down from topic to list of weak students.
- **Term view:** roll up multiple exams in the same subject and class over a date range.
- Optional links from a weak topic to relevant **textbook pages**.

### 2.11 Deployment and operations (school IT)

- Install in school AWS (ap-south-1) via **`exam-eval` CLI** or **CloudFormation Launch Stack**.
- Included: application, database, storage, queues, login, TLS, backups, monitoring dashboards.
- Upgrade path without Kubernetes or GitOps for school IT.

---

## 3. Feature matrix by client

| Feature | Web | Teacher mobile |
|---------|:---:|:--------------:|
| Login | ✓ | ✓ |
| Import roster / admin | ✓ | — |
| Create exam / answer key | ✓ | — |
| Bulk sheet upload | ✓ | — |
| Per-student upload | ✓ | ✓ (camera) |
| Review AI marks | ✓ | Limited / v1 read status |
| Finalize exam | ✓ | — |
| Class & student insights | ✓ | — |
| Student profile (marks) | ✓ | Optional summary |

---

## 4. Explicitly not included (current product scope)

- Student or parent mobile/web portal
- Students uploading their own answer sheets
- Vendor-hosted production datastore for school exam data
- Online proctoring
- Built-in fee collection / billing portal
- QR/barcode on answer sheets for student ID (v1 uses roll-number OCR)

---

## 5. Value outcomes for schools

| Stakeholder | Outcome |
|-------------|---------|
| Teachers | Less time marking; clear view of **which topics to re-explain** |
| Coordinators | Consistent marks data; class-wide diagnostic reports |
| Management | Data stays in **school AWS**; audit-friendly mark changes |
| IT | Predictable install/upgrade; no student data on vendor servers |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-13 | Initial feature catalog |
