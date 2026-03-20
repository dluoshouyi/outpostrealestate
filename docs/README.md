# Outpost Pipeline Dashboard — System Documentation

## Overview

Automated system that pulls deal data from Smartsheet, generates a branded HTML dashboard, encrypts it with StatiCrypt, and publishes it to GitHub Pages. Replaces the previous manual workflow of typing weekly updates into a Word document, uploading to Claude, and hand-merging HTML files.

**Live URL:** `https://dluoshouyi.github.io/outpostrealestate/dashboard.html`
**Password:** `outpost2026!`

---

## Architecture

```
Smartsheet (5 sheets)
       |
       v
generate_dashboard.py (Python)
       |
       v
dashboard.html (raw)
       |
       v
StatiCrypt (Node.js)
       |
       v
GitHub Pages (encrypted HTML)
```

### Data Flow

1. Team updates Smartsheet during the week (Deal Updates, BD Intel, Asset Mgmt sheets)
2. When deals die, rows are moved from the Acquisitions Deal Tracker to the Dead Deals sheet with Reason Lost, Date Lost, and optionally a Reason Lost Narrative
3. The deploy script runs (manually or via Task Scheduler at 5pm daily)
4. Python pulls all 5 Smartsheet sheets via API
5. Script generates a self-contained HTML dashboard
6. StatiCrypt encrypts the HTML with a password prompt
7. Git pushes the encrypted file to GitHub Pages
8. Dashboard is live and accessible to anyone with the password

---

## File Inventory

### Core Files (in `C:\GitHub\`)

| File | Purpose |
|------|---------|
| `generate_dashboard.py` | Main generator script. Pulls from Smartsheet, produces HTML. |
| `dashboard_style.css` | All visual styling. Outpost brand system. Dark/light mode. |
| `dashboard_script.js` | Tab switching, card expand/collapse, Gantt charts, theme toggle. |
| `logo_base64.txt` | Outpost logo encoded as base64 webp for embedding. |
| `deploy.ps1` | PowerShell script that runs the full generate/encrypt/push pipeline. |

### Output (in `C:\GitHub\outpostrealestate\`)

| File | Purpose |
|------|---------|
| `dashboard.html` | The main pipeline meeting dashboard (auto-generated) |
| `development-dashboard.html` | Dev team's dashboard (pulled from SharePoint source folder) |
| Other `.html` files | Additional dashboards from the SharePoint source folder |

---

## Smartsheet Configuration

### API Token

```
jvphstFkAoCyFU0rEwJLV7ysd1Ohl9Lt552gG
```

### Sheet IDs

| Sheet | ID | Purpose |
|-------|----|---------|
| Acquisitions Deal Tracker | `7622588298645380` | Active pipeline deals |
| Deal Updates | `1037322175860612` | Weekly action items, key dates, notes per deal |
| BD & Market Intel | `8211204976627588` | Travel recaps, market intel, industry news, upcoming travel |
| Asset Management | `7793167219773316` | Portfolio site updates, hold/sell analysis |
| Dead Deals | `593819021037444` | Deals that died or were passed on |

### Column Mappings

#### Acquisitions Deal Tracker (Pipeline)

Core columns: Property Name, Deal Stage, Property Address, Seller, Team, Deal Lead, Deal Support, Development Lead, Purchase Price, Useable Acres, Total Acres, Untrended YOC, Gross Yr 3 YOC, IRR, $ / Usable Acre, $ / Total Acre, EM Deposit, PSA Signed, DD Period (Days), Go Hard Date, Closing Date.

Calculated columns (auto-populated by Smartsheet formulas): Gross YOC (numeric), Untrended YOC (numeric), IRR (numeric), PP x Gross YOC, PP x Untrended YOC, PP x IRR, PP x PerUsableAcre, PP x PerTotalAcre.

**Important notes:**
- The sheet uses section header rows (UNDER CONTRACT, NEGOTIATING PSA, etc.) that must be filtered out
- "Team" column contains values like "Southeast Team" which maps to "Central" region
- Spelling is "Useable Acres" not "Usable Acres"
- "PSA Signed" not "PSA Signed Date"
- "Deal Stage" not "Stage"

#### Dead Deals

Same columns as Pipeline, plus: Reason Lost (text), Date Lost (date), Reason Lost Narrative (text, long-form).

