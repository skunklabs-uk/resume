# Job Alert Quality Improvement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `senior-implementation-discipline` before edits and `verification-before-completion` before claiming completion. Use `systematic-debugging` before changing parser behavior after a failed run. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Improve job alert quality so Telegram surfaces a small set of permanent, technically relevant roles worth applying to or inspecting manually.

**Architecture:** `resume` remains the source of truth for strategy, scoring policy, and query calibration. `n8n-workflows` owns executable workflow JSON, local parser fixtures, workflow validation, and import notes. The next iteration should improve the LinkedIn email path first, then enrich from public non-logged sources, then tune scoring with explicit fallback behavior for strong but data-poor records.

**Tech Stack:** Markdown strategy docs, n8n workflow JSON, n8n Code node JavaScript, local Node.js fixture tests, GitNexus, Telegram delivery through n8n.

---

## Decisions Locked

- Optimize for a shortlist of permanent roles worth applying to or inspecting, not a generic market tracker.
- Do not scrape LinkedIn with a logged-in session or personal cookies.
- Use public, non-logged sources only for enrichment: URLs in the alert email, public job pages, company career pages, and public ATS pages such as Greenhouse, Lever, Workable, Ashby, and SmartRecruiters.
- Treat `title-strong but data-poor` as a fallback, not the primary scoring path.
- Add local parser/scoring fixtures before modifying more large n8n Code nodes.
- Keep hard exclusions for freelance, contractor, talent network, marketplace, standalone IC Product Engineer/full-stack/software engineer, sales, content, admin, task-based work, junior, and non-technical operations.

## File Structure

- `profile/target-roles.md`: clarify that strong LinkedIn title-only alerts may require manual inspection when no exclusion signal exists.
- `job-search/scoring-model.md`: add the `dataPoor` fallback policy and decision rules for `inspect manually`.
- `job-search/market-observatory-spec.md`: document optional enrichment fields and public-source-only policy.
- `automations/n8n-workflows.md`: document parser fixture workflow, enrichment boundaries, and output expectations.
- `docs/plans/2026-06-12-job-alert-quality-improvement-plan.md`: this plan.
- `/home/iingenito/projects/personal/n8n-workflows/scripts/`: add local validation helpers for workflow Code node behavior.
- `/home/iingenito/projects/personal/n8n-workflows/tmp/fixtures/`: add sanitized, non-secret email alert fixtures.
- `/home/iingenito/projects/personal/n8n-workflows/workflows/job-search-email-alerts.json`: update parsing, optional enrichment, scoring fallback, and Telegram digest.

## Task 1: Document The Data-Poor Fallback

**Files:**
- Modify: `job-search/scoring-model.md`
- Modify: `job-search/market-observatory-spec.md`
- Modify: `profile/target-roles.md`
- Modify: `automations/n8n-workflows.md`

- [x] **Step 1: Update scoring policy**

Add a section to `job-search/scoring-model.md`:

```markdown
## Data-Poor Alert Fallback

Saved-search email alerts, especially LinkedIn digests, may expose title,
company, location and link but not the full job description. In this case the
system must not infer detailed fit or candidate advantage.

If a record is from a job-alert email, has a strong target title, has no hard
exclusion signal, and lacks enough description to score responsibilities or
domain fit, mark it as `dataPoor: true`.

Decision rule:

- strong target title + no hard exclusion + data poor -> `inspect manually`;
- weak or ambiguous title + data poor -> `ignore` or `monitor`;
- hard exclusion + data poor -> `ignore`;
- enriched public description available -> score normally.

This fallback is a triage rule, not a boost to `apply`.
```

- [x] **Step 2: Update normalized record spec**

Add optional fields to `job-search/market-observatory-spec.md`:

```markdown
- `dataPoor`: boolean indicating that the record has too little description for
  reliable fit/advantage scoring.
- `enrichmentStatus`: `not_attempted`, `found_public_source`, `not_found`,
  `blocked_login`, or `failed`.
- `enrichmentSource`: public URL used for additional text, when available.
```

- [x] **Step 3: Update target role guidance**

In `profile/target-roles.md`, add a calibration note:

```markdown
For LinkedIn email alerts, strong titles such as CTO, Chief Technology Officer,
Head of Engineering, Engineering Manager, Director of Engineering, VP
Engineering, Platform Engineering Lead, Cloud Engineering Manager and DevOps
Lead may require manual inspection when the email does not include enough
description. This must not override hard exclusions or permanent-employment
constraints.
```

- [x] **Step 4: Update n8n handoff**

