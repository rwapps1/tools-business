# Adviser Merge Sheet

A self-contained, browser-based tool that combines Main Stats, Gross Calls, and Meaningful Calls into one sheet per adviser, then produces an anonymised version of that sheet for every individual adviser.

Everything runs locally in the browser (via SheetJS and JSZip, loaded from a CDN) — no files are uploaded to a server.

## How to use

1. Open `adviser-merge-tool.html` in a browser.
2. Enter an output file name — used as the prefix for every downloaded file.
3. Upload the three sheets (`.xlsx`, `.xls`, or `.csv` all accepted):
   - **Main Stats**
   - **Gross Calls**
   - **Meaningful Calls**
4. Once all three are recognised, the status line shows how many advisers were matched, plus any warnings.
5. **Download Master Sheet (XLSX)** — one combined workbook, all advisers.
6. **Download Adviser Files (ZIP)** — one workbook per adviser.
7. **Start over** — clears everything to run again.

## Required columns

### Main Stats
- An adviser name column — detected as any header containing "adviser" (prefers one that also contains "name").
- Recognised if present: `Submitted Value`, `Net Banked`, `Gross Converted`, `Net Converted` — used for currency formatting, sorting, and the Close At First Call calculation.
- `Banked (Business)` (case-insensitive, any sheet) is deleted from the output automatically.

### Gross Calls
- `Description` — used to match rows to advisers.
- `Out` and `In` — summed into a new `Gross Calls` column.
- `Tot Tlk` — carried across as `Total Talk Time`, using the cell's displayed value rather than its raw stored number (so an Excel duration like `12:32:08` survives intact rather than becoming a decimal fraction).

### Meaningful Calls
- `Description` — used to match rows to advisers.
- `Out` and `In` — summed into a new `Meaningful Calls` column.

## Name matching

- Case-insensitive and whitespace-tolerant.
- An initial matches a full first name with the same first letter (`J Smith` ~ `John Smith`).
- A built-in list of common English nicknames is treated as equivalent (`David`/`Dave`, `Robert`/`Bob`, etc.) — not exhaustive, so unusual or regional nicknames won't be caught.
- Matching only ever links entries across *different* sheets — two rows within the same sheet are never merged into one adviser.
- An adviser missing from a sheet still appears in the output, with blanks for that sheet's data.

## Calculated columns

| Column | Formula | Notes |
|---|---|---|
| Close At First Call | (Gross Converted − Net Converted) ÷ Gross Converted | Inserted directly after Net Converted. Blank if Gross Converted is 0 or either value is missing. Shown as a percentage. |
| Gross Calls | Out + In (Gross Calls sheet) | A missing Out or In counts as 0; an adviser entirely absent from the sheet is left blank. |
| Meaningful Calls | Out + In (Meaningful Calls sheet) | Same logic as above. |
| Total Talk Time | Tot Tlk (Gross Calls sheet) | Passed through as displayed, no calculation. |

## Formatting and sorting

- `Submitted Value` and `Net Banked` are stored as real numbers and displayed as UK currency (`£#,##0.00`).
- `Close At First Call` is stored as a real fraction and displayed as a percentage (`0.00%`).
- Columns are auto-sized to fit their widest content (character-count based, capped to a sensible max).
- All rows are sorted by `Net Banked`, highest first. If that column isn't found, rows keep upload order and a warning is shown.

## Outputs

- `<output name> Master Sheet.xlsx` — every adviser, one row each.
- `<output name> Adviser Sheets.zip` — one workbook per adviser, named `<output name> <adviser name>.xlsx`. Each contains every row and all data, but every *other* adviser's name is deleted outright from the underlying cell data (not hidden, styled white, or masked).

## Warnings