**Workflow:** When a deal dies, move the entire row from the Acquisitions Deal Tracker to the Dead Deals sheet. Fill in Reason Lost, Date Lost, and optionally the Reason Lost Narrative for deals that warrant a full story.

#### Deal Updates

Columns: Week Of, Region, Deal Stage, Deal Name, Type, Description, Owner, Status, Date (if Key Date).

Type values: Action Item, Key Date, Note.
Status values: Open, Complete, Removed.

**Important:** Deal names in this sheet don't always match Pipeline property names exactly. The script uses fuzzy matching (e.g., "Bender" matches "Houston - 6750 Bender").

#### BD & Market Intel

Columns: Week Of, Category, Market / Topic, Detail.

Category values: Travel Recap, Market Intel, Industry News, Upcoming Travel.

#### Asset Management

Columns: Week Of, Category, Property, Update, Owner, Status.

---

## Region Mapping Logic

The script determines a deal's region using this priority:

1. **Pipeline "Team" column** (most reliable): "Southeast Team" / "SE Team" maps to Central. "Northeast Team" / "NE Team" maps to Northeast. "West Coast" / "WC Team" maps to West Coast.
2. **Deal Updates "Region" column** via fuzzy name match: If a matching deal update has a Region value (Northeast, Central, West Coast), use it.
3. **Deal Lead inference** (fallback): Chenying/Harry = Northeast. Rob/Daniel/Tres/Clayton = Central. Nick/Jessica/Dennis = West Coast.

---

## Dashboard Structure

### Sub-tabs

1. **Pipeline & Action Items** — BD section (travel, market intel, news), KPI cards, filter bar, deal cards grouped by region with expandable action items and key dates
2. **Weekly Report** — Full 18-column tables per stage (matching the old dashboard format) with row numbers, all deal fields, weighted average totals
3. **EM Forecast** — KPI summary cards (Total Initial EM, Additional EM, Combined Exposure) plus table with confidence progress bars
4. **Asset Management** — 2-column card grid with category headers and checkbox items
5. **Dead / Passed Deals** — KPI cards, full table matching Weekly Report format with Reason Lost and Date Lost columns, expandable narrative rows for deals with Reason Lost Narrative populated

### Top-level tabs

- **Weekly Pipeline Meeting** — Contains all 5 sub-tabs above
- **Development** — Iframe embedding the dev team's separate HTML dashboard

---

## Deployment

### Manual Deployment

```powershell
& "C:\GitHub\deploy.ps1"
```

### What the Deploy Script Does

1. Runs `generate_dashboard.py` to pull from Smartsheet and create `dashboard.html`
2. Encrypts `dashboard.html` with StatiCrypt (password: `outpost2026!`, 90-day remember)
3. Pulls HTML files from SharePoint source folder (`Semi-Stow\Outpost Real Estate\0 RE Tech Tools`), including the dev team's dashboard (any file with "development" in the name gets mapped to `development-dashboard.html`)
4. Encrypts all pulled dashboards with StatiCrypt
5. Git add/commit/push to `dluoshouyi/outpostrealestate`

### Task Scheduler

Windows Task Scheduler is configured to run the deploy script daily at 5:00 PM.

- Task runs only when the user is logged on (corporate machine restricts "Run whether user is logged on or not")
- Program: `powershell.exe`
- Arguments: `-ExecutionPolicy Bypass -File "C:\GitHub\deploy.ps1"`
- The laptop must be on and awake at the scheduled time

### GitHub Pages

- Repo: `https://github.com/dluoshouyi/outpostrealestate`
- Live site: `https://dluoshouyi.github.io/outpostrealestate/`
- Git identity: `dluoshouyi@gmail.com` / `Dan Luo`

---

## Brand System

The dashboard uses the official Outpost brand identity.

### Colors

| Name | Hex | Use |
|------|-----|-----|
| Asphalt | `#001017` | Primary dark background |
| Gunmetal Blue | `#052533` | Secondary dark background |
| Concrete | `#F5F3F0` | Light mode background |
| Horizon Blue | `#8DCAE3` | Accent only (lines, active states, small highlights). Never as text on light backgrounds. |

### Typography

Font: Inter (Google Fonts). Headings uppercase, body sentence case. Labels at 10px with 0.06em letter-spacing.

