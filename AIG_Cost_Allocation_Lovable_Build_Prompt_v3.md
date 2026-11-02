# COMMAND PROMPT — "AIG Ledger" v3.0: Quarterly Cost Allocation Platform with Company Dossiers (Lovable.dev build)

**Ref:** AIG/FIN/CA-APP/2026-03 · **Supersedes:** AIG/FIN/CA-APP/2026-02 · **Version:** 3.0 (30-Oct-2026) **Owner:** Accounts & Finance Department, Almoosa Investment Group L.L.C — Part of Salem Ahmad Almoosa Enterprises **Reviewer:** Group Deputy Chief Accountant

> **What changed from v2.0**
> 
> 1. **ERP / ledger reconciliation removed entirely.** No register import, no bridge, no match statuses, no voucher export. The app is the source of the quarter figures and documents; nothing is compared with an outside ledger.
> 2. **Each quarter is presented as an audit file.** It has a lead schedule, working-paper references, auditor tick-marks, controls and a sign-off. It is not just a table.
> 3. **New Company Dossier.** Click any company to view its quarter document and schedule, print it, or export it to PDF or Excel. Single company, selected companies or all, as required.
> 4. **New Print Studio** for batch packs. Plus a design system locked in a `DESIGN.md`, and a skills map that tells Lovable which skill governs which phase.

---

## PART A — HOW TO RUN THIS PROMPT

Lovable builds best in phases. A single giant message produces a wide but shallow app with untested maths.

| Step | Where | What to paste |
| --- | --- | --- |
| 1 | Lovable → Project → **Knowledge** | Part B in full (standing rules, re-read on every message) |
| 2 | Lovable → Workspace → **Skills** | The skills in Part F (create once, reuse in every project) |
| 3 | Chat, message 1 | Part C + *"Acknowledge. Do not build yet. Reply with architecture, schema with RLS, phase plan mapped to Part D, and conflicts or risks. Wait for approval."* (use **plan mode**) |
| 4 | Chat, one message per phase | Part D, Phase 0 → Phase 10, **one at a time**. Start the next phase only after the phase's acceptance tests show PASS. |
| 5 | After Phase 4 and Phase 10 | Part E golden-figure pack with the validated quarter workbooks attached |
| 6 | Any time Lovable drifts | Part G do-not list |

**Before you start:** enable Lovable Cloud (database, auth, storage). This is group financial and related-party data on a third-party platform. Get IT/management approval, keep the project **private**, and turn on MFA for every user.

Items marked **⚠** are yours to edit each run.

---

## PART B — PROJECT KNOWLEDGE (paste into Lovable → Knowledge)

```
PROJECT: AIG Ledger — quarterly shared-services cost allocation platform for
Almoosa Investment Group L.L.C (AIG), Dubai, UAE. AIG TRN 100202501100003.
AIG issues every allocation document. ~52 related recipient companies.
There is NO external ledger/ERP reconciliation in this app. Do not build one.

ACCOUNTING RULES (non-negotiable)
1. Money = INTEGER FILS (BIGINT). AED 1.00 = 100 fils. No floating-point money.
   decimal.js for any rate/percentage, then ROUND_HALF_UP to fils.
2. Round each department line to fils first; document net = sum of rounded
   lines; VAT = ROUND_HALF_UP(net x rate) on the document net. VAT rate lives
   in ONE effective-dated settings row (default 5%).
3. Document type by the RECIPIENT's VAT status at the document date:
   - AIG VAT Tax Group member -> "Debit Note - Tax Group Member", VAT 0
     (supply disregarded, UAE VAT Decree-Law No. 8/2017, Art. 14)
   - Not a member             -> "Tax Invoice", VAT 5%
   - Net negative             -> "Credit Note" (same tax treatment as the
     company's status; group member credit note carries no VAT)
   - Negative department line inside a positive document -> separate
     "Adjustment" line, never netted away silently.
4. Document date = period end (30-Sep-2026 for Q3).
5. IFRS accrual basis. IAS 24 related-party disclosure. UAE CT 9% (Decree-Law
   No. 47/2022); keep method/basis evidence per line for Art. 34 arm's length.
6. POSTED documents are IMMUTABLE. Correct only by Credit Note referencing the
   original. No hard deletes of anything posted.
7. Document numbers gapless per series per year, assigned by the database at
   posting, never by the client.
8. Closed period = locked. Reopen needs reason + Reviewer approval, logged.
9. Maker-checker: the preparer of a batch can never approve it.
10. Recompute every total from raw lines. Never trust a stored or displayed
    total. Variances are disclosed, never plugged or suppressed.
11. Management decisions are DATA (register MD-YYYY-nnn), never code. Each
    applied decision shows as a separately disclosed reconciling item.
12. Every report and export with checks ends in a banner:
    "ALL CONTROLS PASSED" (green) or "ERROR FOUND" (red) + failing checks.

ENGINEERING RULES
- Stack: React + TypeScript strict + Vite + Tailwind CSS v4 (CSS-first @theme
  tokens) + shadcn/ui; Lovable Cloud (Supabase Postgres, Auth, RLS, Storage,
  Edge Functions); TanStack Query/Table/Virtual; decimal.js; Zod; React Hook
  Form; motion (Framer Motion); React Three Fiber + drei (lazy, decorative
  only); ECharts or Recharts; cmdk; Fuse.js; ExcelJS (export); SheetJS from
  cdn.sheetjs.com (import); @react-pdf/renderer (documents); pdf-lib (merge).
- ONE allocation engine module (src/engine/) used by preview, posting,
  dossiers, reports, exports and tests. No duplicated calculation logic.
- RLS ON for every table. Posting, numbering, close and reversal run in
  Postgres functions / Edge Functions, never client-only.
- No mock data in production views. No hardcoded companies, rates,
  departments or decisions in code — all from master tables.
- Every phase ships with tests; engine coverage >= 95%. Show test output
  before declaring a phase done. Maintain /admin/self-test (in-browser
  engine test run, PASS/FAIL per test).
- Zero TypeScript, lint, console and runtime errors at handover. No
  placeholder text, "TODO", lorem ipsum, dead buttons or "coming soon".
- DESIGN.md at repo root is the design source of truth; read it before any
  UI change and keep it valid.

EXPORT HOUSE STYLE
- Excel: classic functions only — SUM, SUBTOTAL, SUMIFS, SUMPRODUCT, IF,
  ROUND, INDEX/MATCH, COUNTIF, TEXT. NEVER XLOOKUP, FILTER, SORT, UNIQUE,
  LET, LAMBDA, MINIFS (must recalculate in LibreOffice).
- Font colours: blue = input, black = formula, green = cross-sheet link,
  yellow fill = edit/review required. Legend on every sheet. VAT rate in one
  yellow named cell. Frozen headers, AutoFilter, A4, print titles, fit to
  width. Hardcoded figures carry a source note.
- Paper (Excel/PDF): navy #1B2A4A, dark grey, white, light grey, gold
  #C9A227 accent only. No gradients, shadows or clipart on paper.
- File names: AIG_{DocType}_{DocNo}_{CompanyShort}_{Period}.pdf / .xlsx
```