Shown in the status line without stopping the process, when:
- No `Net Banked` column is found in Main Stats (rows aren't sorted).
- `Close At First Call`, `Gross Calls`, `Meaningful Calls`, or `Total Talk Time` can't be computed because a required source column is missing.
- An uploaded file has no recognisable adviser/`Description` column — that file is rejected until it's fixed.

- ## Limitations

- Nickname matching is heuristic, not exhaustive.
- Column detection relies on header text (case-insensitive); unexpected header wording won't be recognised.
- Built for one row per adviser per sheet — not designed for multiple rows per adviser within the same sheet.
-----------------------------------------------------------------------------------------------------------------------------------------------
- # Report Builder

Upload a spreadsheet, apply a saved report, done.

**Live app:** https://rwapps1.github.io/b1/

Report Builder has two modes for working with spreadsheet exports (CSV or Excel) — building repeatable, saved reports, and comparing two versions of a file to see exactly what changed.

## Build a report

Three steps: **Upload → Configure → Report**

Configuration options:

- **Filter rows** — one or more conditions, combined with AND/OR
- **Group by** — group by one or more columns, with a running count per group
- **Calculate** — aggregate numeric columns (sum, average, count, minimum, maximum) per group
- **Columns to include** — choose exactly which columns make it into the final report
- **Highlight duplicates** — flag rows where a chosen column repeats a value
- **Add columns** — append blank columns (e.g. "Notes") to fill in by hand after exporting
- **Sort** — sort by one or more columns, with tie-breaking levels (like Excel's Sort by / Then by)

**Presets** — save any configuration as a named, reusable preset. Once saved, applying it to a new file is a single click. Presets can be exported to a file (to share or back up) and imported back in.

## Compare files

Upload an **earlier** and a **later** version of the same type of file, and choose the column that uniquely identifies a row (e.g. a case reference or ID). The tool shows:

- **Added** — rows new in the later file
- **Removed** — rows present earlier but missing later
- **Changed** — same ID, different values, with the differences shown
- **Unchanged** — everything else

## Exporting

Reports can be exported as CSV or Excel (.xlsx).

## Privacy

Everything runs locally in your browser — files are never uploaded to a server. Saved presets are stored in your browser's local storage, so they're private to your device and browser, not shared or synced elsewhere.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Adviser dashboard

A single-page dashboard for tracking adviser submissions alongside net banked income, built to encourage focus on submissions as the leading indicator that drives future banked income.

Live entirely as one HTML file — no build step, no backend, no server-side storage. Data is parsed and held in the browser only; nothing is ever uploaded anywhere.

## Features

- Upload a `.csv` or `.xlsx` broker summary sheet directly in the browser
- Flexible column matching — works even if headers vary slightly month to month, as long as they contain the expected keywords (see below)
- Summary metric cards: total submissions, total submitted value, total net banked, adviser count
- A toggle between two views:
  - **Submissions** — ranks the top 12 advisers by submissions, table sorted by submitted value (highest first)
  - **Submissions & Net Banked** — ranks the top 12 advisers by net banked, table sorted by net banked (highest first)
- Sortable table — click any column header to re-sort
- Download the table as a JPEG image, ready to paste into an email

## Expected columns

The uploaded sheet should contain columns whose headers include (case-insensitive, exact naming can vary):

| Required | Keyword match |
|---|---|
| Adviser name | contains "adviser" |
| Submissions | contains "submission" (not "value") |
| Net banked | contains "net" and "banked" |

| Optional | Keyword match |
|---|---|
| Submitted value | contains "submitted" and "value" |
| Handovers | contains "handover" |

If the required columns can't be found, the page will show an error rather than guessing.

## Deploying

1. Create a new GitHub repository (public or private).
2. Add `index.html` to the root of the repository.
3. In the repo's **Settings → Pages**, set the source to your main branch, root folder.
4. The dashboard will be live at `https://<your-username>.github.io/<repo-name>/` within a minute or two.

No other setup is required — all libraries (Chart.js, PapaParse, SheetJS, html2canvas) are loaded from CDN at runtime.

## Privacy

All parsing and calculation happens client-side, in the visitor's own browser. The uploaded file is never sent to a server, and nothing is persisted between visits — refreshing the page clears the data.

## Tech

Plain HTML, CSS and JavaScript. No framework, no build tooling. Charting via Chart.js, spreadsheet parsing via PapaParse (CSV) and SheetJS (XLSX), image export via html2canvas.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
CMF Lead Conversion Dashboard

A single self-contained HTML page that tracks lead-to-application conversion from a "CMF – deal list" export. Upload a sheet, see the funnel and a monthly breakdown, filter by date range. No server, no build step — everything runs in your browser.

File: cmf-lead-conversion-dashboard.html

Using it
Open the HTML file in any modern browser (double-click it, or host it somewhere like GitHub Pages).
Click Upload sheet (or drag a file onto the button) and choose your deal list export — .xlsx, .xls, or .csv.
The funnel, the monthly table, and the date filters populate automatically.
To refresh with a new export, just upload again — it replaces the current data.

Nothing is sent anywhere. The file is read and processed entirely in your browser; closing the tab clears it.

What it expects in the sheet

The uploaded file needs a header row with these columns (matching is case-insensitive, so Deal Id and deal id both work):

Column	What it represents
Deal Id	Unique lead identifier
Created	When the lead came in
Converted	Fact find date
Specifics	Quote date
Submitted	Application date

Any other columns in the sheet (e.g. Closed At, Closed By, Close Reason, bav) are ignored.

A row counts as a lead if it has both a Deal Id and a Created date. Rows missing either (including trailing blank rows some exports leave at the bottom) are skipped automatically. A stage (Converted / Specifics / Submitted) counts as reached if it has any date in it; a blank cell, or a placeholder date of 1 Jan 2000 or earlier (seen in some exports as a "not reached" marker), counts as not reached.

How the numbers are calculated
Total leads = count of valid Deal Id rows in the selected date range.
Fact find / Quote / Application counts and percentages are always calculated against that same total — the original number of leads, not against each other.
Everything is anchored to Created. The monthly breakdown groups leads by the month they were created, and each month's percentages are against that month's own lead count. A lead created in April but converted in May is still counted as an April lead throughout.
The date filter (presets or custom range) filters on Created only, and drives both the funnel and the monthly table together.
Hosting it

It's one HTML file with no dependencies to install, so you can:

Open it directly from disk, or
Push it to a GitHub repo and serve it with GitHub Pages, or
Drop it on any static file host.

It loads two small libraries from a CDN (for reading spreadsheets, and for fonts) — an internet connection is needed the first time a page loads them, but no data ever leaves your browser.
