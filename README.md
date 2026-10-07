# Aegis — Local Platform

For a client walkthrough and pitch, see [DEMO.md](DEMO.md).


## Application

- `/lab`: public-data baselines, current and archived CFPB analysis, held-out GAN/diffusion benchmarks on public credit data, Monte Carlo simulation, and fictional synthetic cohorts.
- `/check`: private offer check with optional pasted-text warning-sign review, total-cost calculation, and cash-flow forecast. Its active shield uses aggregate CFPB issue and narrative reports to explain review questions alongside the person's offer, while the LendingClub benchmark prompts for missing existing-debt bills. It does not use public data to score a member or alter their cash-flow arithmetic.
- `/member`: sign-in, consent controls, offer comparison, manual cash flow, CSV transaction import, opt-in personal pattern alerts, save-pause prompts, bill-reserve routing previews, saved checks, reminders, export and account deletion.
- `/partner`: staff access to aggregate reports, suppressed below 20 members.
- `/shield-demo`: sign-in-free fictional offer flow with transaction signals, cash-flow warning, pause review, and virtual payday routing.

Runs locally with Next.js, PostgreSQL and Python/NumPy. No paid services or bank aggregator are required. Member and partner accounts require database setup and operator provisioning; see [database setup](db/README.md). Public research features work without a database.

**Release status:** working local implementation. Public production deployment still requires staff MFA, broader login abuse controls, backup erasure reconciliation, an independent security review and partner/pilot validation. No real credit-union partnership is claimed. The synthetic models are experimental and have not been validated for personal financial decisions.

Aegis contains the two modeling phases in the project brief: public-data baseline analysis and a synthetic behavioral ecosystem. It runs locally with open-source tools and keeps raw datasets on the user's machine.

The local Phase 1 run uses historical U.S. LendingClub repayment records for a macro benchmark, a current CFPB payday-product metadata export, and the official July 2026 CFPB archive for complaint narratives. The synthetic work creates fictional gig-worker, student, and low-income-family cohorts, then runs seeded agent and Monte Carlo scenarios with optional experimental GAN and diffusion samplers. A separate experiment trains those generators on open credit data and compares aggregate generated feature distributions with a held-out slice.

After a public offer check, the shield directly combines entered offer terms and cash-flow findings with aggregate CFPB complaint themes. It shows fee and payment review cards with source counts, and the member workspace pauses a save for a projected shortfall, an upfront-payment warning, or incomplete repayment wording in pasted offer text. The LendingClub benchmark is used only to prompt for existing debt bills when they were not entered. These are review prompts, not lender rankings, personal predictions, or proof of a violation. The offer cost and cash-flow comparison still uses the member's entered amounts and dates.

Impulse, stress, and loan-trigger values are generated simulation parameters. Aegis does not infer personal vulnerabilities, create individual credit scores, connect to bank accounts, or move money. Results describe the chosen dataset or fictional scenario; they are not predictions about a person or population.

The website can check offer wording that a person chooses to paste on `/check` or in the member workspace. Transparent phrase rules highlight the matched words, explain why to review them, and link to FTC or CFPB guidance. The pasted text is analyzed in the current browser and is not uploaded or saved. The check does not inspect lender websites, verify a lender's identity or license, or decide that an offer is a scam. The member shield also matches opted-in transaction descriptions for repeated short-term-credit or fee wording; these are reviewable pattern flags, not psychological profiles or findings that a lender is predatory. At offer comparison time, a projected cash shortfall, opted-in pattern, or selected rule can pause saving until the member reviews it. The virtual bill reserve divides entered income across entered bills before the next payday. It is a planning preview: there is no live bank feed, checkout interception, transfer, or account access.