---

## PART C — MASTER SPECIFICATION (message 1, plan mode)

### C1. Mission

Replace the quarterly Excel cost-allocation workbooks and the separate invoice generator with one platform. The Accounts team will use it to:

- maintain masters;
- compute allocations;
- apply management decisions;
- issue Tax Invoices, Debit Notes and Credit Notes;
- close quarters;
- present every quarter as an audit-ready file.

For any company, the user can view, print or export (PDF or Excel) its quarter document and supporting schedule in two clicks.

Quality bar: every screen and every number must withstand review by a Big 4 audit partner, the FTA and the Group Deputy Chief Accountant.

### C2. Roles

| Role | Can | Cannot |
| --- | --- | --- |
| Admin | Users, roles, settings, masters, number series | Approve own batches |
| Preparer | Import, edit drafts, compute, propose decisions, prepare batches, print drafts (watermarked) | Approve, post, close |
| Reviewer (Group Deputy Chief Accountant) | Review, approve/reject with comments, post, close/reopen, approve decisions | Edit a batch they prepared |
| Viewer (management) | Read, print and export everything | Change anything |
| Auditor (time-limited) | Read everything incl. audit log, evidence; add tick-mark notes | Change figures |

Auth: email + password + MFA; 30-minute idle timeout; every login written to the audit log.

### C3. Domain model (minimum tables — Lovable may add, never remove)

| Table | Key fields |
| --- | --- |
| `entities` | id, legal_name, short_name, aliases\[\], trade_licence_no, emirate, address, TRN, status (Active / In liquidation / Liquidated / Not in group), status_effective_date, relationship (subsidiary / sister / individual related party), permitted_departments_in_liquidation\[\], notes |
| `entity_vat_status` | entity_id, is_tax_group_member, effective_from, effective_to, evidence |
| `entity_aliases` | entity_id, alias, source, evidence, approved_by |
| `departments` | id, code, name, display_order, allocation_method, active |
| `periods` | id, type, code (2026-Q3), start, end, status (Open / In review / Closed / Reopened) |
| `allocation_policies` | department_id, version, effective_from/to, method (Fixed fee / Driver-based / Pass-through / Cost + mark-up %), driver, rate, markup_pct, narration_template, policy_text |
| `cost_pools` / `drivers` | period_id, department_id, total_cost, evidence / period_id, entity_id, driver_code, quantity, evidence |
| `allocation_lines` | period_id, entity_id, department_id, basis_text, quantity, rate, amount_fils, line_kind (Charge / Adjustment / Mark-up), source (Imported / Computed / Manual override), source_ref (file, sheet, cell), wp_ref, status |
| `management_decisions` | ref MD-YYYY-nnn, period_id, entity_id, department_id?, type (Exclude / Override / Carry forward / Force document type / Not issued / Liquidated), amount_fils, reason, authority, decision_date, attachment, status |
| `batches` | period_id, prepared_by, reviewed_by, status, comments, engine_hash |
| `documents` | series, number, type (TI / DN / CN), entity_id, period_id, doc_date, net_fils, vat_fils, total_fils, vat_rate, narration, original_document_id, status (Draft / Posted / Reversed), posted_at/by, pdf_path, xlsx_path, content_hash |
| `document_lines` | document_id, department_id, line_kind, description, basis, net_fils, vat_fils |
| `tickmarks` | owner (line / total / document), symbol, meaning, by, at |
| `exceptions` | rule_code, period_id, entity_id, severity, message, status (Open / Explained / Resolved), explanation |
| `print_jobs` | requested_by, scope (one / selected / all / filter), contents\[\], format (PDF / Excel / ZIP), status, file_path, sha256 |
| `attachments` | owner, path, sha256, uploaded_by |
| `audit_log` | append-only, before/after JSON, reason, prev_hash, row_hash (hash chain) |
| `settings` | VAT rate (effective-dated), issuer details, logo, signatories, number series, tolerances, thresholds |

