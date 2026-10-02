# borang-kt1-digital

Digital replica of Malaysia's **Borang KT 1** — the monthly hotel room fee collection form required by the Penang state government (Jumlah Kutipan Bulanan Fi Kerajaan Tempatan Negeri Pulau Pinang).

Built as a single HTML file that runs entirely in the browser. No server, no backend, no installation — just open the file.

![Status](https://img.shields.io/badge/status-production-green) ![License](https://img.shields.io/badge/license-MIT-blue)

## What It Does

Hotel staff in Penang must submit a monthly form (Borang KT 1) reporting daily room counts and fees collected. This tool automates the painful part — manually counting rooms from the PMS export and filling in 31 rows by hand.

**Before:** Export Add On Tax Report from Sentec PMS → open Excel → manually count rooms per day → handwrite or type into a paper/PDF form → hope you didn't miscount.

**After:** Export → drag and drop → done. Print or save as PDF.

## Features

- **Drag & Drop Import** — Drag the Sentec PMS Add On Tax Report (.xlsx) onto either table. Auto-calculates daily room counts using `SUM(Total Amount) ÷ 2` per Charge Date.
- **Auto Month Detection** — Reads the English month from the Excel metadata and maps it to the correct Bahasa Malaysia month in the dropdown.
- **Negative Value Detection** — Flags contra/reversal entries (negative Total Amount) with the exact row number, date, and guest name so staff can investigate.
- **DRR Cross-Check** — Import the Daily Revenue Report to compare its "Local Govt Fee" MTD total against the Add On Tax total. Confirms whether the data is correct or highlights discrepancies.
- **Dual Table Layout** — Two side-by-side monthly tables on one A4 page, matching the original government form layout.
- **Export to Excel** — Download the filled form data as .xlsx.
- **Print / Save PDF** — A4-optimized print layout that hides all UI controls.
- **Zero Dependencies** — Single HTML file. SheetJS loaded from CDN for Excel read/write. Nothing else.

## How It Works

```
Add On Tax Report (.xlsx)          DRR (.xlsx)
from Sentec PMS                    from Sentec PMS
        │                                │
        ▼                                ▼
┌──────────────────┐          ┌─────────────────────┐
│  Parse Excel     │          │  Read "Macro DRR"   │
│  Find header row │          │  sheet, find "Local  │
│  Sum Total Amount│          │  Govt Fee" row,     │
│  per Charge Date │          │  get MTD value      │
│  Divide by 2     │          └─────────┬───────────┘
│  = Rooms per day │                    │
└────────┬─────────┘                    │
         │                              │
         ▼                              ▼
┌──────────────────┐          ┌─────────────────────┐
│  Fill Borang KT1 │          │  Cross-check total  │
│  daily room count│◄────────►│  Match = ✅ correct  │
│  + auto-set month│          │  Mismatch = ❌ error │
└──────────────────┘          └─────────────────────┘
         │
         ▼
    Print / PDF / Excel
```

### Calculation

The Add On Tax Report from Sentec PMS charges RM 2.00 per room night as the Local Government Fee. Each row = one guest-night charge.

```
Daily rooms sold = SUM(Total Amount for that Charge Date) ÷ 2
```

Negative values in Total Amount indicate contras or reversals (e.g., `-2.00` = one room reversed). These are flagged but still included in the calculation since they represent legitimate adjustments.

## Usage

### Quick Start

1. Download `LGF_Form_Borang_KT1.html`
2. Open in any browser
3. Drag your Add On Tax Report onto the blue drop zone
4. (Optional) Click **🔍 DRR Cross-Check** and select your DRR file
5. Click **🖨️ Print / Save PDF** to generate the form

### Getting the Reports from Sentec PMS

**Add On Tax Report:**
1. Log in to [pms.sentec.io](https://pms.sentec.io)
2. Go to **Reports** → **Local Tax Report** → **Add On Tax**
3. Set Start Date and End Date for the full month
4. Under Revenue Center, select **Local Government Fee** only (untick Tourism Tax)
5. Click **Generate** → download as Excel (.xlsx)

**Daily Revenue Report (DRR):**
1. Log in to [pms.sentec.io](https://pms.sentec.io)
2. Go to **Reports** → **Daily Revenue Report**
3. Select the last day of the month
4. Download as Excel (.xlsx)

## Tech Stack

- Vanilla HTML/CSS/JavaScript — single file, no build step
- [SheetJS (xlsx)](https://cdn.sheetjs.com) — client-side Excel read/write
- Browser File API — drag and drop
- CSS print media queries — A4 form layout

## Context

This form is mandated by the Penang state government for all hotels, hostels, and guesthouses. It reports:

- Daily count of rooms sold (**Bil. Bilik Dijual**)
- Monthly total and fee calculation
- Hotel details and star rating
- Authorized signatures and official stamp

The digital version replicates the exact layout of the physical Borang KT 1 form so the printed output is accepted by the authorities.

## License

MIT

---

Built for [Hotel Neo+ Penang](https://www.neohotels.com/) front office and finance teams.
