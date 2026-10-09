# NeuroLens

**An acute traumatic brain injury (TBI) clinical communication and decision-support platform, with doctor-approved, multilingual 3D explanations for families.**

Capstone Project I & II · Team 41 · School of Computer Science, Taylor's University
Supervisor: Dr. Soobia Saeed

> ⚠️ **Academic prototype.** NeuroLens is not a medical device and must not be used for clinical decisions. This repository contains **no real patient data**; only public, de-identified datasets are used.

---

## Table of Contents (Preview) 

## _Contents are subjected to change_

1. [About the Project](#about-the-project)
2. [Key Features](#key-features)
3. [System Overview](#system-overview)
4. [Tech Stack (Preview)](#tech-stack-preview)
5. [Repository Structure (Preview)](#repository-structure-preview)
6. [Getting Started](#getting-started)
7. [Team](#team)
8. [Contribution Rules](#contribution-rules)
9. [Versioning and Releases](#versioning-and-releases)
10. [Data and Ethics Policy](#data-and-ethics-policy)
11. [Licence](#licence)
12. [Acknowledgements](#acknowledgements)

---

## About the Project

In acute TBI, every minute counts. After a head CT, the emergency team must pass the findings to neurosurgery quickly, decide whether to operate, and explain the injury to a family who may not share the doctor's first language. Today, images, reports and urgent messages are spread across separate systems, and families often give consent without truly understanding the scan.

NeuroLens brings this together in one secure workflow:

- **Track 1 – Clinical decision:** AI measures the haemorrhage, the emergency doctor refers the case, and neurosurgery receives a closed-loop alert with a transparent decision-support panel.
- **Track 2 – Family communication:** in parallel, a labelled 3D model and a doctor-approved explanation are prepared, so that the family can understand the injury while the patient is prepared for surgery.

The 3D explanation **never delays** the clinical decision, and **every AI output is reviewed by a doctor**.

---

## Key Features

| Module | What it does |
| --- | --- |
| **Case Hub & Referral** | One case thread per episode; state machine from CT to treatment; referral, ownership handover, closed-loop alerts with escalation |
| **Secure Imaging Exchange** | DICOM intake (Orthanc), role-based access, identity layering and pseudonymisation, encryption, audit log, AI model integrity checks |
| **AI Haemorrhage Analysis** | Multi-class segmentation of five haemorrhage subtypes; volume, thickness, midline shift; lesion-to-region mapping |
| **Clinical Decision Support** | GCS recording and trend, Rotterdam CT score, guideline-based surgical checklist (for reference only) |
| **3D Lens** | Layered, labelled 3D model; explain mode with synchronised highlights; timestamp watermark |
| **Explanation Agent** | Family-friendly scripts generated from structured facts; locked safety segment; doctor approval; English and Bahasa Melayu first |
| **Lumi** | Raspberry Pi 5 robot assistant for alerts, narration and scope-limited voice Q&A; stores no patient data |

---

## System Overview

```
Doctor / Nurse PC (browser) ──┐
                              ├── HTTPS / WSS 443 ──► Caddy ──► FastAPI ──► PostgreSQL
Lumi (Raspberry Pi 5) ────────┘                                   │  │
                                                                  │  └──► Redis ──► Celery workers
CT sender ── DICOM ──► Orthanc ── webhook ────────────────────────┘          ├─ AI (GPU)
                                                                              ├─ 3D mesh
                                                     MinIO (encrypted) ◄──────┤─ LLM (Ollama)
                                                                              └─ Speech (STT / TTS)
```

- **REST** for every action that reads data or changes state.
- **WebSocket** for server push only (alerts, case updates, explain-mode synchronisation, Lumi heartbeat). Events carry IDs, not patient details.

---

## Tech Stack Preview

| Area | Technologies |
| --- | --- |
| Frontend | React, TypeScript, Vite, Tailwind CSS, TanStack Query, three.js / react-three-fiber, NiiVue |
| Backend | Python, FastAPI, Pydantic, SQLAlchemy, Alembic, PostgreSQL, Redis, Celery, MinIO |
| Imaging | Orthanc, pydicom, dcm2niix, nibabel, SimpleITK |
| AI | PyTorch, MONAI, nnU-Net, TotalSegmentator, BLAST-CT (baseline) |
| 3D | scikit-image, trimesh, glTF (`.glb`) |
| Language and speech | Ollama (local LLM), faster-whisper, Piper |
| Security | JWT, Argon2, TOTP MFA, TLS 1.3 (Caddy), RBAC, audit hash chain |
| Robot | Raspberry Pi 5, Python asyncio, websockets, reSpeaker XVF3800, gpiozero |
| DevOps | Docker Compose, GitHub Actions, pytest, ruff |

---

## Repository Structure Preview

```
neurolens/
├── .github/        # CODEOWNERS, pull request template, CI workflows
├── backend/        # FastAPI app, state machines, workers, tests
├── ml/             # Haemorrhage segmentation, measurements, evaluation
├── mesh/           # Mask-to-3D pipeline and labelling
├── explainer/      # Explanation templates, prompts, vocabulary
├── frontend/       # Web application and 3D Lens viewer
├── lumi/           # Raspberry Pi robot client
├── orthanc/        # Orthanc (simulated PACS) configuration
├── docs/           # Architecture, interfaces, security, UI
├── scripts/        # Demo data, de-identification, utilities
├── docker-compose.yml
└── Caddyfile
```

---

## Getting Started

> This section will be completed as the project develops.

**Prerequisites:** Git, Docker Desktop (or Docker Engine with Compose), Node.js LTS, Python 3.11+.

```bash
git clone <repository-url>
cd neurolens
cp .env.example .env          # fill in local secrets; never commit .env
docker compose up --build
```

Then open `https://localhost/api/health`. You should see `ok`.

Model weights and datasets are **not** stored in this repository. See `ml/README.md` for download instructions and licences.

---

## Team

| Role | Member         | Responsibilities | Main folders |
| --- |----------------| --- | --- |
| **R1** Team Lead, Backend & Robotics Engineer · **Maintainer** | Chris          | Case state machines, referral and alerts, REST and WebSocket APIs, workers, Lumi, integration and deployment; final approval of all pull requests | `backend/`, `lumi/`, `orthanc/`, `.github/` |
| **R2** Security & Data Protection Engineer | Ong Ding Zhang | Authentication and RBAC, identity layering and de-identification, encryption, audit log, model integrity, threat model | `backend/app/core/`, `backend/app/services/` (security), `docs/security/` |
| **R3** 3D Visualisation Developer | Tan Kah Lok    | Mesh pipeline, layered labels, 3D viewer, explain-mode animation | `mesh/`, `frontend/src/components/Lens3D/` |
| **R4** Frontend & Explanation Experience Developer | Kong Jun Fan   | Web application, explanation agent and script editor, usability testing | `frontend/`, `explainer/` |
| **R5** Clinical AI & Algorithm Lead | He Jing        | Datasets, haemorrhage segmentation, measurements, decision-support rules, clinical correctness review | `ml/`, `backend/app/services/decision_support.py` |

---

## Contribution Rules

### Governance

- **Chris (R1) is the sole maintainer.** Only the maintainer approves and merges pull requests into `main`, creates tags and publishes releases.
- All other members contribute **only through pull requests**. Nobody pushes directly to `main`.
- A pull request may be **approved, returned for changes, or closed without merging**. The maintainer's decision is final; the reason will be given in the pull request.
- Code owners (see `.github/CODEOWNERS`) are asked to review changes to their folders before the maintainer merges.

### Workflow

1. Create or pick an **issue** describing the task.
2. Update your local `main` and create a branch from it (or from the version tag the maintainer specifies):
   ```bash
   git switch main
   git pull
   git switch -c feat/backend-alert-escalation
   ```
3. Commit small, focused changes using the commit format below.
4. Push and open a pull request into `main`:
   ```bash
   git push -u origin feat/backend-alert-escalation
   ```
5. Fill in the pull request template, link the issue, and request a review.
6. Address review comments by pushing new commits to the same branch.
7. After the merge, the branch is deleted automatically. Start the next task from an updated `main`.

### Branch Names

```
<type>/<area>-<short-description>
```

| Type | Use for |
| --- | --- |
| `feat/` | A new feature |
| `fix/` | A bug fix |
| `docs/` | Documentation only |
| `refactor/` | Code changes that neither fix a bug nor add a feature |
| `test/` | Adding or correcting tests |
| `chore/` | Build, CI, dependencies, configuration |
| `exp/` | Experiments that may never be merged (e.g. model trials) |

Areas: `backend`, `security`, `ml`, `mesh`, `explainer`, `frontend`, `lumi`, `infra`, `docs`.

Examples: `feat/ml-volume-measurement`, `fix/frontend-offline-banner`, `docs/security-threat-model`.

### Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<area>): <summary in the imperative mood, max 72 characters>

<optional body: what changed and why>

<optional footer: Closes #12, BREAKING CHANGE: ...>
```

- **type:** `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `style`
- **area:** the same areas as branch names
- Write the summary in British English, in the imperative mood ("add", not "added"), without a full stop.

Examples:

```
feat(backend): add escalation timer for unacknowledged alerts
fix(ml): correct voxel spacing in volume calculation
docs(interfaces): define AI output JSON schema v1
test(security): cover nurse access to clinical endpoints
chore(infra): pin PostgreSQL to version 16
```

A change that breaks an agreed interface **must** include `BREAKING CHANGE:` in the footer and update `docs/interfaces/`.

### Pull Requests

Every pull request must:

- [ ] Target `main` (or the branch the maintainer names)
- [ ] Have a clear title in commit format, e.g. `feat(frontend): add GCS entry form`
- [ ] Link an issue (`Closes #<number>`)
- [ ] Describe what changed, why, and how it was tested
- [ ] Pass CI (lint and tests)
- [ ] Contain **no secrets, `.env` files, patient data, datasets or model weights**
- [ ] Stay focused on one task (roughly under 400 lines changed where possible)
- [ ] Update documentation if an interface or behaviour changes

**Merge method:** squash and merge, so that each accepted pull request becomes one commit on `main`.

### Code Review Expectations

- Reviewers comment on correctness, security, readability and tests.
- Be specific and polite. Suggest, do not demand.
- Medical terms, rules and script templates must be reviewed by **He Jing** before merging.
- Security-related changes must be reviewed by **Ding Zhang** before merging.

### Issues and Labels

| Label | Meaning |
| --- | --- |
| `must` / `should` / `could` | Priority |
| `backend`, `security`, `ml`, `mesh`, `explainer`, `frontend`, `lumi`, `infra`, `docs` | Area |
| `bug`, `feature`, `question` | Type |
| `blocked` | Waiting on another task |

---

## Versioning and Releases

- Versions are **git tags on `main`**, created only by the maintainer: `v0.1`, `v1.0`, `v1.1`, `v1.5`, …
- A **major version** (v1, v2, v3) marks a milestone; a **minor version** (v1.1, v1.5) marks a completed feature block.
- A version is tagged only when all its tasks are merged, CI passes, and the acceptance test has been reproduced on another member's machine.
- Each tag has a GitHub Release describing what it contains.
- If an older version needs a fix, the maintainer creates a `release/vX.x` branch from that tag.

The full roadmap and the owner of each version are in `docs/roadmap.md`.

| Version | Milestone |
| --- | --- |
| v0.1 | Repository skeleton |
| v1.0 | Backend core with stub AI |
| v1.1 / v1.2 | Security foundation / frontend skeleton |
| v1.5 | Closed-loop alerts and Lumi v1 |
| v2.0 / v2.1 / v2.5 | Real AI / decision support / 3D Lens |
| v3.0 | Explanation and explain mode |
| v3.1 | Capstone I demonstration release |

---

## Data and Ethics Policy

- **No real patient data**, in any branch, issue, pull request or comment.
- Only public datasets are used, according to their licences. Each dataset's licence and citation are recorded in `ml/DATASETS.md`.
- Datasets, model weights, `.env` files and generated images are excluded by `.gitignore` and must never be committed.
- If a secret is committed by mistake, tell the maintainer immediately and **rotate the secret**; deleting the file is not enough.
- AI outputs are always presented as **decision support**. The system never states "normal" when no haemorrhage is detected.
- Any user study requires the supervisor's approval and, where needed, ethics approval before it starts.

---

## Licence

All rights reserved by the project team and Taylor's University until the team and supervisor decide on a licence. Third-party components keep their own licences.

---

## Acknowledgements

- **Dr. Soobia Saeed**, project supervisor
- **Dr. Sumathi**, Capstone Project coordinator
- A senior nurse who kindly shared the real-world emergency workflow
- The creators of the public datasets and open-source tools listed in `ml/DATASETS.md`