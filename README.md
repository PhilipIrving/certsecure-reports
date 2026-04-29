# HUB CertSecure — Report Generator

A single-file web application for generating compliance and engagement reports across all HUB CertSecure clients. Built and maintained by Philip Irving, Account Manager, HUB CertSecure (HUB International).

---

## Access

The app is hosted on GitHub Pages:

```
https://philipirving.github.io/certsecure-reports/
```

> A custom domain can be configured — contact your administrator.

---

## Authentication

Login is handled via **Supabase**. Each user has a display name and one of three roles:

| Role | Access |
|---|---|
| **Account Administrator** | Generate all reports, view client configuration (read-only) |
| **Account Manager** | Everything above + edit client settings (go-live dates, structure, active status) |
| **System Administrator** | Full access — includes adding/deleting clients, managing users, and application settings |

To reset your password, use the **Forgot Password** link on the login screen.

To update your role, a System Administrator must edit your user in **Settings → Users**, or you can update your metadata directly in the Supabase dashboard.

---

## Application Structure

The app has two main tabs:

### Report Generator
Three-step workflow:
1. **Select a client** from the grid (General / Subcontractors / Tenants)
2. **Configure reports** — select which reports to generate and set options
3. **Generate** — files download automatically and appear in Recent Activity

The **↻ Refresh Data** button (center of page) triggers a data sync from the Evident platform.  
The **🏢 HUB-Wide Annual** button generates an across-all-clients annual review.

### Client Configuration
Manage the client roster. Changes write directly to `config/clients.json` in the repository.

- **Account Managers** can edit existing clients (go-live date, structure, active status)
- **System Administrators** can also add and delete clients
- Inactive clients are hidden from the report picker but preserved in the config

---

## Reports

All seven reports are generated **client-side in the browser** using live data from the repository. No server processing is required.

### Client Reports (require client selection)

| Report | Type | Data Source | Notes |
|---|---|---|---|
| Compliance Summary | PDF | `data/` CSVs | Executive overview — KPIs, status breakdown, NC reasons, expiring coverage |
| Compliance Detail | Excel | `data/` CSVs | Entity-level data across 5 sheets |
| Engagement Summary | PDF | `data/` CSVs | Email activity for selected window (30/60/90 days) |
| Engagement Detail | Excel | `data/` CSVs | 5 sheets — No COI, No Engagement, NC+No Engagement, Undeliverables |
| Year End Review | PDF | `snapshots/YYYY/` | Full-year narrative — trend, engagement, expirations, outstanding NC |
| Producer Overview | PDF | `snapshots/YYYY/` | External-facing one-pager for producers |

### Program-Wide Reports (no client required)

| Report | Type | Data Source | Notes |
|---|---|---|---|
| HUB-Wide Annual | PDF | `snapshots/YYYY/` | All clients combined — totals, trend, most improved, common NC reasons |

### PDF Output
PDFs open the browser print dialog automatically. Select **Save as PDF** as the destination. All generated files also appear in the **🔔 Recent Activity** drawer for re-download without regenerating.

---

## Data Sources

All data is synced nightly from the Evident platform and stored in the `evident-sync` repository.

```
evident-sync/
├── config/
│   └── clients.json          ← Client roster, go-live dates, structure
├── data/
│   ├── insureds.csv
│   ├── coverages.csv
│   ├── criteria.csv
│   ├── custom_properties.csv
│   └── engagement.csv
├── snapshots/
│   └── YYYY/
│       └── YYYY-MM/
│           ├── insureds.csv
│           ├── coverages.csv
│           ├── criteria.csv
│           ├── custom_properties.csv
│           ├── engagement.csv
│           ├── summaries.csv
│           └── sync_metadata.csv
├── reports/                  ← Generated report output (server-side)
└── src/
    ├── generate_reports.py
    ├── build_compliance_summary.py
    ├── build_compliance_detail.py
    ├── build_engagement_summary.py
    ├── build_engagement_detail.py
    ├── build_yearend_client.py
    ├── build_producer_overview.py
    └── build_yearend_hubwide.py
```

### CSV Column Reference