Departments are seeded from master data, not code: Accounts, Tax, Legal, IT, Vehicle, Insurance, HR, Procurement, Admin, Transport. Admin can add more.

### C4. Allocation engine (`src/engine/`)

The engine is pure TypeScript with no UI imports.

1. **Inputs:** period; companies with status and VAT status as at period end; allocation lines; approved decisions; VAT rate.
2. **Methods:**
   - Fixed fee.
   - Driver-based, using largest-remainder rounding so lines sum exactly to the pool. The fils moved are disclosed.
   - Pass-through, with optional cost + mark-up, shown as a separate Mark-up line. The narration already states "inclusive of 5% mark-up" for IT and Vehicle direct expenses.
   - Imported summary amount. The quarter Summary is final.
3. **Decision layer:** applied in a fixed order: status → exclusions → overrides → carry-forwards → forced type. Each application creates a reconciling item.
4. **Document layer:** per company: lines → net → type by VAT status → VAT → total. Negative department lines stay as Adjustment lines. A negative net becomes a Credit Note.
5. **Narration builder:** built from the policy template using real charged figures. It matches the existing wording, e.g. *"Debit Note Issued from AIG to {company} towards cost allocation of shared service charges for the 3rd Quarter (Q3) - July to September 2026, pertaining to {departments}, aggregating to AED {net}."*
6. **Outputs:** documents, department totals, gross / excluded / net / VAT / total, reconciling items, exceptions, and a deterministic `engine_hash`. Posting re-runs the engine on the server and refuses if the hash differs.
7. **Invariants (throw, never warn):**
   - Σ lines = Σ document net + Σ excluded.
   - net + VAT = total on every document.
   - VAT = 0 on every Debit Note and on group-member Credit Notes.
   - Every line maps to exactly one company.
   - No company has two documents of one type in one period unless one is a reversal.
   - Driver pools sum exactly.

### C5. Modules and screens

**1. Command Centre (home).**

- Period switcher: Q1 · Q2 · Q3 · Q4 · YTD.
- KPI tiles: gross, excluded, net, VAT, total to issue, documents posted vs expected, open exceptions.
- Quarter close ring and controls banner.
- "Changed since your last visit" feed.
- A decorative 3D hero, on wide screens only.

**2. Master data.**

- Companies: card and table views; status timeline; VAT-group history; aliases; permitted departments while in liquidation; documents; evidence.
- Departments; versioned policies with a diff view; signatories; number series.
- Bulk edit with a preview and undo before save. Every change needs a reason and is logged.

**3. Import Studio (quarter workbooks only).**

- Drag and drop `.xlsx`. Auto-detects the legacy Q1 layout (SUMMARY + department tabs) and the standard layout (Cover, Summary, Consolidation, Controls, Policy, Setup + department tabs).
- Flow: Detect → Map columns → Map companies (Fuse.js score, manual override with evidence, alias saved) → Validate → Diff → Commit as Draft.
- Shows:
  - department tab vs Summary per department;
  - amounts that never reach the Summary (proposed as management exclusions);
  - text-stored numbers and merged headers;
  - cells with more than 2 decimals (the count sets the tolerance, ±0.005 × n);
  - serial dates, converted.
- A SHA-256 duplicate-file block stops the same file being imported twice. Every line keeps its source file / sheet / cell.

**4. Quarter Workspace — the Excel replacement.**

- Virtualised company × department grid.
- Excel behaviour: in-cell edit; Tab/Enter navigation; paste ranges from Excel; fill-down; undo/redo; multi-select with live Σ / count / average.
- Filters:
  - an Excel-style ▾ filter on every column, with a value tick-list and search;
  - number rules (>, ≥, <, ≤, =, between, non-zero, zero, positive, negative);
  - active-filter chips with ×;
  - a sticky totals row computed on **visible rows only** (SUBTOTAL behaviour).
- Each cell shows its provenance on hover. Manual overrides are yellow and need a reason.
- Workflow: Draft → Submit → Approve/Reject (comments per cell) → Post.
- **Clicking a company name opens its Company Dossier (module 5).**

**5. Company Dossier — the core of v3.** Route `/q/{period}/c/{company}`. It is shareable and printable.

- **Header:**
  - legal name, TRN, tax status chip, company status chip;
  - a liquidation banner with permitted departments, when relevant;
  - quarter switcher (Q1 / Q2 / Q3 / Q4 / YTD);
  - previous/next company (keys J / K).
- **Live A4 preview** of the quarter document — Tax Invoice, Debit Note or Credit Note — exactly as it will print. Drafts carry a diagonal "DRAFT — NOT POSTED" watermark.
- **Supporting schedule:**
  - department lines with method, basis, policy version and working-paper ref;
  - net, VAT, total;
  - Adjustment and Mark-up lines shown separately;
  - decisions applied, as reconciling items with MD links;
  - excluded amounts with reasons.