In `automations/n8n-workflows.md`, document that executable workflows should:

```markdown
- parse LinkedIn HTML before falling back to plain text;
- enrich only from public non-logged sources;
- mark strong title-only records as `dataPoor`;
- send data-poor strong-title records to manual inspection, not automatic apply.
```

- [ ] **Step 5: Commit strategy docs**

Run:

```bash
git status --short
git add profile/target-roles.md job-search/scoring-model.md job-search/market-observatory-spec.md automations/n8n-workflows.md docs/plans/2026-06-12-job-alert-quality-improvement-plan.md
git commit -m "docs(job-search): define data-poor alert policy"
```

Expected: one documentation commit in `resume`.

## Task 2: Add Local Fixtures For LinkedIn Alert Parsing

**Files:**
- Create: `/home/iingenito/projects/personal/n8n-workflows/tmp/fixtures/linkedin-head-of-engineering-alert.json`
- Create: `/home/iingenito/projects/personal/n8n-workflows/scripts/test-job-search-email-alerts.js`
- Read: `/home/iingenito/projects/personal/n8n-workflows/workflows/job-search-email-alerts.json`

- [x] **Step 1: Create sanitized fixture**

Create a fixture shaped like the Gmail node output:

```json
{
  "id": "fixture-linkedin-head-of-engineering",
  "subject": "Ignazio: il tuo avviso di offerte di lavoro per “Head Of Engineering” (Italia)",
  "from": "jobalerts-noreply@linkedin.com",
  "date": "2026-06-12T05:33:19.000Z",
  "textPlain": "Vedi le nuove offerte di lavoro... Chief Technology Officer WeHunt · Verona, Veneto, Italia ... CTO Jobgether · Italia ... Technical Director Energy Transition (Italy) Mott MacDonald · Genova, Liguria, Italia ...",
  "textHtml": ""
}
```

Keep the fixture free of personal email content beyond already observed public
job alert text. Do not include tokens, cookies, tracking links with private
parameters, or credential values.

- [x] **Step 2: Create a workflow Code-node test runner**

Create `scripts/test-job-search-email-alerts.js` that:

```javascript
const fs = require('fs');
const vm = require('vm');

const workflow = JSON.parse(fs.readFileSync('workflows/job-search-email-alerts.json', 'utf8'));
const fixture = JSON.parse(fs.readFileSync('tmp/fixtures/linkedin-head-of-engineering-alert.json', 'utf8'));

function codeNode(name) {
  const node = workflow.nodes.find(candidate => candidate.name === name);
  if (!node) throw new Error(`Missing node: ${name}`);
  return node.parameters.jsCode;
}

function runParse() {
  const code = codeNode('Parse and Score Alerts');
  return vm.runInNewContext(`(function(){${code}\n})()`, {
    $input: { all: () => [{ json: fixture }] },
    console
  }, { timeout: 5000 })[0].json;
}

function assert(condition, message) {
  if (!condition) throw new Error(message);
}

const report = runParse();
assert(report.parsedCount >= 3, `Expected at least 3 parsed records, got ${report.parsedCount}`);
assert(report.records.some(job => /Chief Technology Officer/i.test(job.title)), 'Missing CTO record');
assert(report.records.some(job => /CTO Jobgether/i.test(job.title)), 'Missing CTO Jobgether record');
assert(!report.records.some(job => /Vedi le nuove offerte/i.test(job.title)), 'Digest preheader became a job title');

console.log(JSON.stringify({
  parsedCount: report.parsedCount,
  matchCount: report.matchCount,
  titles: report.records.map(job => job.title)
}, null, 2));
```

- [x] **Step 3: Run fixture test before behavior change**

Run:

```bash
node scripts/test-job-search-email-alerts.js
```

Expected: the current parser test passes for split behavior, but it may still
show `ignore` for strong title-only records. That is the baseline to improve in
Task 4.

Observed baseline on 2026-06-12: the current parser failed all 8 real LinkedIn
fixtures, parsing 8 records instead of 48 and leaking the email preamble as the
title. This justified the parser fix before scoring changes.

## Task 3: Improve LinkedIn Email HTML Parsing

**Files:**
- Modify: `/home/iingenito/projects/personal/n8n-workflows/workflows/job-search-email-alerts.json`
- Modify: `/home/iingenito/projects/personal/n8n-workflows/tmp/fixtures/linkedin-head-of-engineering-alert.json`
- Modify: `/home/iingenito/projects/personal/n8n-workflows/scripts/test-job-search-email-alerts.js`

- [ ] **Step 1: Extend fixture with sanitized HTML**