**insureds.csv** — `client`, `insured_id`, `insured_name`, `primary_contact_email`, `compliance_status`, `verification_status`, `next_expiration`, `active`, `paused`

**coverages.csv** — `client`, `insured_id`, `insured_name`, `coverage_type`, `expiration_date`

**criteria.csv** — `client`, `insured_id`, `insured_name`, `primary_contact_email`, `overall_compliance`, `non_compliance_reasons` *(pipe-delimited)*

**custom_properties.csv** — `client`, `insured_id`, `field_name`, `field_value`

**engagement.csv** — `Event ID`, `Date`, `Email`, `Type`, `Subject`, `Client`

---

## GitHub Actions Workflows

| Workflow | File | Trigger | Purpose |
|---|---|---|---|
| Sync | `sync.yml` | Manual (Refresh Data button) or scheduled | Pulls latest data from Evident API |
| Generate Reports | `generate-reports.yml` | Manual | Server-side report generation (all clients in parallel) |
| Monthly Snapshot | `monthly-snapshot.yml` | Scheduled (monthly) | Captures a point-in-time snapshot |

---

## Settings

Accessible via the ⚙ gear icon in the header *(System Administrators only)*.

| Setting | Description |
|---|---|
| Data Owner | Repository owner username (e.g. `PhilipIrving`) |
| Repository | Repository name (e.g. `evident-sync`) |
| Branch | Branch to read from (default: `main`) |
| Access Token | Personal access token with `repo` and `workflow` scopes |
| CSV Paths | Configurable paths to each data file |
| Admin — Edge Function URL | Supabase Edge Function URL for user management |

Settings are stored in `localStorage` — they persist across sessions on the same browser but must be re-entered on a new device.

---

## User Management

System Administrators can manage users in **Settings → Users**:
- Add users with display name and role
- Edit display name and role
- Reset passwords
- Remove users

User data is stored in **Supabase Auth** (`https://nwcyavfkmbhcadtrvyqy.supabase.co`).

The Edge Function `manage-users` must be deployed to Supabase for user management to work. See `manage-users-edge-function.ts` for the latest version.

---

## Client Configuration Reference

Clients are defined in `config/clients.json`. Each entry:

```json
{
  "client_name": "Canadian Pacific Kansas City",
  "rp_common_name": "cpkansascity",
  "program_start_date": "2025-05-01",
  "go_live_date": "2025-11-05",
  "structure": "general",
  "active": true
}
```

| Field | Description |
|---|---|
| `client_name` | Must exactly match the `client` column in all CSVs — do not change after go-live |
| `rp_common_name` | API name used for platform access and logo files |
| `program_start_date` | Date the client joined the program |
| `go_live_date` | Date the platform went live for the client — used for engagement reporting |
| `structure` | `general` (vendors), `project` (subcontractors), or `location` (tenants) |
| `active` | `true` = visible in report picker · `false` = hidden but preserved |

---

## Roles & Permissions

| Permission | Account Administrator | Account Manager | System Administrator |
|---|---|---|---|
| Generate all reports | ✓ | ✓ | ✓ |
| View client configuration | ✓ | ✓ | ✓ |
| Edit client settings | ✗ | ✓ | ✓ |
| Add / delete clients | ✗ | ✗ | ✓ |
| Manage users | ✗ | ✗ | ✓ |
| Access settings | ✗ | ✗ | ✓ |

---

## Version History

| Version | Summary |
|---|---|
| v0.08 | Refresh Data moved to page header · GitHub branding removed from UI · Two-tab layout (Report Generator + Client Configuration) · Dynamic client loading from clients.json · Three-tier role system · Display name + role badge in header |
| v0.07 | Role system (Account Administrator / Account Manager / System Administrator) · Client Configuration tab with write-back to clients.json |
| v0.06 | Dynamic client loading · go_live_date wired to engagement reports |
| v0.05 | Two-tab layout added |
| v0.04 | HUB-Wide Annual · Year End Review · Producer Overview · all reports client-side · PDF iframe print |
| v0.03 | Notification drawer with re-download · Refresh Data workflow fix |
| v0.02 | Auth restored · version on login screen · clients visible |
| v0.01 | Initial versioned release |

---

*HUB CertSecure — Serviced by HUB International · Confidential*