- **Quarter history:** Q1–Q4 net / VAT / total with document number and type, a sparkline, and movement vs the prior quarter with an explanation field.
- **Department mix chart** for the company.
- **Checks block:**
  - net + VAT = total;
  - VAT correct for status;
  - lines tie to the document;
  - TRN present where required;
  - status consistent;
  - ending in a PASS/FAIL banner.
- **Action bar** (sticky; also in ⌘K and on keys):
  - **View** — full-screen document;
  - **Print** (P) — browser print with A4 print CSS: document page + schedule page + checks;
  - **PDF** (Shift+P) — @react-pdf/renderer;
  - **Excel** (E) — the per-company workbook, see C6;
  - **Pack** — PDF + Excel in a ZIP;
  - **Copy narration**;
  - **Email draft** — generates the text and attachments only, never sends.
- **Print options sheet**, saved per user:
  - contents (document only / document + schedule / full pack with history and checks);
  - copies; original / copy marking;
  - bilingual English/Arabic headings;
  - include signatures / QR.

**6. Print Studio (batch).**

- Choose scope: one company, a ticked selection, all companies with a document, or the current grid filter.
- Choose contents and format:
  - one merged PDF, sorted by document number, with a cover and index page;
  - one PDF per company in a ZIP;
  - one Excel workbook per company in a ZIP;
  - one consolidated Excel quarter pack.
- Companies with no document (excluded or liquidated) are skipped and listed on the cover with the reason.
- Runs as a `print_jobs` record with progress and a file SHA-256. The result is downloadable again later.

**7. Documents.**

- Register of all Tax Invoices, Debit Notes and Credit Notes, with filters and visible-row totals.
- **Tax Invoice** content per UAE VAT Executive Regulation **Art. 59**:
  - "Tax Invoice" heading;
  - issuer name / address / TRN;
  - recipient name / address / TRN where registered;
  - sequential number, issue date and date of supply;
  - description, net per line, VAT rate and amount, total in AED.
- **Credit Note** per **Art. 60**, referencing the original document.
- **Debit Note — Tax Group Member** statement: "Supply between members of the same VAT Tax Group — disregarded for VAT purposes (Art. 14)."
- Signatory block, and a QR code that opens the document's in-app verification page (internal only).
- Posted documents are read-only. "Reverse" creates a linked Credit Note.
- **E-invoicing readiness:** store the UAE PINT AE (Peppol) data fields and offer a JSON/CSV field-map export per document, for the Accredited Service Provider later. Do not transmit.

**8. Quarter Audit File** — each quarter presented like a working-paper file.

- **Index (A–H):**
  - A Lead schedule;
  - B Company × department matrix;
  - C Department schedules;
  - D Documents register;
  - E Management decisions & reconciling items;
  - F VAT support;
  - G Related parties & transfer-pricing support;
  - H Controls & sign-off.
- **Lead schedule:** gross → less excluded (by decision) → net → VAT → total. Each figure carries a W/P ref and drills to its working paper.
- **Auditor tick-marks**, with a legend on every page:
  - ✓ agreed to source workbook cell;
  - Σ footed;
  - ⊕ cross-footed;
  - Ⓡ recomputed by engine;
  - Ⓓ agreed to management decision;
  - Ⓥ VAT recomputed. The engine applies them automatically when its check passes. Auditors may add notes.
- **Prepared / Reviewed / Approved sign-off** with name and timestamp. After approval the file shows a "CLOSED" stamp.
- **Export:**
  - the whole file as one indexed PDF (bookmarks per section);
  - or a multi-sheet Excel (see C6).

**9. Reports** — all filterable, with drill-down, and exportable to Excel and PDF.

| # | Report | Purpose |
| --- | --- | --- |
| R1 | Quarter Audit File | Module 8 |
| R2 | Executive Summary | Rows = companies, columns = departments, last column = company total, last row = department total, bottom-right = grand total |
| R3 | Company statement | All periods, documents, decisions, YTD |
| R4 | Department report | Pool vs allocated, recipients, method, policy version |
| R5 | Document register | Type, number, status, totals |
| R6 | VAT support | Per quarter: Tax Invoices count / taxable value / output VAT **by recipient emirate** (VAT 201 Box 1 support); Debit Notes (disregarded); Credit Notes; excluded |
| R7 | Related-party schedule (IAS 24) | Relationship, nature, amounts |
| R8 | Transfer-pricing support (CT Art. 34) | Method, driver, cost base, mark-up, evidence |
| R9 | YTD summary | Department × quarter, movement, share %, top 10 |
| R10 | Variance analysis | Any two periods, by company and department, % and AED thresholds |
| R11 | Exceptions | Rule-based (C7) with status |
| R12 | Decision log | MD register with effect per period |
| R13 | Audit trail | Log, hash-chain verification |
| R14 | Close certificate | Checklist, controls, sign-offs |
| R15 | Board pack (PDF) | Cover, KPIs, YTD matrix, charts, exceptions, banner, sign-off |
| R16 | Report builder | Pick dimensions + measures → pivot + chart → save and share |

**10. Period Close.**

- Checklist: imports validated; exceptions explained; decisions approved; documents posted; controls PASS; audit file signed.
- Reviewer sign-off locks the period.
- Reopen needs a reason and approval, and is logged.
- Approved carry-forwards (e.g. MD-2026-005) appear automatically in the next quarter.

