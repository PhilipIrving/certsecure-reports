# CertSecure Report Generator
### Serviced by HUB International

Internal tool for generating compliance and engagement reports across HUB CertSecure clients.
No login required · No API keys for end users · Runs entirely in the browser.

---

## What This Does

The Report Generator reads compliance and engagement data from synced CSV files in the `evident-powerbi` repository and builds four report types for any of 16 clients — all inside the browser. No server needed, nothing to install.

| Report | Format | What it covers |
|---|---|---|
| Compliance Summary | PDF | KPIs, status breakdown, COI status, expiring coverage |
| Compliance Detail | Excel (5 sheets) | Full entity-level compliance data, NC reasons, expiring coverage |
| Engagement Summary | PDF | Email KPIs, open rate, delivery issues, 60-day funnel |
| Engagement Detail | Excel (5 sheets) | No COI list, no engagement, NC + no recent engagement, undeliverables |

---

## File Structure

When you push to GitHub, the repo should look exactly like this:

```
certsecure-reports/
├── index.html          ← the app (ROOT level — not in a subfolder)
├── README.md
└── assets/
    └── logo.png
```

> **GitHub Pages serves `index.html` automatically from the root.** If it ends up in a subfolder, the app will not load.

---

## Setup (First Time)

### 1. Create a new GitHub repository

