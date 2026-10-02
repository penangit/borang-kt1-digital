# borang-kt1-digital

Digital replica of Malaysia's **Borang KT 1** - the monthly hotel room fee collection form required by the Penang state government (Jumlah Kutipan Bulanan Fi Kerajaan Tempatan Negeri Pulau Pinang).

Built as a single HTML file that runs entirely in the browser. No server, no backend, no installation - just open the file.

![Status](https://img.shields.io/badge/status-production-green) ![License](https://img.shields.io/badge/license-MIT-blue)

## The Problem

Every hotel, hostel, and guesthouse in Penang must submit a monthly **Borang KT 1** form to the state government reporting daily room counts and Local Government Fees (LGF) collected. The current process is entirely manual:

1. Export the Add On Tax Report from Sentec PMS as an Excel file
2. Open the Excel file and manually count rooms sold per day
3. Handwrite or type those numbers into a paper/PDF form (31 rows)
4. Calculate the monthly total and fee amount by hand
5. Hope nothing was miscounted

This takes 15-30 minutes per month and is error-prone. A single miscount means the entire form needs to be redone.

## The Solution

Export the report, drag and drop it onto the form, done. The tool parses the Excel file, calculates everything automatically, and produces a print-ready form that matches the official government layout pixel-for-pixel.

**Before:** Export > manually count > handwrite > calculate > hope for the best.
**After:** Export > drag and drop > print.

## Features

### Core

- **Drag & Drop Import** - Drag the Sentec PMS Add On Tax Report (.xlsx) directly onto the form. The tool auto-parses the Excel structure, identifies the header row, and calculates daily room counts.
- **Two Form Versions** - Supports both the old format (2 months side-by-side on one A4 page) and the new format (1 month per page). Users switch between them with a toggle button in the toolbar.
- **Auto Month Detection** - Reads the English month name from the Excel file's metadata rows and maps it to the correct Bahasa Malaysia month (e.g., "September" becomes "September" in the dropdown, "March" becomes "Mac").
- **Star Rating Selection** - Clickable checkboxes for hotel star rating (1 & ke bawah, 2, 3, 4, 5) that match the official form's layout. Defaults to 3-star for Hotel Neo+ Penang.

### Validation & Cross-Check

- **Negative Value Detection** - Flags contra/reversal entries (negative Total Amount values) with the exact Excel row number, charge date, and guest name. These are highlighted in a warning panel so staff can investigate before submitting.
- **DRR Cross-Check** - A second validation layer. Import the Daily Revenue Report (DRR) to compare its "Local Govt Fee" Month-To-Date (MTD) total against the Add On Tax total. If they match, the data is confirmed correct. If they don't match, the exact discrepancy is shown in RM and number of rooms, with guidance on what to look for (missing entries vs extra contra entries).

### Output

- **Print / Save PDF** - A4-optimized print layout using CSS print media queries. All UI controls (toolbar, drop zones, buttons) are hidden. The output matches the official government form layout exactly.
- **Export to Excel** - Download the filled form data as .xlsx with the same structure as the printed form.
- **Clipboard Paste** - Paste room count data directly from Excel or Google Sheets as an alternative to file import.

### Design

- **Bilingual UI** - All messages, errors, and help text appear in both Bahasa Malaysia and English.
- **Single File** - The entire application is one HTML file. No build step, no npm, no bundler. SheetJS is loaded from CDN for Excel read/write. Everything else is vanilla HTML/CSS/JavaScript.
- **Pixel-Perfect Replica** - The form layout exactly replicates the physical Borang KT 1 so printed output is accepted by the Penang state government without question.

## How It Works

### Architecture

```
Add On Tax Report (.xlsx)              DRR (.xlsx)
from Sentec PMS                        from Sentec PMS
        |                                    |
        v                                    v
+--------------------+          +-------------------------+
|  1. Parse Excel    |          |  1. Read "Macro DRR"    |
|  2. Find header row|          |     sheet               |
|     (auto-detect)  |          |  2. Find "Local Govt    |
|  3. Extract:       |          |     Fee" row (col 2)    |
|     - Charge Date  |          |  3. Read MTD value      |
|     - Total Amount |          |     (col 8)             |
|  4. Group by date  |          +------------+------------+
|  5. Sum amounts    |                       |
|  6. Divide by 2    |                       |
|  7. = Rooms/day    |                       |
+---------+----------+                       |
          |                                  |
          v                                  v
+--------------------+          +-------------------------+
|  Fill Borang KT1   |          |  Cross-check totals     |
|  - Daily room count|<-------->|  Match    = confirmed   |
|  - Monthly total   |          |  Mismatch = show diff   |
|  - Auto-set month  |          |    in RM and rooms      |
+--------------------+          +-------------------------+
          |
          v
    Print / PDF / Excel
```

### The Calculation Formula

The Add On Tax Report from Sentec PMS records Local Government Fee charges at **RM 2.00 per room night** (for 3-star and below) or **RM 3.00 per room night** (for 4-star and above). Each row in the report represents one guest-night charge.

```
Daily rooms sold = SUM(Total Amount for that Charge Date) / RM rate per room

For a 3-star hotel (RM 2.00 rate):
  Day 1: Total Amount = RM 178.00 -> 178 / 2 = 89 rooms
  Day 2: Total Amount = RM 180.00 -> 180 / 2 = 90 rooms
  ...and so on for all 31 days

Monthly total = SUM of all daily room counts
Fee collected = Monthly total x RM rate per room
```

### Handling Negative Values (Contras/Reversals)

Negative values in the Total Amount column indicate contras or reversals:

```
Example: Total Amount = -2.00
This means: 1 room night was reversed/cancelled

The negative value is:
  - Included in the daily sum (it reduces the room count for that day)
  - Flagged in a warning panel with row number, date, and guest name
  - The warning helps staff verify these are legitimate adjustments
```

### DRR Cross-Check Logic

The Daily Revenue Report (DRR) contains an independent record of LGF collections. The cross-check compares two independently calculated totals:

```
Source 1: Add On Tax Report
  Total = SUM of all Total Amount values across the month

Source 2: DRR "Macro DRR" sheet
  Total = "Local Govt Fee" row, MTD column (column index 8)

If |Source 1 - Source 2| < 0.01:
  -> Match confirmed, data is correct
  
If mismatch:
  -> Show difference in RM
  -> Show difference in rooms (diff / 2)
  -> Indicate direction:
     - DRR higher: Add On Tax may have missing entries
     - Add On Tax higher: Add On Tax may have extra contra entries
```

### Excel File Parsing

The Add On Tax Report from Sentec PMS has a specific structure:

```
Row 1-6:  Metadata rows (hotel name, report period, etc.)
          - Row containing month name is detected for auto-month-set
Row 7:    Header row (detected by finding "Charge Date" column)
Row 8+:   Data rows with columns:
          - Charge Date (date of the room night)
          - Guest Name (for contra/reversal identification)
          - Total Amount (RM value, positive or negative)
```

The parser auto-detects the header row by scanning for a row containing "Charge Date" in any column, making it resilient to format changes in the report structure.

### Form Versions

The tool supports two official form layouts:

| | Old Format | New Format |
|---|---|---|
| **Layout** | 2 months side-by-side on one A4 page | 1 month per A4 page |
| **Title** | "Fi Kerajaan Tempatan" | "Fi Hotel" |
| **Fields** | Shared star rating, shared signatures | Has No. Lesen field |
| **Table** | Two narrow tables (left/right) | One centered table |
| **Summary** | Two summary rows (one per month) | One summary row below table |

Users toggle between formats using the Lama/Baru switch in the toolbar. Both formats support the same import logic, cross-check, and export features.

## Usage

### Quick Start

1. Download `LGF_Form_Borang_KT1.html`
2. Open in any modern browser (Chrome, Edge, Firefox)
3. Drag your Add On Tax Report (.xlsx) onto the blue drop zone
4. (Optional) Click **DRR Cross-Check** and select your DRR file to validate
5. Fill in signature details and click **Today** to set dates
6. Click **Print / Save PDF** to generate the form

### Getting the Reports from Sentec PMS

**Add On Tax Report:**
1. Log in to [pms.sentec.io](https://pms.sentec.io)
2. Go to **Reports** > **Local Tax Report** > **Add On Tax**
3. Set Start Date and End Date for the full month
4. Under Revenue Center, select **Local Government Fee** only (untick Tourism Tax)
5. Click **Generate** - the report opens as a Google Sheet
6. Download as Excel (.xlsx)

**Daily Revenue Report (DRR):**
1. Log in to [pms.sentec.io](https://pms.sentec.io)
2. Go to **Reports** > **Daily Revenue Report**
3. Select the last day of the month
4. Download as Excel (.xlsx)

## Tech Stack

- **Vanilla HTML/CSS/JavaScript** - Single file, no build step, no framework
- **[SheetJS (xlsx)](https://cdn.sheetjs.com)** - Client-side Excel read/write (loaded from CDN)
- **Browser File API** - Drag and drop file handling
- **CSS Print Media Queries** - A4 form layout that hides all interactive elements when printing

Total file size: ~55KB (excluding CDN-loaded SheetJS)

## PMS Integration Opportunity

This tool currently works as a standalone HTML file that processes exported Excel reports. However, all the logic (formula, parsing, cross-check, form generation) could be integrated directly into Sentec PMS to eliminate the export step entirely:

**Current workflow:**
```
PMS > Export Excel > Open HTML tool > Import > Print
```

**Possible integrated workflow:**
```
PMS > Generate Borang KT1 > Print
```

### What the PMS already has

- The Add On Tax data (Charge Date + Total Amount per guest-night)
- The DRR data (Local Govt Fee MTD total)
- Hotel details (name, address, star rating, license number)
- Staff details (for signature fields)

### What this tool adds

- The Borang KT 1 form layout (pixel-perfect replica of the government form, both old and new formats)
- The calculation formula: `SUM(Total Amount per Charge Date) / rate = rooms sold per day`
- Negative value detection and flagging for contra/reversal entries
- DRR cross-check logic comparing two independent data sources
- A4 print-optimized CSS layout

### Integration approach

The simplest path would be a server-side report generator within Sentec PMS that:

1. Queries the same Add On Tax data directly from the database (no Excel export needed)
2. Applies the same grouping and formula (`SUM(Total Amount) per Charge Date / rate`)
3. Runs the DRR cross-check internally
4. Renders the Borang KT 1 form as a printable HTML page or PDF
5. Pre-fills hotel details and staff signatures from the PMS profile

This would reduce the monthly process from a multi-step manual workflow to a single button click inside the PMS.

## Regulatory Context

Borang KT 1 is mandated by the Penang state government for all hotels, hostels, and guesthouses operating in Negeri Pulau Pinang. The form reports:

- **Bil. Bilik Dijual** - Daily count of rooms sold (1st to 31st of each month)
- **Jumlah Bilik Dijual** - Monthly total rooms sold
- **Kadar** - Rate per room night (RM 2 for 3-star and below, RM 3 for 4-star and above)
- **Jumlah Kutipan** - Total fee collected (rooms x rate)
- **Hotel details** - Name, address, license number, star rating
- **Signatures** - Prepared by (Disediakan oleh) and verified by (Disahkan oleh) with name, position, and date

The digital version replicates the exact layout of the physical form so the printed output is accepted by the authorities without question.

## License

MIT

---

Built for [Hotel Neo+ Penang](https://www.neohotels.com/) front office and finance teams.