**11. Audit & Evidence.**

- Log viewer with before/after diff and a hash-verify button.
- Evidence library with SHA-256 for every file.
- "Explain this number" on any figure: total → lines → source cell or decision.

**12. Admin.** Users, settings, VAT history, number series, backup (full JSON + Excel), self-test.

### C6. Excel exports (ExcelJS, house style from Part B)

**Per-company workbook** — `AIG_{Type}_{No}_{Company}_{Period}.xlsx`:

1. **Document** — A4 print-ready replica of the PDF. Totals are formulas linked to Workings.
2. **Workings** — the department lines:
   - input amounts in blue;
   - `ROUND` net per line;
   - VAT = `ROUND(net × VATRate, 2)` using the yellow named cell;
   - Adjustment and Mark-up lines labelled;
   - a source note per hardcoded figure.
3. **History** — Q1–Q4 with green links for the current quarter.
4. **Checks** — net + VAT = total; Σ lines = document; VAT rule by status; ending in `=IF(AND(...),"ALL CONTROLS PASSED","ERROR FOUND")`.

**Quarter pack workbook:**

- Cover;
- Index;
- Lead (green links);
- Matrix (Executive Summary layout);
- one sheet per department;
- Documents;
- Decisions;
- VAT support;
- Related parties;
- Checks (banner).

The single source-of-truth data sheet is hidden-but-unlocked, with a key row for SUMIFS.

**Acceptance:**

- the file recalculates in LibreOffice with zero `#` errors;
- changing the yellow VAT cell flows to every total and the banner;
- formula totals equal the app to the fils.

### C7. Controls and exception rules

**Controls** (each shows PASS/FAIL on every report, dossier and export):

1. Every imported row maps to one company; no duplicate mapping per period.
2. Department tab totals = Summary column, or the difference is fully explained by approved decisions.
3. Engine recompute = imported totals within the rounding tolerance; any difference is disclosed.
4. Σ department lines = Σ document net + Σ excluded. Net + VAT = total. VAT = round(net × rate).
5. VAT = 0 on all Debit Notes and group-member Credit Notes; VAT > 0 on every non-zero Tax Invoice.
6. YTD = Σ quarters for gross, each department, net, VAT and total.
7. Numbering is gapless; no posted document has been edited (content-hash check).
8. The audit hash chain is intact.
9. Every exported file's totals = engine totals (checked at export time; the result is printed in the footer).

**Exception rules** (thresholds configurable):

- VAT treatment changed between quarters.
- Company status changed.
- Quarter swing > 50% **and** > AED 10,000.
- Credit Note issued.
- Negative line inside a document.
- Fixed-fee department missing for an active company.
- Active company absent from a quarter.
- **Liquidated or "not issued" company carrying a non-zero amount without an MD reference.**
- Company in liquidation charged outside its permitted departments.
- Nil all year.
- Same department billed for overlapping periods (narrations stating "Q1 & Q2").
- Manual override > AED ⚠ 5,000.
- Tax Invoice recipient with no TRN *(confirm whether the company is VAT-registered)*.
- Company in a workbook but not in the master.

### C8. Experience design — "Quiet Ledger"

**Direction:** calm, expensive, fast. Benchmarks: Linear, Stripe Dashboard, Ramp, Mercury. One brand, two registers:

- **Showcase surfaces** (Command Centre hero, Dossier header, Audit File cover). Editorial luxury: generous whitespace, a large serif display figure for the headline number, a subtle paper-grain overlay on a fixed layer only, and the double-bezel card (outer shell + inner core with concentric radii).
- **Work surfaces** (grid, registers, schedules, forms). Dense, quiet, fast. The density of a spreadsheet, the finish of a product.

**Tokens:**

- Colours:
  - navy #1B2A4A base and gold #C9A227 accent;
  - warm greys;
  - semantic tokens for success / warning / error / info. Defined in Tailwind v4 `@theme`, with light and dark themes designed together (dark by default after 18:00 Asia/Dubai, user override). Contrast is AA in both themes.
- Type:
  - **Geist** for UI;
  - **Geist Mono** or Geist with `tabular-nums` for every figure;
  - an editorial serif (e.g. Instrument Serif / Fraunces) for display numbers and the audit-file cover only.
  - Amounts are right-aligned; negatives in brackets; AED once in the header, not on every cell.
- Icons: one ultra-light line family (Phosphor Light). No emoji as icons.
- Elevation: soft diffused shadows or tonal surfaces. No harsh 1px grey boxes and no heavy drop shadows.

**Motion** (Emil primary, Jakub secondary, Jhey only for empty states and the close celebration):

- Grid, filters, keyboard actions and cell edits: **no animation**. These happen hundreds of times a day.
- Drawer, dossier and sheet transitions: 180–240 ms with a custom cubic-bezier or a bounce-0 spring. Exits are subtler than entries.
- KPI count-up and the dossier shared-layout transition (row → dossier header): once per visit.
- Quarter close: a single restrained gold "seal" moment when the audit file is approved.
- Animate transform / opacity only. `prefers-reduced-motion` turns every motion off. No pulsing dots, no hover-scale on every card, and no staggered entry on lists after the first load.

**3D** (React Three Fiber, lazy, decorative, never inside grids, forms or documents):

