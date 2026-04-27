# Project Plan — Used Vehicle Opportunity Portfolio

**Head of Data 101 · Session 1 deliverable**

| Field | Value |
|---|---|
| Author | Till Luder |
| Program | MSc Data & AI for Strategic Management, Albert School · Mines Paris / PSL |
| Repository | https://github.com/tluder-dot/HeadOfData101 |
| Scope | Volkswagen Golf · Germany · AutoScout24 public listings |
| Status | Session 1 — product framing |
| Last updated | April 2026 |

---

## 1. Executive summary

This project simulates the work of a data product unit inside a German retail / consumer bank that is evaluating whether to enter the used-vehicle acquisition and financing market. The team (one person, wearing all four delivery hats) owns an end-to-end pipeline — from scraped listings, through a BigQuery warehouse, through regression and classification models, into a business-facing BI dashboard — that surfaces listing-level acquisition opportunities to an internal investment committee.

The scope is deliberately narrow: **used Volkswagen Golf listings in Germany**, sourced from AutoScout24. The Golf is Europe's most liquid used-car segment and the natural entry point for a consumer bank building a financed-resale portfolio. Depth is prioritized over breadth; a defensible single-scope delivery beats a shallow multi-market one.

By the final review, the product answers one question for the committee: **which listings look attractive enough to prioritize for acquisition review this week, and how confident is the data layer in that judgment?**

---

## 2. Business question and product thesis

**The bank's thesis (simulated).** A retail / consumer bank in Germany is evaluating an adjacent retail-investment product: acquire selected used vehicles into an inventory portfolio, resell them through a dealer network, and finance the end buyer on the sale side. The bank already has the financing capability; what it lacks is a repeatable analytical process to identify *which vehicles* deserve acquisition capital.

**The data team's job (this project).** Build the pipeline that turns public listing data into a prioritized, defensible review queue for the committee. The team does not make the acquisition decision. It makes the decision *supportable* — by surfacing the right candidates, the right evidence, and the right limits.

**The decision the product supports.** For each listing in scope:

- what is its observed market price?
- what is the expected price given its characteristics (mileage, age, power)?
- what is the probability the broader market treats it as "top-price" (the external AutoScout tier label)?
- given both signals, should it go on this week's review queue, be monitored, or be deprioritized?

The product explicitly does *not* automate the acquisition decision. It is a prioritization layer that feeds human committee review, consistent with Section 7.5 of the field guide: *"Treat model outputs as screening signals that still require analyst review and business context."*

---

## 3. Scope decision and justification

**Scope:** Volkswagen Golf · Germany · AutoScout24 listings · 30–50 scrape pages · snapshot as of acquisition date.

**One-sentence defense to the committee:** *"We focused on the German used Golf market because the Golf is the single most liquid and price-stable used-vehicle segment in Europe, making it the obvious entry point for a consumer bank building a financed-resale portfolio."*

**Why this scope specifically over alternatives considered:**