If the Gmail item includes `textHtml`, add a sanitized representative HTML
sample with repeated job blocks and links. Keep only enough structure to test:

```html
<a href="https://www.linkedin.com/comm/jobs/view/...">Chief Technology Officer</a>
<span>WeHunt</span>
<span>Verona, Veneto, Italia</span>
```

- [x] **Step 2a: Add plain-text structured extraction**

Added a LinkedIn plain-text parser for Gmail alert blocks using the repeated
`Visualizza offerta di lavoro` rows, preserving canonical `/jobs/view/<id>` URLs
and separate title/company/location fields. HTML-first extraction remains a
future hardening step if plain text stops being reliable.

- [ ] **Step 2b: Add HTML-first extraction**

Update the parser Code node so LinkedIn parsing attempts structured HTML
extraction before plain-text splitting.

Expected normalized records:

```json
{
  "title": "Chief Technology Officer",
  "company": "WeHunt",
  "location": "Verona, Veneto, Italia",
  "url": "https://www.linkedin.com/...",
  "source": "LinkedIn alert email"
}
```

- [x] **Step 3: Keep plain-text fallback**

If structured HTML extraction finds no repeated job blocks, keep the current
plain-text splitter. The fallback must still avoid creating a preheader job.

- [x] **Step 4: Verify parser extraction**

Run:

```bash
node scripts/test-job-search-email-alerts.js
```

Expected:

- `parsedCount` matches the number of job rows in the fixture.
- LinkedIn records have separate `title`, `company`, `location`, and canonical
  job URL when the plain-text alert provides them.
- No record title starts with `Vedi le nuove offerte`.

## Task 4: Add Data-Poor Scoring Fallback

**Files:**
- Modify: `/home/iingenito/projects/personal/n8n-workflows/workflows/job-search-email-alerts.json`
- Modify: `/home/iingenito/projects/personal/n8n-workflows/scripts/test-job-search-email-alerts.js`

- [x] **Step 1: Detect data-poor records**

In the scoring Code node, calculate:

```javascript
const usefulDescriptionLength = normalize(description)
  .replace(normalize(title), '')
  .replace(normalize(record.company || ''), '')
  .replace(normalize(record.location || ''), '')
  .trim()
  .length;
const dataPoor = record.source === 'LinkedIn alert email' && usefulDescriptionLength < 120;
```

- [x] **Step 2: Define strong target titles**

Use explicit title patterns:

```javascript
const strongTitlePatterns = [
  /chief technology officer/i,
  /\bcto\b/i,
  /head of engineering/i,
  /engineering manager/i,
  /director of engineering/i,
  /vp engineering/i,
  /platform engineering lead/i,
  /cloud engineering manager/i,
  /devops lead/i
];
const strongTargetTitle = strongTitlePatterns.some(pattern => pattern.test(title));
```

- [x] **Step 3: Apply fallback action**

After hard exclusions and before final return:

```javascript
if (dataPoor && strongTargetTitle && !hardGate.outOfScope && !hasHardTitleExclusion) {
  recommendedAction = 'inspect manually';
}
```

Keep `apply` impossible for data-poor records unless a later enrichment step
provides a real description.

- [x] **Step 4: Add assertions**

Extend the test runner:

```javascript
const cto = report.records.find(job => /Chief Technology Officer/i.test(job.title));
assert(cto.recommendedAction === 'inspect manually', `Expected CTO to inspect manually, got ${cto.recommendedAction}`);
assert(cto.dataPoor === true, 'Expected CTO fixture record to be dataPoor');
```

- [x] **Step 5: Verify fallback**

Run:

```bash
node scripts/test-job-search-email-alerts.js
```

Expected: strong title-only records become `inspect manually`; excluded roles
remain `ignore`.

## Task 5: Public Enrichment Spike

**Files:**
- Modify: `/home/iingenito/projects/personal/n8n-workflows/workflows/job-search-email-alerts.json`
- Modify: `/home/iingenito/projects/personal/n8n-workflows/scripts/test-job-search-email-alerts.js`
- Optional create: `/home/iingenito/projects/personal/n8n-workflows/docs/job-alert-enrichment-notes.md`

- [ ] **Step 1: Add enrichment contract only**

Do not implement broad web crawling first. Add fields to records:

```json
{
  "enrichmentStatus": "not_attempted",
  "enrichmentSource": ""
}
```

- [ ] **Step 2: Decide first public enrichment path**

Use this order:

1. URL already present in LinkedIn email.
2. If URL is a LinkedIn redirect, preserve it for manual inspection.
3. Do not fetch LinkedIn if login is required.
4. Later spike: search company careers/ATS by `title + company`.