- **"Ledger Tower"** on the Command Centre: department columns rise in glass slabs to heights equal to the selected quarter's real totals, with gold threads flowing to company nodes. Slow orbit; pauses off-screen; static SVG fallback on low-power devices or reduced motion.
- Optional YTD 3D landscape with a 2D heatmap fallback.

**Productivity:**

- ⌘K palette: jump to company / document / report, and run "print Q3 dossier for …".
- Shortcuts sheet (?).
- Breadcrumbs, recent items, global search.
- Undo toast for reversible actions.
- Empty states that state the next step.
- Skeleton loaders, never spinners on data.

**Data viz:**

- department mix donut (≤ 6 slices, else a bar);
- quarter trend;
- Sankey (departments → companies);
- company × department heatmap;
- a waterfall for the lead schedule (gross → excluded → net → VAT → total).

Every chart has tooltips in AED, a click-through to its lines, a table alternative and a PNG export.

**Responsive:** full workspace on desktop; tablet: review, approve, dossier; phone: dashboard, dossier (document as a scrollable A4), approvals, print/export. No horizontal page scroll, and grids become cards on mobile. Safe areas respected; 44 px touch targets.

**Accessibility:** WCAG 2.2 AA, full keyboard use, visible focus, labelled icon buttons, and colour never the only signal.

**Performance:**

- first load < 2.5 s;
- 60 companies × 12 departments × 4 quarters scrolls at 60 fps;
- engine recompute < 200 ms;
- single-company PDF < 1.5 s;
- 52-company merged PDF < 20 s with progress shown;
- 3D, PDF and Excel libraries load on demand.

### C9. Non-functional

- **Security:** RLS per role; no service key in the client; signed URLs; Zod validation on client and server; CSP headers.
- **Reliability:** an error boundary per module (one failure never blanks the app); error toast with a reference ID.
- **Data:** nightly backup export; one-click full export.
- **Locale:** Asia/Dubai; DD-MMM-YYYY; AED.

---

## PART D — PHASE PROMPTS (one per message)

> Prefix each phase with: *"Follow Project Knowledge and DESIGN.md strictly. Build Phase N only. Finish with: files changed, tests written with PASS/FAIL output, screenshots (light, dark, mobile), and anything you could not do."*

**Phase 0 — Foundation + design system**

- Stack, Tailwind v4 `@theme` tokens per C8, layout shell (sidebar, top bar with period switcher, ⌘K skeleton).
- Auth with MFA and five roles; RLS baseline; `audit_log` hash chain with triggers.
- Error boundaries; `/admin/self-test`.
- Then generate **`DESIGN.md`** from the repo (skill `create-design-md`, repository mode, export target `css-tailwind`) and lint it.

*Accept:*

- login works per role;
- a Viewer write is rejected;
- the audit log shows before/after;
- the hash chain verifies;
- DESIGN.md lint + export pass;
- no console errors;
- Lighthouse accessibility ≥ 95.

**Phase 1 — Master data**

- Companies with aliases, effective-dated VAT-group status, company status, permitted departments while in liquidation.
- Departments, versioned policies, periods, settings, signatories, number series.
- Bulk import of the company master with preview.
- ⚠ Seed: the current company master and VAT group list.

*Accept:*

- VAT status "as at date" is correct at period boundaries;
- no hard delete of referenced records;
- every change is logged with a reason.

**Phase 2 — Allocation engine + tests**

- C4 in full, with integer fils and decimal.js.
- Tests:
  - half-up edges (x.xx5);
  - negative net → Credit Note;
  - group-member Credit Note VAT 0;
  - Adjustment lines inside positive documents;
  - Mark-up lines;
  - driver pools exact;
  - decision ordering;
  - idempotent hash;
  - narration text matches the template.

*Accept:* ≥ 95% coverage; all PASS in the runner and on /admin/self-test.

**Phase 3 — Import Studio**

- C5-3 in full.
- ⚠ Pre-approved mappings:
  - Falconcity Advertising → Basmat Alebdaa Advertising L.L.C. S.O.C.;
  - Mr. Ahmad Personal → Midnight Auto Repairing O.P.C. L.L.C.;
  - Osalco Trading Est. → Osalco Importing & Trading Est.;
  - "…Air Cargo … LLC" → "…L.L.C." (spelling only);
  - ⚠ Mazeed Accounting L.L.C. → *confirm: new company or renamed MCLEDGER Accounting Services?*

*Accept:*

- a duplicate file is blocked;
- a wrong column mapping is caught by the Summary-vs-tabs check;
- every amount shows its source cell.

**Phase 4 — Quarter Workspace + Management Decisions**

- Grid per C5-4; MD register.
- ⚠ Load the decisions in force:
  1. The quarter Summary is final; amounts on department tabs that do not reach it are management exclusions.
  2. Q1 — Mr. Ahmad Personal (Midnight Auto Repairs): not in group → no allocation; AED 375.00 in the Q1 Tax tab is excluded.
  3. Q1–Q3 — Parent (Almoosa Investment Group LLC): not invoiced (issuer; Head Office cost). *(Confirm.)*
  4. Q2 — Nippon International L.L.C.: not liquidated; Tax Invoice 4,875.00 + 243.75 = 5,118.75; disclosed management adjustment.
  5. Q3 — Falconcity Hotels and Resorts: Liquidated; net AED 375.00 (Legal 750.00 less Tax 375.00) carried to Q4; no Q3 document.
  6. ⚠ Q3 — Al Moosa Tech Solutions L.L.C.: marked "not issue", AED 12,625.00 → *authority and reason needed.*
  7. ⚠ Q3 — Mr. Alharith Almoosa: marked "not issue", AED 667.50 (Transport) → *authority and reason needed.*
  8. ⚠ Q3 — Al Moosa Owners Association Management Services (AED 3,000.00) and Wheel International Co. (AED 2,250.00): liquidated with charges on the Summary → *confirm exclusion or carry-forward.*
  9. ⚠ Permitted departments for each company in liquidation.

