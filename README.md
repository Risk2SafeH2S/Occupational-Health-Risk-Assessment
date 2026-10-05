# OHA Studio (v1.1.0)

**Occupational hygiene assessment studio by Risk2Safe.** It carries an IH engagement from scope to a signed report: OHID, similar exposure groups (SEGs), OHRA, exposure statistics (AIHA and EN 689), control evaluation and report generation.

OHA Studio is a single `index.html` file that runs entirely in the browser. Engagement data stays on the consultant's device and in the engagement file they save. Nothing is sent to a server unless the user switches on AI drafting with their own API key.

> **Professional use.** The app supports the hygienist's judgement; it does not replace it. Every exposure rating, SEG boundary, OEL selection, sample exclusion and final decision is made and recorded by a named person.

---

## Features

| Step | What it does |
| --- | --- |
| 0 · Engagement | Engagement type (baseline, compliance, targeted), scope (facilities, workforce, seasons) and exclusions. The decision standard covers OEL rule, statistical test, confidence levels, shift model and minimum n; it locks once agreed and needs a reason to unlock. Also holds the data request register and the kick-off record, whose non-routine tasks feed OHID. |
| 1 · OHID | Areas with hazardous-area zone and agents with hazard sub-category. Also the engagement OEL library (source, legal status, citation), tasks, screening readings and immediate concerns. One-click sour gas hazard library and data request list. |
| 2 · SEGs | SEG register with mandatory definition, rationale and shift length/rotation, plus an approval status. Validation flags GSD > 3, poor lognormal fit and individuals above 2 × GM. |
| 3 · OHRA | Exposure profiles (SEG × agent × averaging time) with exposure rating 0–4, health effect 1–4, confidence and rationale. Includes a suggested Act now / Sample / Confirm list, a risk matrix and a monitoring plan. |
| 4 · Exposure | Sample entry with flow, equipment, lab, context, blanks and exclusions, and CSV import of lab results. Runs the statistics engine and draws a log-probability plot. Ends with the hygienist's recorded decision. |
| 5 · Controls | Control evaluation by hierarchy, RPE adequacy (required PF vs assigned PF), recommendations, the reassessment schedule and MOC review flags. Close-out covers handover and the exposure-database export. |
| 6 · Report | Report preview with print to PDF, Word (.doc) and HTML. Optional AI draft of the executive summary. Sign-off is blocked by open QA items and freezes a SHA-256 snapshot hash. |
| ADNOC OHRM | Optional **HSE-OH-ST03** framework, chosen in the decision standard or with the preset button. It adds an OHID set-up block covering project phase, approach, workshop, the assessment team with an Appendix 4 competency check, clause 7(e) activity coverage and the Appendix 6 information checklist. It also adds a two-stage OHRA: severity 1–6 and likelihood A–F on the Appendix 9 6 × 6 matrix, and quantitative likelihood from the 95% UCL band. Outputs include the survey frequency, HSP and biological-monitoring triggers, a RAP with EOH/HSE/Facility approvals, the Appendix 5 Health Hazard Risk Register (CSV export) and the Appendix 7 deliverables list. |
| QA checks | About 25 validation rules graded Block, Flag or Info. Flags need an acknowledgement note, which carries into report limitations. |
| Audit | Hash-chained audit trail of every edit, with chain verification. |

### Statistics engine (v1.0.0)

- Lognormal descriptive statistics: GM, GSD, arithmetic mean, X95.
- **Arithmetic-mean 95% UCL** by generalized confidence interval (Krishnamoorthy & Mathew), seeded so results are reproducible. It approximates Land's exact method used by IHSTAT; a simulation in the test suite confirms about 95% coverage. Cox's approximation is shown for reference only, because it under-covers at small n.
- **X95 upper confidence limit** and **exceedance fraction with UCL**, from exact noncentral-t calculations (no lookup tables).
- **EN 689 preliminary test** (n = 3–5) and **statistical test** (UTL 70%, 95% vs OEL, n ≥ 6).
- **AIHA exposure category** from X95 and from its UCL.
- **Non-detects:** robust regression on order statistics (Helsel), with multiple detection limits. Falls back to LOD/√2 substitution, labelled as such, when there are fewer than 3 detects or more than 80% censored.
- **Brief & Scala** shift adjustment for TWA limits, driven by each SEG's shift length.
- Probability-plot correlation coefficient as a lognormal fit check.

