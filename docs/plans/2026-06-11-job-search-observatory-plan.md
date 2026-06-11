# Job Search Observatory Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `senior-implementation-discipline` before edits and use bounded subagents for implementation tasks when the write set is isolated. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a resume-centered source of truth for career positioning, target roles, market observatory logic, search queries, and n8n implementation handoff.

**Architecture:** The `resume` repository owns strategy and decision rules; `n8n-workflows` owns executable n8n workflow JSON and cluster import details. Documents in `profile/` define the candidate signal; documents in `job-search/` define demand observation and scoring; documents in `automations/` translate strategy into operational workflow requirements.

**Tech Stack:** Markdown documentation, local PDF/DOCX CV assets, GitNexus for repository awareness, n8n for later automation execution.

---

## File Structure

- `profile/positioning.md`: professional positioning, target narrative, strengths, CTO company-size guidance.
- `profile/target-roles.md`: role families, aliases, Italian/English titles, exclusions, and search intent.
- `job-search/market-observatory-spec.md`: data sources, collection cadence, normalized job record, scoring model boundaries.
- `job-search/linkedin-query-seeds.md`: precise saved-search query seeds for LinkedIn/Indeed-style alerts.
- `job-search/italy-market-sources.md`: Italian public/open/accessible sources for demand, sector, and salary-adjacent signals.
- `job-search/scoring-model.md`: scoring dimensions, weights, explainability, and calibration loop.
- `automations/n8n-workflows.md`: mapping from strategy docs to n8n workflows and operational constraints.
- `docs/plans/2026-06-11-job-search-observatory-plan.md`: this execution plan.

## Operating Principles

- Avoid scraping logged-in LinkedIn as the default path; prefer saved-search alerts, email ingestion, public APIs, and public market sources.
- Aggregate by role family, but query by narrow aliases. A broad family is for scoring and analysis, not for a single LinkedIn alert.
- Treat salary sources as weak signals unless they are precise enough for the target role and geography.
- Keep assumptions explicit. If a role recommendation is inferred from CV evidence rather than directly stated, mark it as inference.
- Commit small, reviewable document increments after each coherent milestone.

## Task 1: Positioning Document

**Files:**
- Create: `profile/positioning.md`
- Read: `cv/Ignazio.Ingenito.pdf`
- Read: `profile/sources/cv-analysis/Ignazio.Ingenito.1.pdf`
- Read: `profile/sources/cv-analysis/Ignazio.Ingenito.2.pdf`

- [ ] **Step 1: Extract profile evidence**

Run:

```bash
pdftotext cv/Ignazio.Ingenito.pdf -
```

Expected: readable CV text with current experience, skills, and seniority signals.

- [ ] **Step 2: Draft positioning**

Create `profile/positioning.md` with these sections:

```markdown
# Posizionamento Professionale

## Sintesi

## Punti Di Forza

## Target Primari

## Target Adiacenti

## CTO: Aziende Piccole Vs Grandi

## Ruoli Meno Prioritari

## Narrativa Per Candidature

## Assunzioni Aperte
```

Expected: document in Italian, pragmatic, non-promotional, with explicit distinction between direct CV evidence and inference.

- [ ] **Step 3: Review consistency**

Run:

```bash
rg -n "infer|Infer|CTO|Head of Engineering|Platform|Cloud|AI|Document|Transformation" profile/positioning.md
```

Expected: the document covers the agreed target directions and marks inferred claims where appropriate.

## Task 2: Target Role Families

**Files:**
- Create: `profile/target-roles.md`
- Read: `profile/positioning.md`

- [ ] **Step 1: Define role families**

Create role families for:

```text
CTO / Fractional CTO / Technical Co-founder
Engineering Leadership
Platform / Cloud / DevOps Leadership
AI / Data / Document Intelligence Leadership
Innovation / Digital Transformation
Enterprise / Solution / Cloud Architecture
```

Expected: each family has scope, fit rationale, Italian aliases, English aliases, company-size notes, and exclusion rules.

- [ ] **Step 2: Validate query readiness**

Run:

```bash
rg -n "Alias|Italia|Estero|Escludere|Company|Query" profile/target-roles.md
```