*Accept:*

- Part E golden figures tie to the fils;
- visible-row totals correct for every tax-status × document combination;
- an override demands a reason and is logged;
- clicking a company opens its dossier route (placeholder data is not allowed — build the route shell with real engine output).

**Phase 5 — Company Dossier + documents** *(the core of v3)*

- C5-5 and C5-7 in full:
  - live A4 preview;
  - schedule; history; checks;
  - action bar with View / Print / PDF / Excel / Pack / Copy narration / Email draft;
  - print-options sheet;
  - print CSS (`@page size: A4`, page breaks, repeated table headers);
  - per-company Excel per C6;
  - posting via server function with gapless numbering;
  - reversal by Credit Note;
  - PINT AE field export.

*Accept:*

- for 3 sample companies (one Tax Invoice, one Debit Note, one Credit Note): app = PDF = Excel to the fils;
- PDFs rendered to images and inspected;
- Excel recalculated in LibreOffice: zero `#` errors and banner PASS;
- J/K and P/E shortcuts work;
- posting twice is impossible;
- 20 parallel posts stay gapless.

**Phase 6 — Print Studio**

- C5-6 in full: scope selector (one / selected / all / current filter); contents; merged PDF with cover and index; ZIP per company; consolidated Excel; skipped-company list with reasons; job history with SHA-256.

*Accept:*

- printing "all Q3" yields 42 documents (25 TI, 16 DN, 1 CN) and lists the 10 skipped companies with their reasons;
- the merged-PDF page count matches the index;
- re-download returns an identical hash.

**Phase 7 — Quarter Audit File**

- C5-8: index A–H, lead schedule with W/P refs, automatic tick-marks with legend, auditor notes, sign-offs, CLOSED stamp, bookmarked PDF and multi-sheet Excel.

*Accept:*

- every lead-schedule figure drills to its working paper;
- every tick-mark corresponds to a passing engine check;
- the exported file's totals = the app.

**Phase 8 — Reports, YTD and report builder**

- R2–R16, filters, live visible-row totals, drill-down, saved views.

*Accept:*

- YTD = Σ quarters for every measure;
- R6 by emirate ties to the Tax Invoice register;
- each report's totals = engine (automated test per report).

**Phase 9 — Period close, exceptions, audit & evidence**

- C5-10/11 and C7.

*Accept:*

- editing a closed period is rejected;
- reopen needs the Reviewer and is logged;
- every C7 control appears on the close certificate;
- every exception rule has a test that fires it.

**Phase 10 — Experience polish & hardening**

- Apply C8 fully: Ledger Tower with fallback, motion system, charts, ⌘K actions, empty states, mobile layouts.
- Then audit:
  1. **web-design-guidelines** skill: fetch the current Vercel Web Interface Guidelines and fix every finding;
  2. **design-motion-principles** in Audit mode: fix every Critical and Important finding;
  3. **ui-ux-pro-max** pre-delivery checklist at 375 / 768 / 1024 / 1440 px in both themes;
  4. RLS tests per role per table;
  5. full regression of Phases 0–9.

*Accept:*

- Lighthouse ≥ 90 performance and ≥ 95 accessibility;
- zero console errors on every route;
- all tests PASS;
- screenshots light, dark and mobile.

---

## PART E — GOLDEN-FIGURE ACCEPTANCE PACK (after Phases 4 and 10)

> *"Import the attached quarter workbooks. For each figure, show the app value, an independent recompute from raw lines, and the difference. Any difference ≠ 0.00 must be traced to a line or decision. Never adjust a figure to match."*

**Q3 2026 — verified independently from `AIG_Q3-2026_Cost_Allocation_All_SAMPLE.xlsx` (52 companies, all row and column tie-outs 0.00):**

| Ref | Measure | Expected (AED) |
| --- | --- | --- |
| E3.1 | Gross (Σ 10 departments, 52 companies) | 1,252,742.19 |
| E3.2 | Excluded — not issued (Parent 156,224.79; Tech Solutions 12,625.00; Mr. Alharith 667.50; Midnight 0.00) + liquidated (Owners Association 3,000.00; Wheel 2,250.00; Falconcity Hotels 375.00; three others 0.00) | 175,142.29 |
| E3.3 | Net of documents (E3.1 − E3.2) | 1,077,599.90 |
| E3.4 | Output VAT (Tax Invoices only) | 28,201.43 |
| E3.5 | Total of documents | 1,105,801.33 |
| E3.6 | Documents: Tax Invoices / Debit Notes / Credit Notes / skipped | 25 / 16 / 1 / 10 |
| E3.7 | Department totals: Accounts 317,250.00 · Tax 286,500.00 · Legal 294,000.00 · IT 15,685.31 · Vehicle 135,530.92 · Insurance 43,898.30 · HR 26,144.00 · Procurement 5,700.00 · Admin 88,767.66 · Transport 39,266.00 | Each tie, Σ = E3.1 |
| E3.8 | Credit Note — Al Ardh Al Taibah Landscape Gardening (group member) | −10,875.00, VAT 0.00 |
| E3.9 | Adjustment lines inside positive documents: Alco Palm Jumeirah Villas Tax −6,000.00; DIRP Insurance −166.99; Emirates Tex-Cote Insurance −162.60; Evolve Contracting Insurance −1,185.17 | Shown as Adjustment lines |
| E3.10 | Falconcity Hotels: Legal 750.00 + Tax −375.00 | 375.00 carried to Q4; no document |

