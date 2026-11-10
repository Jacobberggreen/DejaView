<p align="center">
  <img src="docs/images/logo.svg" width="84" alt="Déjà View logo">
</p>

<h1 align="center">Déjà View</h1>

<p align="center">
  <b>Build Less, Build Better.</b><br>
  Ask for a dashboard in plain English. Déjà View first checks whether one already exists, and only builds a new one when it has to.
</p>

<p align="center">
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/license-All%20rights%20reserved-red" alt="All rights reserved">
  </a>
  <img src="https://img.shields.io/badge/React-19-61dafb?logo=react&logoColor=white" alt="React">
  <img src="https://img.shields.io/badge/Vite-8-646cff?logo=vite&logoColor=white" alt="Vite">
  <img src="https://img.shields.io/badge/runs-100%25%20in%20the%20browser-success" alt="Runs 100% in the browser">
  <img src="https://img.shields.io/badge/tests-129%20unit%20%C2%B7%2032%2F32%20decision%20cases-brightgreen" alt="Tests">
</p>

<p align="center">
  Team <b>Error code 418</b> · Since AI Hackathon, Turku, 6-8 Nov 2026<br>
  Challenge: <b>Bayer, "Build Less, Build Better: AI for Governed Analytics"</b>
</p>

> [!CAUTION]
> ## ⛔ PROPRIETARY SOFTWARE. ALL RIGHTS RESERVED.
> **Copyright © 2026 Team Error code 418. This is NOT open source.**
>
> You may **look** at this page. You may **NOT** copy, reuse, modify, redistribute, host, sell, fork into your own product, train on, or build anything derived from this code, design, prompts, data or documentation, in whole or in part, without **prior written permission** from the authors. Seeing the code does not give you any license to use it. See [LICENSE](LICENSE).

**[▶ Watch the demo video](media/deja-view-demo.mp4)**

Everything runs in the browser. There is no backend, and no BI metadata leaves the user's machine.