Expected: each role family has enough aliases to generate saved-search queries without collapsing into one broad search.

## Task 3: Market Observatory Specification

**Files:**
- Create: `job-search/market-observatory-spec.md`
- Read: `profile/positioning.md`
- Read: `profile/target-roles.md`

- [ ] **Step 1: Define source strategy**

Document source groups:

```text
Saved-search/job-alert emails
Public job APIs or public job feeds
Italian market sources
Salary and compensation guides as weak signals
Manual review feedback from applications
```

Expected: spec explains what each source can and cannot prove.

- [ ] **Step 2: Define normalized job record**

Document fields:

```text
source
capturedAt
title
company
location
remotePolicy
url
description
roleFamily
seniority
companySizeAssumption
marketDemandSignals
profileFitSignals
candidateAdvantageSignals
recommendedAction
explanation
```

Expected: field list is stable enough for later n8n implementation.

## Task 4: Query Seeds

**Files:**
- Create: `job-search/linkedin-query-seeds.md`
- Read: `profile/target-roles.md`

- [ ] **Step 1: Generate narrow query seeds**

Document precise query seeds grouped by role family. Include examples such as:

```text
"Head of Engineering"
"Engineering Manager"
"Responsabile Sviluppo Software"
"DevOps Manager" Kubernetes
"Cloud Architect" AWS
"Responsabile Cloud"
"AI Lead"
"Document Intelligence"
"Digital Transformation Manager" cloud
"Fractional CTO"
"Technical Co-founder"
```

Expected: no single query tries to represent an entire role family.

- [ ] **Step 2: Add calibration notes**

For each query group, document how to decide whether to keep, split, narrow, or retire a query after observing alert quality.

Expected: the system can improve over time from observed false positives and strong matches.

## Task 5: Italy Market Sources

**Files:**
- Create: `job-search/italy-market-sources.md`

- [ ] **Step 1: Document candidate sources**

Include Italian or Italy-relevant sources:

```text
Unioncamere Excelsior
ISTAT Professioni
INAPP Atlante del Lavoro
Anitec-Assinform
Osservatori Politecnico di Milano
EURES
Cedefop Skills-OVATE
```

Expected: each source has purpose, signal type, likely granularity, update cadence if known, and limitations.

## Task 6: Scoring Model

**Files:**
- Create: `job-search/scoring-model.md`
- Read: `job-search/market-observatory-spec.md`
- Read: `job-search/italy-market-sources.md`

- [ ] **Step 1: Define scores**

Document:

```text
marketDemandScore
profileFitScore
candidateAdvantageScore
applicationPriorityScore
```

Expected: each score has input signals, weights, and explanation text rules.

- [ ] **Step 2: Add Italian-market variant**

Document an Italy-specific demand score that weights observed local alerts and Italian sources more heavily than global salary guides.

Expected: score avoids broad salary-guide overfitting.

## Task 7: n8n Handoff

**Files:**
- Create: `automations/n8n-workflows.md`
- Read: `job-search/market-observatory-spec.md`
- Read: `job-search/scoring-model.md`
- Read: `job-search/linkedin-query-seeds.md`

- [ ] **Step 1: Map strategy to workflows**

Document workflow responsibilities:

```text
email alert ingestion
public source ingestion
normalization
classification
scoring
report generation
manual review capture
```

Expected: n8n implementation remains a consumer of this repo's strategy, not the source of strategy.

## Task 8: Verification And Commit

**Files:**
- All created Markdown files.

- [ ] **Step 1: Check repository status**

Run:

```bash
git status --short
```

Expected: only planned Markdown files are modified or created.

- [ ] **Step 2: Check references**

Run:

```bash
rg -n "source-material|cv/exports" README.md profile job-search automations
```

Expected: no stale paths in tracked project docs.

- [ ] **Step 3: Update GitNexus**

Run:

```bash
gitnexus analyze
gitnexus status
```

Expected: GitNexus reports the current commit/worktree status accurately.

- [ ] **Step 4: Commit**

Run:

```bash
git add docs/plans profile job-search automations README.md
git commit -m "docs: define job search observatory strategy"
```

Expected: one focused documentation commit.