**Q1, Q2 and YTD:**

| Ref | Test | Expected |
| --- | --- | --- |
| E1 | Q1 gross / excluded / net / VAT / total | ⚠ from the validated Q1 pack |
| E2 | Q2 gross / excluded / net / VAT / total | ⚠ from the validated Q2 pack |
| E2.1 | Q2 Nippon International Tax Invoice | 4,875.00 + 243.75 = 5,118.75 |
| E1.1 | Q1 Mr. Ahmad Personal Tax 375.00 | Excluded; no document |
| E4 | YTD = Q1 + Q2 + Q3, every measure and department | Difference 0.00 |
| E5 | VAT on every Debit Note and group-member Credit Note | 0.00 |
| E6 | Company mapping | 100%, no duplicates per quarter |
| E7 | Per-company PDF = Excel = app (3 samples) and quarter pack Excel recalculated | Equal to the fils; banner ALL CONTROLS PASSED |

⚠ Files: `Q1_COST_ALLOCATION_2026.xlsx`, `FINAL - Q2 COST ALLOCATION FROM AIG APR TO JUN 26.xlsx`, `FINAL_Q3_COST_ALLOCATION_JUL-SEP26 - V2.xlsx`.

---

## PART F — SKILLS MAP (create in Lovable → Workspace → Skills; reference by name in phase messages)

**Your design skills** (upload the SKILL.md files you already have):

| Skill | Governs | Phase | Scope rule |
| --- | --- | --- | --- |
| `ui-ux-pro-max` | Design-system choice, UX rules, pre-delivery checklist | 0, 10 | Accessibility and touch rules override every aesthetic rule |
| `high-end-visual-design` | Showcase surfaces: whitespace, double-bezel cards, island buttons, typography | 0, 5, 7, 10 | **Showcase only.** Its `py-24+` whitespace and entry animations do not apply to the grid, registers or forms. Its font ban (no Inter) is adopted → Geist |
| `frontend-design` | Overall aesthetic intent, no "AI slop" | 0, 10 | Direction fixed as "Quiet Ledger" (C8) |
| `design-motion-principles` | Motion creation (Phase 10) and motion audit | 5, 10 | Weighting: Emil primary, Jakub secondary, Jhey for empty states and the close seal |
| `apple-ui-designer` | Phone and tablet layouts: sheets, large titles, safe areas | 5, 10 | Mobile surfaces only |
| `web-design-guidelines` | Final UI compliance audit (fetches the live Vercel rules) | 10 | Run after every major UI phase too |
| `create-design-md` | Locks tokens into `DESIGN.md` | 0, then after any token change | Repository mode, `css-tailwind` export |
| `web-artifacts-builder` | Only if you also want a single-file offline demo of the dossier | optional | Not used inside Lovable |
| `material-3` | Reference for elevation and shape token scale only | 0 | **Do not** use MD3 components or Roboto — they conflict with the navy/gold brand |

**Accounting skills to create** (short, testable rule sets):

| Skill | Enforces | Phase |
| --- | --- | --- |
| `aig-project-knowledge` | Part B | all |
| `fils-money-engine` | Integer fils, decimal.js, half-up, largest remainder, invariants | 2 |
| `uae-vat-documents` | Art. 59/60 content, tax-group Debit Note wording, VAT 201 emirate mapping, PINT AE fields | 5, 8 |
| `excel-house-style-export` | Classic functions, colour legend, Checks banner, LibreOffice-safe | 5, 6, 7 |
| `audit-file-tickmarks` | W/P index A–H, tick-mark legend, sign-off rules | 7 |
| ⚠ *your next skill* |  |  |

---

## PART G — DO-NOT LIST (paste when Lovable drifts)

- Do not build any external ledger/ERP import, reconciliation, bridge or voucher export.
- Do not do `number` arithmetic on money or round with `toFixed`.
- Do not compute posting totals in the browser only.
- Do not hard-delete or edit posted documents, closed periods or audit rows.
- Do not net a negative department line away; show it as an Adjustment line.
- Do not hardcode companies, departments, rates or decisions.
- Do not use XLOOKUP / FILTER / SORT / UNIQUE / LET / LAMBDA in exports.
- Do not let a printed or exported figure differ from the engine; check at export time.
- Do not print an unposted document without the DRAFT watermark.
- Do not hide a variance to make a control pass.
- Do not ship mock data, placeholders, dead buttons or "coming soon".
- Do not animate grids, filters, keyboard actions or printed documents. Do not put 3D anywhere near data entry.
- Do not mark a phase complete without test output and screenshots.