### Design Rules

- Zero border-radius (everything sharp/square)
- No gradients, no shadows, no hover animations
- Dark mode default, light mode toggle in topbar
- Horizon Blue is accent only, never used as body text

---

## Dark/Light Mode

A toggle button in the topbar switches between dark and light mode. The user's preference is saved to localStorage and persists across visits.

- Dark mode: Asphalt/Gunmetal backgrounds, white text
- Light mode: Concrete/White backgrounds, black text
- Both modes are brand-compliant per Outpost identity guidelines

---

## Design Iteration (for Associates)

### Setup

A Claude Project called "Pipeline Dashboard Design" allows non-technical team members to iterate on the dashboard's visual design without touching code.

**Project contents:**
- `sample_dashboard.html` — Self-contained preview with mock data and CSS baked in
- `project_instructions.md` — Explains what they can change, brand rules, workflow

**Project prompt:**
> When the user describes a visual change, modify the CSS in the sample dashboard and render the full updated HTML as an artifact so they can see the result live. When they're satisfied, provide the updated CSS as a separate downloadable file called dashboard_style.css.

### Associate Workflow

1. Open the Claude Project, describe changes in plain English
2. Claude renders a live preview as an artifact
3. Iterate until satisfied
4. Ask Claude for the updated CSS file
5. Send the CSS file to the dashboard maintainer
6. Maintainer drops it in `C:\GitHub\dashboard_style.css` and runs deploy

No Python, Git, Node.js, or terminal access required.

---

## Development Dashboard Integration

Currently using **Option A (iframe embed)**. The main dashboard's Development tab loads the dev team's HTML file via iframe.

### Dev Team Workflow

1. Build/update their dashboard HTML
2. Save it to the shared SharePoint folder: `Outpost Real Estate > 0 RE Tech Tools` with any name containing "development"
3. At 5pm, the deploy script pulls it, renames to `development-dashboard.html`, encrypts, and pushes

### Future Path (Option B)

When the dev team moves to Smartsheet, add their sheet IDs to the script config, build `gen_development_tab()` functions, and replace the iframe with generated HTML. This gives full styling consistency, dark/light mode support, and eliminates the separate file dependency.

---

## Prerequisites

### Software (on the deploy machine)

- Python 3.x with `smartsheet-python-sdk` (`pip install smartsheet-python-sdk --break-system-packages`)
- Node.js (for `npx staticrypt`)
- Git (configured with push access to the repo)

### Execution Policy

PowerShell execution policy must be set to RemoteSigned:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Unblock-File "C:\GitHub\deploy.ps1"
```

### Folder Structure

```
C:\GitHub\
  generate_dashboard.py
  dashboard_style.css
  dashboard_script.js
  logo_base64.txt
  deploy.ps1
  outpostrealestate\       (git repo folder)
    dashboard.html         (generated + encrypted)
    development-dashboard.html
    (other dashboards)
```

The repo folder must NOT be on OneDrive/SharePoint to avoid sync corruption. Local disk only.

---

## Troubleshooting

### "No deals found" or wrong deal count

Run the column checker to verify Smartsheet column names haven't changed:

```powershell
python -c "import smartsheet; c=smartsheet.Smartsheet('jvphstFkAoCyFU0rEwJLV7ysd1Ohl9Lt552gG'); s=c.Sheets.get_sheet(7622588298645380); print([col.title for col in s.columns])"
```

### Deals in wrong region

Check the Team column value for the deal, then check the region mapping logic in `get_region()` in `generate_dashboard.py`.

### Dead deals header showing as a deal

The script filters out rows where Property Name matches known section headers (UNDER CONTRACT, DEAD DEALS, TOTAL, etc.) and rows with no address/seller/lead. If a new header name appears, add it to the `skip` set in `gen_dead_deals_tab()`.

### Task Scheduler not running

The laptop must be on and awake at the scheduled time. If missed, run manually: `& "C:\GitHub\deploy.ps1"`

### StatiCrypt errors

Ensure Node.js is installed and `npx` is in PATH. Test with: `npx staticrypt --version`

### Git push failures

Check that the git remote is configured correctly and credentials are cached:

```powershell
cd C:\GitHub\outpostrealestate
git remote -v
git push
```