The engine is validated against published reference values by `tests/stats-regression.mjs`:

- EN 689 Annex F tolerance factors (2.187 at n = 6, 1.820 at n = 30).
- One-sided 95/95 tolerance factors.
- Student t and normal quantiles.

The suite runs on every push before deployment.

---

## Repository layout

```
index.html                      the whole app (HTML, CSS, statistics engine, app logic)
README.md                       this file
tests/stats-regression.mjs      statistics regression suite (Node 18+)
.github/workflows/deploy.yml    runs the suite, then deploys to GitHub Pages
```

The statistics engine sits between `/*STATS-BEGIN*/` and `/*STATS-END*/` markers in `index.html`. The test suite extracts and runs exactly that code, so the tested engine is the shipped engine.

---

## Deploying on GitHub and the Risk2Safe domain

1. **Create the repository** and add these files to the `main` branch.
   - A **public** repository is free with GitHub Pages, but all code is visible.
   - A **private** repository needs a paid GitHub plan for Pages. See "Licensed data" below before choosing.
2. **Enable Pages:** go to Settings → Pages → Build and deployment, and set Source to **GitHub Actions**. The included workflow tests the engine and deploys only if every check passes.
3. **Custom domain:** in Settings → Pages, set the custom domain to the app address you use on risk2safe.com. Then in Cloudflare DNS, add a CNAME from that host to `<your-github-user>.github.io`, and enable HTTPS once the certificate is issued.
4. **Trial / licence gate:** route the app host through your existing Cloudflare Worker, so the Worker checks the trial or licence before passing the request to GitHub Pages. The app needs no changes for this, because the gate controls access to the page, not to data.
5. **Link it from the Risk2Safe catalogue.**

To run locally, open `index.html` in a current browser. No build step or server is needed.

---

## Data, privacy and security

- **Where data lives:**
  - The browser's local storage holds an autosave of the current engagement.
  - The engagement file (`.ohs.json`) is the system of record. Save it to controlled storage and back it up.
- **Encryption:** "Save encrypted file" uses AES-256-GCM with a PBKDF2 (SHA-256, 250,000 iterations) key from your passphrase. There is no passphrase recovery.
- **Workers** are recorded by pseudonym. Do not enter individual health or biological monitoring results; the app links surveillance needs at SEG level only.
- **AI drafting** is off until you add a key in Settings.
  - Calls go directly from the browser to Anthropic or OpenAI.
  - Only aggregated, structured findings are sent; worker identities are never included.
  - AI text is marked as a draft in the app and noted in the report.
  - Check that your client contracts allow engagement data to go to an AI provider.
- **Audit trail:** each entry is hash-chained to the previous one, so tampering is detectable but not prevented. Sign-off records name, credentials, time and a SHA-256 hash of the data; it is not a qualified electronic signature. Any edit after signing withdraws the sign-off.
- **Content Security Policy** restricts network access to Google Fonts and the two AI endpoints.

---

## Licensed data (OELs)

**No OEL values ship with the app.** Each engagement holds its own OEL library, entered with source, legal status, effective date and citation.

- ACGIH TLVs and BEIs are copyrighted. Enter them only from your firm's licence, and never commit them to a public repository.
- The demo engagement uses placeholder values labelled "Demo value only". They are not real limits.
- The OEL selection rule (most protective, legal first, or a specified source priority) is set in the decision standard. A profile can pin a specific OEL, which requires a reason.

---

## Engagement file format

The file is JSON with `meta.format = "ohastudio-engagement"` and `meta.formatVersion = 1`. Files from later versions are migrated on open.

The **exposure-database package** (Controls → Close-out) exports the following for a client's in-house hygienist or a later engagement:

- Agents, OELs and SEGs.
- Profiles with their statistics.
- Samples, recommendations, MOC entries and the handover record.