1. Go to [github.com/new](https://github.com/new)
2. Name it `certsecure-reports`
3. Set visibility to **Public** — GitHub Pages requires this on a free account
4. Do **not** initialize with any files (no README, no .gitignore)

### 2. Push the files

Open a terminal in the folder where you downloaded `index.html` and run:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/PhilipIrving/certsecure-reports.git
git push -u origin main
```

### 3. Enable GitHub Pages

1. In the repo, go to **Settings → Pages**
2. Under **Source**, select `Deploy from a branch`
3. Set branch to **main**, folder to **/ (root)**
4. Click **Save**
5. Your app will be live at: `https://philipirving.github.io/certsecure-reports/`

Share that URL with the team. It's live immediately.

---

## Configure the App

Click **⚙ Settings** in the top-right corner and fill in:

| Field | What to enter | Default |
|---|---|---|
| GitHub Repository Owner | Your GitHub username | `PhilipIrving` |
| Repository Name | The repo where your CSVs live | `evident-powerbi` |
| Branch | Branch name | `main` |
| GitHub PAT | Token with `workflow` scope — only needed for **Refresh Data** | *(leave blank if not using)* |
| Insureds / Parties CSV | Path to compliance CSV in your data repo | `data/insureds.csv` |
| Requirements / Coverage CSV | Path to per-coverage data CSV | `data/requirements.csv` |
| Notifications / Engagement CSV | Path to email engagement CSV | `data/notifications.csv` |

Settings are saved in your browser's local storage — enter once, persists across sessions.

---

## How to Generate a Report

### Step 1 — Select a Client

All 16 clients are listed in three groups:

- **General** (9 clients) — compliance tracked by Entity Type
- **Subcontractors** (4 clients) — compliance tracked by Project
- **Tenants** (3 clients) — compliance tracked by Location

Click any client card to continue.

### Step 2 — Configure

Click a report card to enable it (highlights blue). Enable any combination — one or all four at once.

**Global Filters** apply across all enabled reports:
- **Expiring Within** — limit data to entities expiring within 7 / 30 / 60 / 90 days
- **Report Date** — defaults to today; change this for backdated reports

### Step 3 — Download

- **Excel files** — download directly to your computer
- **PDF files** — open in a new browser tab, then `Ctrl+P` / `Cmd+P` → **Save as PDF**
  - Enable **Background graphics** in the print dialog to keep colors and formatting

---

## The Four Reports

### Compliance Summary (PDF)

Executive one-page overview of compliance status.

| Section | What it shows |
|---|---|
| Program KPI Cards | Active, Compliant, Non-Compliant, New — with % of active |
| Compliance Status Breakdown | Color-coded count and % for each status |
| COI Submission Status | COI on File vs. No COI on File |
| No COI on File — Breakdown | Of no-COI entities: how many are NC, New, or Compliant |
| Expiring Coverage | Entity counts at 7 / 30 / 60 / 90 day thresholds + summary table |

---

### Compliance Detail (Excel)

Full data workbook with entity-level compliance data.

| Sheet | What it contains |
|---|---|
| Summary | Key metrics: Active, Compliant, NC, New, COI counts |
| Entity / Project / Location List | Every entity with status, expiration, and notification email |
| Expiring Coverage | Coverage expiring within 90 days, sorted soonest-first |
| NC by Coverage & Criteria | Non-compliant entities ranked by coverage type |
| Non-Compliance Reasons | Full NC reason text per entity and coverage type |

**Filter:** All / Non-Compliant Only / Compliant Only / New & Pending Only

---

### Engagement Summary (PDF)

Executive one-page overview of email engagement.

| Section | What it shows |
|---|---|
| Email KPI Cards | Sent, Opened, Clicked, Delivery Issues — with rates |
| Open Rate Breakdown | Opened vs. Not Opened |
| Deliverability Status | No Issues vs. Issues Detected |
| Delivery Issue Breakdown | Hard Bounce, Rejected, Soft Bounce, Deferral — with severity |
| 60-Day Activity Window + Funnel | Sent / Opened / Clicked in last 60 days + visual funnel |

---

### Engagement Detail (Excel)

Full engagement workbook identifying entities that need follow-up.

| Sheet | What it identifies |
|---|---|
| Summary | Emails Sent, Opened, Clicked, Issues, No COI count |
| No COIs on File | Entities with no certificate submitted |
| No Engagement Since Go-Live | Entities with zero email activity in history |
| NC + No Recent Engagement | Non-Compliant entities with no engagement in selected window |
| Undeliverable Emails | All emails with a delivery failure — type, date, subject |

**Filter:** Engagement window — 30 / **60** (default) / 90 days

---

## Refreshing Data

The app reads CSVs synced to `evident-powerbi` by a scheduled GitHub Action. For most reports, the scheduled data is current enough.

**To force a fresh sync:**

1. Open ⚙ Settings → enter a GitHub PAT with `workflow` scope
2. Click **↻ Refresh Data** in the header
3. The sync runs in `evident-powerbi` and refreshes in ~2 minutes

---

## Client Key Reference

Each client has a `key` that must match the `client` column value in your CSVs.
If a client's report is empty, verify the key below matches what's actually in your data.

| Client | Key | Structure |
|---|---|---|
| A G Equipment | `agequipment` | General |
| Canadian Pacific Kansas City | `cpkansascity` | General |
| Capital Railroad Contracting, Inc. | `capitalrailroad` | General |
| Kolb Grading | `kolbgrading` | General |
| Mizuho Bank | `mizuhobank` | General |
| Musselman & Hall Contractors, LLC | `musselmanhall` | General |
| Paragon Geophysical Services, Inc. | `paragongeophysical` | General |
| Trinity Chemical Industries LLC | `trinitychemical` | General |
| United Coal Company LLC | `unitedcoal` | General |
| BAUER Foundation Corp. | `bauer` | Subcontractors |
| ESS Companies | `esscompanies` | Subcontractors |
| Scandroli Construction | `scandroli` | Subcontractors |
| Skyline Developers Construction LLC | `skylinedevelopers` | Subcontractors |
| EMMES | `emmes` | Tenants |
| Gart Properties | `gartproperties` | Tenants |
| The Abbey Management Company | `theabbeycompany` | Tenants |

To update a key: open `index.html`, find the `CLIENTS` array near the top of the `<script>` block, and change the `key` value for the relevant client.

---

## CSV Column Reference

If your CSV column names differ from the defaults, update the `C` object near the top of `index.html`.

**`insureds.csv`**

| App field | Default CSV column |
|---|---|
| Entity Name | `display_name` |
| Contract Number | `contract_number` |
| Entity Type (General clients) | `party_type` |
| Project Name (Subcontractor clients) | `project_name` |
| Location Name (Tenant clients) | `location_name` |
| Compliance Status | `compliance_status` |
| Verification Status | `verification_status` |
| Next Expiration | `next_expiration` |
| Notification Email | `contact_email` |
| Has COI | `has_coi` |

**`requirements.csv`**

| App field | Default CSV column |
|---|---|
| Coverage Type | `coverage_type` |
| Non-Compliance Reason | `non_compliance_reason` |
| Expiration Date | `expiration_date` |
| Days Until Expiration | `days_until_expiration` |

**`notifications.csv`**

| App field | Default CSV column |
|---|---|
| Email Address | `email_address` |
| Issue Type | `issue_type` |
| Sent At | `sent_at` |
| Subject | `subject` |
| Opened | `opened` |
| Clicked | `clicked` |

---

## Troubleshooting

**"Could not load CSV" error**
Check Settings — verify repo name, branch, and file paths are correct.

**Client shows 0 entities**
The client `key` doesn't match your CSV. See the Client Key Reference above.

**Expiring Coverage or NC sheets are missing from Excel**
These require `requirements.csv`. If that file doesn't exist in the repo, the sheets are silently skipped.

**PDF colors not appearing when saved**
In the print dialog, open "More settings" and enable **Background graphics**.

**Refresh Data says "Sync failed"**
Your PAT may have expired. Create a new one at [github.com/settings/tokens](https://github.com/settings/tokens) with `workflow` scope.

**CSVs won't load (private repo)**
GitHub raw file URLs only work for public repos. Make the data repo public, or contact Philip.

---

*HUB CertSecure — Internal use only. Questions? Contact Philip Irving.*