- [ ] **Step 3: Stop condition**

Stop the spike if enrichment requires cookies, login, browser automation with a
personal session, or broad search scraping. Mark:

```json
{
  "enrichmentStatus": "blocked_login"
}
```

Expected: no logged LinkedIn automation is introduced.

## Task 6: Telegram Digest Recalibration

**Files:**
- Modify: `/home/iingenito/projects/personal/n8n-workflows/workflows/job-search-email-alerts.json`

- [x] **Step 1: Split digest sections**

Telegram should show:

```text
Matches
...

Needs manual inspection
...

Below threshold
...
```

- [x] **Step 2: Keep message compact**

Limit to:

- up to 5 `matches`;
- up to 5 `inspect manually`;
- up to 3 below-threshold examples only when no stronger records exist.

- [x] **Step 3: Verify Telegram text locally**

Run the parser and formatter with the fixture.

Expected: CTO title-only appears under `Needs manual inspection`, not hidden
inside `Best parsed jobs below threshold`.

Observed on 2026-06-12: the local runner validates Telegram sections across 8
sanitized LinkedIn fixtures. `inspect manually` records appear under `Needs
manual inspection`; below-threshold examples are capped separately.

## Task 7: Validation, Import, And Manual Feedback Loop

**Files:**
- Modify: `/home/iingenito/projects/personal/n8n-workflows/docs/2026-06-11-job-search-radar-handoff.md`
- Modify: `/home/iingenito/projects/personal/resume/job-search/scoring-model.md`

- [ ] **Step 1: Run workflow validation**

Run in `n8n-workflows`:

```bash
./scripts/validate-workflows.sh
node scripts/test-job-search-email-alerts.js
npx gitnexus detect-changes --repo n8n-workflows
```

Expected: workflow JSON validates, fixture test passes, GitNexus risk is low or
understood.

- [ ] **Step 2: Commit workflow changes**

Run:

```bash
git status --short
git add workflows/job-search-email-alerts.json scripts/test-job-search-email-alerts.js tmp/fixtures/linkedin-head-of-engineering-alert.json docs/2026-06-11-job-search-radar-handoff.md
git commit -m "fix(workflows): improve LinkedIn alert triage"
git push
```

- [ ] **Step 3: Import to n8n**

Run the existing importer from the cluster environment.

Expected:

```text
Importing 7 workflows...
Successfully imported 7 workflows.
```

- [ ] **Step 4: Manual test**

Run `Job Search Email Alerts` manually in n8n.

Expected:

- Telegram sends without n8n attribution.
- LinkedIn email parses into separate jobs.
- Strong title-only records appear as `inspect manually`.
- `matchCount` still counts only true top matches.

- [ ] **Step 5: Capture feedback**

For each Telegram item, classify manually:

```text
keep | split | narrow | retire | apply | inspect-only | ignore-correct | ignore-wrong
```

Feed repeated outcomes back into `job-search/scoring-model.md` and
`job-search/linkedin-query-seeds.md`.

## Critic Review

### Risk 1: We Overfit To One LinkedIn Email

Mitigation: use at least two fixtures before broad scoring changes: one good
CTO/Head alert and one noisy alert. Do not change global weights from one run.

### Risk 2: Data-Poor Fallback Creates Too Many Manual Inspections

Mitigation: only strong title patterns qualify. Ambiguous titles such as
Technical Lead, Product Engineer, Software Engineer, AI Engineer, and Solution
Architect do not get automatic `inspect manually` without supporting context.

### Risk 3: Enrichment Becomes Scraping

Mitigation: public-source-only policy is explicit. Stop at `blocked_login` for
LinkedIn login walls. No personal cookies, no browser session automation.

### Risk 4: Workflow Code Nodes Become Unmaintainable

Mitigation: add local fixture tests now. Do not introduce a service or package
until repetition proves it is worth the overhead.

### Risk 5: Telegram Looks Better But Decisions Stay Bad

Mitigation: Telegram is only the digest. The real success metric is whether
manual feedback shows fewer obvious false positives and fewer missed strong
roles after one week of alerts.

## Success Criteria

- LinkedIn digest email no longer creates preheader records.
- Strong permanent title-only records become `inspect manually`, not `ignore`.
- Standalone IC, freelance, contractor, sales, content, admin and task-based
  roles remain excluded.
- Telegram shows a short actionable section for manual inspection.
- The system still avoids logged LinkedIn scraping.
- At least one fixture test protects parser and scoring behavior before import.