Recommendations, the SEG register and results also export as CSV.

### Lab results CSV

Create the samples in the app first (sample/COC ID, worker, date, flows). Then import the lab's results with these columns:

```
coc,value,nd
BZ-101,0.12,
BZ-102,0.05,Y
```

`nd` accepts `Y`, `yes`, `1`, `true`, `ND` or `<`. For a non-detect, `value` is the LOD.

---

## Known limitations in v1.0.0

- One user per engagement file. Collaboration means handing over the file; there is no concurrent editing.
- Word export is HTML-based (`.doc`) and opens in Word without native styles.
- Not yet implemented:
  - Bayesian decision analysis (AIHA category probabilities from a prior).
  - Land's exact UCL; the GCI method is used instead. Cross-check a few profiles against IHSTAT on your first ADNOC engagement.
  - Datalogger time-series import and peak analysis for acute toxic gases.
  - Noise (ISO 9612) and heat stress (WBGT) calculators.
  - Unit conversion of results; results must be entered in the unit of the selected OEL.
- QA rules use fixed thresholds (5% flow drift, 10% blanks, 70% of shift duration, GSD > 3). They are not yet configurable per firm.

## ADNOC HSE-OH-ST03 notes

The ADNOC rules follow HSE-OH-ST03 Version 1 (August 2019). Three points in the standard are inconsistent, so the app makes these choices; change them if your ADNOC company directs otherwise:

1. **OEL hierarchy.** Appendix 2, sections 3 and 10, give: ADNOC HSE-OH-ST02 / Cabinet Decree 12 of 2006 Annex 7b, then UK EH40 WEL, then OSHA PEL, then NIOSH REL, then ACGIH TLV. The Appendix 5 legend instead puts NIOSH REL before OSHA PEL. The preset uses the Appendix 2 order, which is stated twice. The priority list is editable in the decision standard and matches on source names, so enter OEL sources with those names.
2. **Survey frequency.** The Appendix 2 Figure 1 note gives 1–10% OEL every 5 years and 10–50% every 4 years. Section (a)(ii)-2 gives under 1% every 5 years, 1–10% every 4, 10–50% every 3, 50–100% annual. The app uses section (a)(ii)-2 because clause 7(h) cites it.
3. **"95% UCL" basis.** The standard defines the UCL as the upper confidence limit of the arithmetic mean and directs IHSTAT, so the default basis is AM UCL95. X95 UCL95 can be selected instead.

Other rules the app applies:

- Severity is rated on health effect alone; likelihood takes account of existing controls.
- Quantitative assessment is flagged as required for severity 3 or above or a Medium rating, unless a justification is recorded.
- High and High-Medium ratings are treated as non-ALARP and require a RAP.
- A 95% UCL above 50% of the OEL places the SEG in the Health Surveillance Plan.
- Medium-and-above risk for an agent with a BMGV or BEI triggers biological monitoring.
- Fixed-area samples are excluded from compliance statistics.

Severity and likelihood suggestions are prompts only: severity from the IARC group or acute toxicity, likelihood from exposure hours per week. Rate against the full Appendix 3 descriptors. The matrix colours were read from Appendix 9 and are checked cell by cell in the test suite.

## Roadmap

1. Bayesian categories and Land's exact UCL, with Expostats cross-validation added to the regression suite.
2. Logger import with peak frequency, duration and magnitude for H₂S and other acute agents.
3. Noise (ISO 9612) and heat stress modules.
4. Native DOCX export.
5. Optional Cloudflare Workers backend for team sync, a lab portal and client read-only access, keeping this static studio as the core.

---

## Standards referenced

- BS EN 689:2018+AC:2019.
- AIHA, *A Strategy for Assessing and Managing Occupational Exposures* (4th ed.).
- BS EN 482.
- ISO/IEC 17025 (laboratory data provenance).
- NIOSH NMAM, OSHA and HSE MDHS methods (method library entries).
- Hierarchy of controls as in ISO 45001.

Confirm the editions your clients require and record them in the engagement.

---

Risk2Safe · H²S Consultants · info@risk2safe.com