To try the website without signing in, open [`/check`](http://127.0.0.1:3000/check), expand **Check the offer's wording**, and paste text from a loan offer. Aegis displays the exact matched phrase, a plain-language explanation, and source guidance next to the cost and cash-flow results. You can instead open [`/shield-demo`](http://127.0.0.1:3000/shield-demo) for a labeled fictional walkthrough. A member can sign in to import an optional bank CSV, review transaction-description patterns, and use those patterns in the offer pause flow. Everything clients use is available on the website; no browser extension is needed.

Try the member shield with the fictional [sample transaction CSV](examples/shield-demo-transactions.csv). It contains invented entries and no real account data.

## This workspace

The app is running at http://127.0.0.1:3000. Fictional local account credentials and restart instructions are in `LOCAL-ACCESS.local.md` (private and ignored by Git). Its isolated PostgreSQL database listens on port 55433.

For a fresh native PostgreSQL installation, run `node scripts/setup-native-local.mjs`; set `AEGIS_PG_BIN` to the PostgreSQL binary directory if needed. This refuses to overwrite an existing local setup.

## Run locally

Requirements: Node.js 22.12 or later (before Node 23), Python 3.9+, and NumPy for model and simulation commands.

```sh
npm ci
npm run models:install
npm run dev -- --hostname 127.0.0.1
```

Open [http://127.0.0.1:3000](http://127.0.0.1:3000). The research lab at `/lab` has two sections: **Phase 1 · Baseline macro models** and **Phase 2 · Synthetic ecosystem**.

To run the optimized production build locally, use:

```sh
npm run build
npm start
```

Run commands from the project directory. The app reads user-provided datasets from `data/raw/` and writes reports to `data/results/`.

## Phase 1: public datasets

The local workspace uses a LendingClub loan-outcome benchmark, current and archived CFPB complaint summaries, and the Statlog German Credit benchmark as a non-U.S. comparison. LendingClub is the current U.S. baseline. CFPB is U.S. market context, not a bank transaction or loan-default dataset. To refresh and train on the CC0 LendingClub archive:

```sh
npm run models:download:lendingclub
npm run models:train -- --source lendingclub --input data/raw/lendingclub/loans.csv --row-limit 100000 --seed 42 --output data/results/baseline-latest.json
```

The downloader verifies the archive's CC0 metadata and records the source, retrieval date, and SHA-256 under `data/raw/lendingclub/metadata.json`. The benchmark covers historical U.S. personal loans from 2007–2018; it is not payday-specific, current, or representative of Aegis members. This downloaded file contains default and paid outcomes for issued loans. It does not include the rejected-application cohort, so Aegis makes no rejection analysis claim.

In this archive adaptation, `loan_status` 0 means default and 1 means paid/OK; the adapter maps default to the model's positive adverse label. The mapping is covered by a regression test.

To refresh the CFPB payday complaint slice and calculate its separate aggregate report:

```sh
npm run models:cfpb:download
npm run models:cfpb -- --input data/raw/cfpb/payday-complaints.csv --output data/results/cfpb-latest.json
```

The downloader uses the CFPB public API, keeps only payday-product rows and the date, product, issue, and company fields, and writes retrieval details to `data/raw/cfpb/metadata.json`. CFPB complaint volume is not representative of all consumers and is not evidence that a company violated the law. Current API exports are metadata-only: CFPB ceased publishing complaint narratives on August 14, 2026. Previously published narratives are available through the [CFPB FOIA Reading Room](https://www.consumerfinance.gov/foia-requests/foia-electronic-reading-room/cfpb-consumer-complaint-database-narratives-archive/).

To reproduce historical narrative analysis from an official CFPB file:

```sh
npm run models:cfpb:archive:download
npm run models:cfpb -- --input data/raw/cfpb/historical-jul-2026.zip --product-contains 'Payday loan, title loan, personal loan, or advance loan' --source-url https://www.consumerfinance.gov/foia-requests/foia-electronic-reading-room/cfpb-consumer-complaint-database-narratives-archive/ --output data/results/cfpb-loan-narratives-latest.json
```

The July 2026 archive produced 1,793 complaints in that product group, including 410 published narratives. The report contains issue counts and transparent phrase-match totals only; it does not save narrative text. The raw ZIP stays local and is ignored by Git. A complaint is a consumer-submitted account, and phrase matches are exploratory indicators, not validated stress labels or proof of lender misconduct. The site shows the aggregate counts on `/lab` and inside `/check` research context.

To run the Statlog German Credit benchmark for comparison:

```sh
npm run models:download:statlog
npm run models:train -- --source statlog-german --input data/raw/statlog-german/german.data --output data/results/baseline-latest.json
```

The local baseline uses U.S. LendingClub repayment labels. Home Credit Default Risk and FICO Explainable ML / HELOC remain supported inputs, but their files are not included or fetched because their access and use terms must be checked by the data recipient. CFPB complaints remain market context and are never treated as repayment/default labels.

CFPB complaints are summarized separately from repayment models. The archived narrative file is processed for transparent phrase matches for income disruption, payment difficulty, cash shortfall, and fee pressure, then its text is discarded from the report.

Complaint counts are descriptive reports of submitted complaints, not a representative sample, default labels, or lender safety ratings.

## Phase 2: synthetic ecosystem

Run reproducible scenarios for the three fictional cohorts:

```sh
npm run models:simulate -- --profile gig-worker --trials 1000 --seed 42 --output data/results/simulation-latest.json
npm run models:simulate -- --profile student --trials 1000 --seed 42
npm run models:simulate -- --profile low-income-family --trials 1000 --seed 42
```

Optional NumPy GAN and diffusion samplers can operate on generated simulation rows:

```sh
npm run models:simulate -- --profile student --trials 1000 --seed 42 --engine gan --samples 500 --synthetic-output data/results/synthetic.csv
```

The generated event sequences, trigger rules, and aggregate rates are scenario assumptions, not behavioral measurements or causal findings.

To test both generators on real, locally stored public data, run:

```sh
npm run models:download:statlog
npm run models:benchmark:generators -- --source statlog-german --input data/raw/statlog-german/german.data --row-limit 1000 --samples 500 --seed 42 --output data/results/generator-benchmark-statlog.json
npm run models:download:lendingclub
npm run models:benchmark:generators -- --source lendingclub --input data/raw/lendingclub/loans.csv --row-limit 5000 --samples 500 --seed 42 --output data/results/generator-benchmark-lendingclub.json
```

Each run uses an 80/20 train/holdout split and reports aggregate feature mean, spread, and correlation distances against held-out data. Generated records are discarded, and the reports contain no private member records. These are distribution checks, not evidence of privacy guarantees, causal behavior, or accuracy for a consumer decision. `/lab` shows the reports and can rerun either benchmark locally.

## Verify

```sh
npm test
npm run models:test
npm run build
npm audit --omit=dev
```

Raw datasets and generated artifacts under `data/raw/` and `data/results/` are ignored by Git. The included Statlog downloader stores its source attribution beside the data file. The Kaggle and FICO sources must be obtained and used according to their terms.