- **Audi A3 Germany** (the instructor's baseline scope): perfectly workable but offers no differentiated narrative — "same scope, better execution" has a ceiling. Golf gives the same technical foundation with a stronger thesis.
- **Premium compact (BMW 1, Mercedes A-Class)**: narrower customer financing base for a retail bank. Less credible as a portfolio entry point.
- **Multi-country comparison**: the field guide warns explicitly against partially aligned multi-scope deliveries in Step 1.5. The comparison story is defendable only if definitions remain consistent, which is a cost this delivery doesn't need to pay.
- **Golf in Germany specifically**: dealer-heavy, high-volume, multiple fuel types in meaningful proportions (petrol-dominant, diesel in older listings, growing hybrid/EV share), and eight generations currently in the market simultaneously — enough analytical texture to say interesting things without breaking volume or semantic consistency.

**Sub-variant handling (deferred).** AutoScout24's URL `/lst/volkswagen/golf` returns all variants (base Golf, Golf Variant, Golf Sportsvan, Golf Plus, GTI, R, etc.). The decision to filter or keep all variants is deliberately deferred to Session 3, after inspecting the raw scrape. A real bank analyst would want to see the raw mix before deciding whether sporting variants (GTI/R) and estate variants (Variant) belong in the same portfolio category — that is itself a product judgment, not a cleaning task.

**Scale target:** 30–50 scrape pages → ~800 raw rows → ~500–600 rows post preprocessing. Enough volume for a non-trivial regression fit and a classifier with meaningful precision-recall behavior. Not so much that the scrape becomes fragile or the warehouse becomes slow.

---

## 4. Target user

The primary consumer of the final product is a simulated **internal investment committee** inside the bank, meeting weekly. Members are not data scientists; they are portfolio managers, risk officers, and a Head of Retail Financing. They want:

- a defensible short list of acquisition candidates each week,
- enough context to ask good questions about *why* a listing surfaced,
- visible limits on the signal, so they don't over-trust it.

The committee reads the dashboard in under ten minutes. Technical depth lives in the repo and can be defended verbally; the dashboard is the decision surface. This framing drives the BI design in Section 6.

A secondary consumer is the course reviewer, who will judge whether the whole delivery holds together end to end. Both audiences benefit from the same thing: a pipeline whose logic is explainable without requiring the reader to reconstruct the code.

---

## 5. Product promise and BI outcome

The committee-facing dashboard has **four pages**, each answering one decision-level question. Wireframe mockups (visual-only, placeholder numbers) are in [`docs/wireframes/`](./wireframes/) and linked inline below.

### Page 1 — Market Overview: *"What is in scope?"*

What the committee sees: the size and shape of the market the bank is looking at. Listing count, price bands, fuel and variant mix, age distribution. This page anchors every later number — it is the baseline against which opportunities are framed.

→ [`wireframes/dashboard_page_1_market_overview.svg`](./wireframes/dashboard_page_1_market_overview.svg)

### Page 2 — Expected Price & Gap Analysis: *"What looks mispriced?"*

Output of the regression layer. Actual vs expected-price scatter, residual distribution, top-10 listings below expected. This is the first signal: listings whose price is meaningfully below what the model thinks is normal for their characteristics.

→ [`wireframes/dashboard_page_2_expected_price.svg`](./wireframes/dashboard_page_2_expected_price.svg)

### Page 3 — Opportunity Scoring: *"What looks promising?"*

Output of the classification layer. Probability distribution with a user-tunable threshold, feature importance for transparency, and a ranked review queue of top-scoring candidates. The threshold slider is deliberate: threshold is an operating decision, not a model truth (Section 7.3 of the field guide).

→ [`wireframes/dashboard_page_3_opportunity.svg`](./wireframes/dashboard_page_3_opportunity.svg)

### Page 4 — Committee Recommendation: *"What should we do this week?"*

The hero of the product. An executive summary band, counts of "review now" vs "monitor," an estimated portfolio upside, and a ranked recommendation table with explicit confidence and action flags. A visible limitations panel so the committee sees where the signal stops being trustworthy.

→ [`wireframes/dashboard_page_4_decision.svg`](./wireframes/dashboard_page_4_decision.svg)

### Backward plan — what the BI needs from upstream

Working from the dashboard backward, the pipeline must persist these artifacts to BigQuery:

1. A clean listings table with stable schema — `fact_listings` + dimensions (Sessions 3–4)
2. An analytical view for modeling and BI — `vw_regression_dataset`, `vw_classification_dataset` (Session 5)
3. Expected-price predictions, per listing — `fact_expected_price_predictions` (Session 6)
4. Top-price probabilities, per listing, per model — `fact_top_price_predictions` (Session 7)
5. A BI-ready decision view joining all of the above — `vw_bi_dashboard` with a `decision_flag` column (Session 5, consumed in Session 8)

This backward map is the contract that keeps every session honest about what it owes downstream.

---

## 6. Data source review

**Source:** [AutoScout24](https://www.autoscout24.com), public listings, German market (`cy=D`). Chosen because the course baseline uses it, because the platform exposes a `data-price-label` attribute (the external "top-price" marker this project's classifier targets), and because its HTML structure is relatively stable across listing cards.

### Fields expected per listing card

| Field | Source | Role downstream |
|---|---|---|
| `make` / `model` | `data-make` / `data-model` attribute | dimension key · filter |
| `mileage` | `data-mileage` | regression + classifier feature |
| `price` | `data-price` | regression target (page 2) + classifier feature (page 3) |
| `price_label` | `data-price-label` | classification target (derived to binary `top_price`) |
| `registration` | `data-first-registration` (MM-YYYY) | derives `age_years`, `registration_year`, `registration_month` |
| `fuel` | `data-fuel-type` | dimension + classifier feature |
| `country` | `data-listing-country` | filter (Germany-only by scope) |
| `power_hp` | parsed from card span text via regex | regression + classifier feature |

### Known limitations and constraints (written as product facts, not excuses)

Per the field guide's Step 1.5 framing:

- **Snapshot, not stream.** One scrape = one market snapshot. The product supports a "refresh" concept but the underlying cadence is manual re-run. The committee sees the refresh date on every page.
- **Dealer-biased visibility.** AutoScout24 is dealer-heavy; private listings are underrepresented. The product is consistent with how a bank would realistically source inventory (dealers), so this is a feature, not a bug — but it's stated.
- **No demand signal.** Listings show asking prices, not transaction prices or days-on-market. Expected price ≠ expected resale velocity. Page 4 makes this limitation visible to the committee explicitly.
- **Rate limits and scrape fragility.** The baseline scraper has minimal retry logic and no HTML snapshot archival. Silent coverage drift (Session 2.2 of the field guide) is a real risk; mitigation is field-level null-rate monitoring during the scrape.
- **Price label is external.** `top_price` is derived from AutoScout's own internal pricing algorithm. The classifier is effectively approximating another party's pricing engine. This is stated explicitly in the Page 4 limitations panel so the committee does not mistake it for an independent bank valuation.
- **Volume budget.** 30–50 pages balances two constraints: enough rows for meaningful modeling, few enough requests to stay polite to the source. Daily refresh is not justified — weekly committee cadence doesn't need it.

---

## 7. Baseline vs improvements map

The repo inherited from the instructor is, per the field guide's own framing, *"close to the minimum that could justify a pass, not the ceiling of what the final delivery should become."* This table makes explicit what is being kept, what is being extended, and where value is being added.

| Layer | Baseline (inherited) | This project adds |
|---|---|---|
| Scope | Audi A3 Germany · 3 pages · ~44 rows | Volkswagen Golf Germany · 30–50 pages · ~800 raw rows |
| Scraping | Configurable, single run, saves timestamped raw CSV | Field-level null-rate logging, listing-level dedup during loop, explicit acquisition risk log in README |
| Preprocessing | Schema validation, IQR outliers, logical checks | Month-precision age feature; Golf-specific fuel map extensions (hybrid, electric); variant handling decision documented |
| Warehouse | Fact + 4 dimensions via BigQuery DDL scripts | Own GCP project + dataset; documented build order verification |
| SQL | Read-only example queries against fact + dims | Narrowed analytical views aligned to scope; a "candidates" view combining regression + classification output for BI |
| Regression | LinearRegression (mileage, age, power), in-sample prediction persistence | Out-of-sample persistence pattern (cross-validated predictions or train/predict split); residual diagnostics; coefficient interpretation in business units |
| Classification | Three-model comparison (LogReg, RF, HGB), threshold sweep, all three persisted | Business-aligned threshold selection (not default 0.5); calibration check; an explicit champion-model choice documented in the repo |
| BI | *(not in baseline — explicit gap)* | Four-page Power BI report following the wireframes in this plan |
| Documentation | README + field guide | This PROJECT_PLAN.md + per-session decision logs + wireframes |

The explicit goal is to leave the repo so another reader could open it cold and reconstruct every decision without opening a notebook.

---

## 8. Team, roles, and ownership discipline

**The team is one person.** All four delivery roles defined in the syllabus — Product Owner, Data Analyst (Engineer), Data Scientist, BI Lead — are held by the same contributor. This is not a loophole. The field guide's Section 1.2 rule still applies in the single-person case, just inverted: *the one person must be able to defend the whole pipeline and be honest with themselves about which hat they are wearing at any given moment.*

Practical discipline adopted:

- **One hat at a time.** When writing the scraper (Data Engineer hat), do not make modeling assumptions. When fitting the regression (Data Scientist hat), do not silently fix upstream data issues. Each session's work is framed by its role boundary.
- **Write decisions down as they are made.** Each session's output includes a short decision log (per-session markdown in the README or notebook header) — not because a teammate is reading it, but because Session 9's defense depends on being able to explain every choice end-to-end, and memory degrades fast.
- **Repo is continuously reviewable.** The guide's Section 1 says: *"keep their repo continuously reviewable."* Applied here as: push to GitHub at the end of every working session, even if the state is intermediate. The instructor can see progress at any time at the repo URL above.

---

## 9. Milestones and delivery rhythm

| Session | Deliverable | Status |
|---|---|---|
| 01 | Initial Project Plan | **THIS DOCUMENT** |
| 02 | Raw dataset — one committed CSV under `data/raw/` | Upcoming |
| 03 | Cleaned dataset — processed CSV with stable schema under `data/processed/` | Upcoming |
| 04 | BigQuery warehouse — fact + dims loaded, queryable | Upcoming |
| 05 | Analytical SQL — scoped views for modeling + BI | Upcoming |
| 06 | Regression baseline — expected-price predictions persisted | Upcoming |
| 07 | Classification — top-price probabilities persisted, model evaluation summary | Upcoming |
| 08 | Final BI report — four-page Power BI dashboard | Upcoming |
| 09 | Committee defense — 10-minute presentation, evidence package complete | Upcoming |

Between-session rhythm: one or two substantive commits per working day, repo pushed to GitHub at the end of every session. README and this plan updated whenever a scope, scale, or methodology decision changes — the plan is a living contract, not a Session 1 artifact frozen in time.

---

## 10. Risks and constraints

Named early, revisited each session.

| Risk | Likelihood | Mitigation |
|---|---|---|
| AutoScout24 HTML structure change breaks the scraper mid-project | Medium | Multiple CSS selector fallbacks already in baseline; add captcha-hint logging; save raw HTML sample for manual inspection on first scrape |
| Scrape blocked by rate limiting | Low-Medium | 1-second sleep between pages (already in baseline); single run per session; polite User-Agent |
| Class imbalance in `top_price` makes classification metrics misleading | High | Use precision/recall + ROC-AUC, not accuracy; threshold sweep mandatory; communicate limits in Page 4 |
| Regression overfits on a small dataset | Medium | Keep features to the three interpretable ones; report R² and MAE on held-out split; do not persist in-sample predictions |
| BigQuery setup blocks Session 4 progress | Medium | Set up GCP account and project before Session 4, not the night of |
| Dashboard drifts from warehouse contract | High in the final weeks | Every Power BI measure maps to a named BigQuery view field; no chart-level business logic |
| Solo delivery means no second opinion on modeling choices | Always | Document decisions in plain English in the repo; rehearse defense explanations with an AI reviewer before Session 9 |

---

## 11. Success criteria

The project succeeds if, at Session 9, a cold reviewer can open the repo and:

1. **Understand the business problem** without reading any code — from this plan and the README alone.
2. **Walk the pipeline** end-to-end — from raw scrape to BI — and see one durable artifact per stage, with the contract between stages documented.
3. **Open the dashboard** and follow the four-page decision story in under ten minutes, without asking what any field means.
4. **Find explanations for every non-obvious choice** — scope, variant handling, threshold selection, model selection — in writing, in the repo.
5. **See the limits of the signal clearly.** The product does not over-claim. Committee confidence language is calibrated to the data that supports it.

Accuracy of any individual model is not among the success criteria. Robustness, traceability, and defensibility are.

---

*This plan is a living document. It will be updated whenever a scope, methodology, or architectural decision changes. The most recent version lives in the repository's `docs/` folder, alongside the field guide it complements.*