[Demo video](media/deja-view-demo.mp4) · [How it works](#how-it-works) · [Governance](#governance-controls)

> The source code is in a private repository and is not published. This repo only presents the project.

## The problem

Self-service BI creates dashboard sprawl: duplicated KPI logic, unclear owners, technical debt. AI agents make it worse, because building one more dashboard becomes effortless. Bayer's own words: *"The next breakthrough may not be building more analytics. It may be knowing what deserves to exist at all."*

## What it does

Type a request such as *"Show Kerendia in Sweden against budget per month"*. The tool:

1. **Understands** it: product, market, measure, period (shown as editable chips, so the user can correct it).
2. **Searches** the catalog of 360 existing reports and scores them on what they actually measure, not what they are called.
3. **Decides**, by rules, one of four outcomes:

| Outcome | Meaning | Dashboards in the catalog |
|---|---|---|
| **REUSE** | A trusted report already answers it. Open or export it. | ±0 |
| **EXTEND** | A new report replaces an untrusted one, on the same approved data source. | ±0 |
| **CREATE** | Nothing fits, but an approved data source has the KPIs: build a thin report on it. | +1 |
| **No build** | No approved data source has the KPIs: send a request to the BI team. | 0 |

4. **Explains** the decision in three fixed parts: *What to do / Why / What changes*.
5. **Exports** a Power BI project (`.pbip`) with a `governance.json` next to it, and **registers** the new report in the catalog, so the next person who asks finds it.

Nothing is created without an explicit click, and the app never creates a data source or changes an existing dashboard.

<table>
  <tr>
    <td width="50%"><img src="docs/images/home.png" alt="Start page: a request is typed, chips show what was understood, and a map of 360 dashboards lights up the ones that could fit"></td>
    <td width="50%"><img src="docs/images/matches.png" alt="Matches: three existing dashboards ranked, with warnings such as inactive owner and not certified"></td>
  </tr>
  <tr>
    <td align="center"><b>1. Describe</b><br>Plain English, with editable chips showing what was understood</td>
    <td align="center"><b>2. Match</b><br>Existing dashboards ranked, with the trust warnings visible</td>
  </tr>
  <tr>
    <td width="50%"><img src="docs/images/review.png" alt="Review: the decision, governance rules passed, data source and lineage"></td>
    <td width="50%"><img src="docs/images/build.png" alt="When nothing suitable exists: a new dashboard on an approved data source, +1 in the catalog"></td>
  </tr>
  <tr>
    <td align="center"><b>3. Review (reuse)</b><br>The decision, governance rules, owner and lineage</td>
    <td align="center"><b>Or build</b><br>Only when nothing fits: +1 dashboard, on an approved data source</td>
  </tr>
</table>

## Demo path (about 3 minutes)

1. **Sales rep A** asks for Kerendia in Sweden against budget. A follow-up question appears where it matters (e.g. the period, or "certified EMEA report vs. uncertified Sweden report"). The result is an EXTEND or CREATE with the reasoning shown. Export the `.pbip`.
2. **Sales rep B** asks almost the same thing and gets REUSE: *"It already exists."* That is "Build less" in practice.
3. **Review** shows the evidence before anything happens: which local KPI variants were replaced by the official one, which governance rules applied, who owns the report, its data source and lineage.

## How it works

Code drives the flow and the AI assists. Small local models (3-8B) are weak at free reasoning, so we use them only where text must be written or compared.

```
Question → Interpreter → Matcher → Searcher → Governance → Explainer → approval
                                                                          │
                          REUSE: export existing report                   │
                          EXTEND / CREATE: build .pbip → register         │
                          No build: request to the BI team  ◄─────────────┘
```

| Agent | Job | Uses an LLM |
|---|---|---|
| Orchestrator | Runs the flow, owns the shared state, pauses for follow-up questions, the only contact with the UI | No |
| Interpreter | Splits the question into product, market, measure, period; phrases follow-up questions | Yes, when the measure is not written plainly |
| Matcher | Maps each part to the data (synonyms, then embeddings, then the official KPI) | No (embeddings) |
| Searcher | Scores existing reports, detects ambiguity | No |
| Governance | REUSE / EXTEND / CREATE / No build, and records which rules decided it | No |
| Explainer | Three-part explanation from facts on the shared state only | No (an LLM text is optional and off by default) |

Agents never talk to each other or the UI directly; they read and write one shared state (a blackboard). Every decision is therefore traceable.

## AI approach

| Technique | Library | Used for |
|---|---|---|
| Local LLM | WebLLM (Qwen3, WebGPU) | Interpreting vague measures, phrasing follow-up questions. Output is validated JSON; invalid output is retried, then falls back to a question. |
| Embeddings | Transformers.js (`bge-small-en-v1.5`) | Comparing meaning: "how it's going" lands near *Net Sales*. It can only rank what exists, so it cannot invent a KPI. |
| Rules | Plain code | Matching weights, trust checks and the decision cases. Fast, predictable, and the demo works even if the model misbehaves. |
| Database | SQLite-WASM | The catalog, in the browser. |
| Packaging | JSZip | The `.pbip` download. |

**The LLM never writes Power BI files.** It only produces a small specification; code validates it against the catalog and fills in a template exported from Power BI Desktop. Every LLM text that is shown is checked for invented numbers and for contradicting the decision.

Reports are scored on KPI coverage (40%), endorsement (20%), usage in the last 3 months (15%), active owner (10%), fresh data source (10%) and no open signals (5%). The weights are a starting point.

## Governance controls

- **Trusted** means certified or promoted, active owner, active lifecycle, and a healthy data source. Certification alone is not enough: in the data, 48 of 50 certified reports use non-canonical KPIs.
- **Official KPI first** (RULE-002): local variants such as *Net Sales Local* are swapped for the official KPI and the user is told. Exceptions need a documented reason.
- **Owner required** (RULE-003) and **lineage shown** (RULE-004): every new report has an owner and a data source, and the explanation says which other dashboards depend on it.
- **Do not migrate waste** (RULE-006): legacy and retire-candidate reports are never replaced.
- **Every action is explained** (RULE-008) and written to `decision_log` with the rules that decided it.
- **The human decides.** EXTEND, CREATE and requests always need a confirming click, and buttons say what happens.
- `governance.json` is exported with each new report: source request, decision, data source, owner, KPIs, candidates checked, approval.

## Data assumptions

The data is Bayer's **synthetic** builder data pack (24 sheets, loaded into an in-browser SQLite catalog). Nothing here is real Bayer performance. We checked the data and our findings changed the design:

- Lifecycle flags and endorsement do not predict usage, so we base trust on owner, KPI status, refresh health and lineage together.
- Report names say almost nothing about content, so we match on the KPIs a report covers.
- The product in a request must be read from its text, not from `product_context` (it disagrees in 65 of 75 rows).
- The data pack has only 10 unique request texts. **We wrote our own sales-rep phrasings and the synonym table (`kpi_synonyms`, everyday words to KPIs). Those are our assumptions, not Bayer data.**
- `Migration_Backlog` suggestions are inputs, not answers.

## Enterprise BI integration

In the prototype the Excel data pack is the catalog. In production the catalog is filled from the BI platforms:

- **Power BI:** Scanner API (Admin API) for tables, measures and lineage; Fabric REST API (Get Report Definition, PBIR) for visuals.
- **Tableau:** Metadata API (GraphQL) for fields and lineage; REST API for content, owners and usage.
- **Microsoft Graph (Entra ID):** only to check whether an owner is still active.
- **Publishing** would set owner, workspace and endorsement through the Power BI REST API from `governance.json`.
- **Permissions:** delegated, so the tool only sees what the user already may see; read-only except when publishing.
- **The model is pluggable** behind `ai/llm.js` (e.g. a local Ollama or a company-hosted model).

## Known limitations

- **The app writes directly to the catalog.** That is a prototype simplification. In production, registration goes through the BI platform's APIs.
- The catalog is saved in the browser (OPFS), so it is per browser and per user.
- No authentication and no real publishing to Power BI. The user downloads the `.pbip`.
- Needs WebGPU for the language model, and a first-time model download. The decisions, search and explanation work without the language model.
- A `.pbip` can only be opened in Power BI Desktop, which runs on Windows.
- Synthetic data: the report names, owners and figures are illustrative.
- Duplicate search and Tableau-to-Power BI migration are out of scope (a separate task).

## Team

**Jacob Berggren, Omar Lundgren, Sebastian Gabrielsson, Adam Hultgren**. Built with the help of Claude Code.

## License

**Copyright © 2026 Team Error code 418. ALL RIGHTS RESERVED.**

This is proprietary software, not open source. No permission is granted to use, copy, modify, distribute, host, sell or create derivative works from any part of this project without prior written permission from the authors. See [LICENSE](LICENSE).
