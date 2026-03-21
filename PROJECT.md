# PROJECT.md — תיק טיול

## Project Goal
A single HTML file app for building a complete "trip folder" (תיק טיול) before each annual school trip.
Nir fills in all required forms online, then exports a PDF per appendix — formatted to Ministry of Education standards.

---

## Files
- **Main file:** `/Users/nirozari/Projects/Tik Tiyul/index.html`
- **GitHub:** https://github.com/niroari/tiktiyul
- **Vercel (live):** https://tiktiyul.vercel.app
- **Reference PDFs:** same folder (ogdan.pdf + individual appendix files + הודעת מסע.pdf)
- Every push to GitHub auto-deploys to Vercel

---

## Tech Stack
- Single HTML file (no build tools)
- Firebase Auth (Google + email/password) — same pattern as Shelach Reports
- Firebase Firestore — cloud storage per user
- html2canvas + jsPDF — PDF export (one PDF per appendix)
- SheetJS (xlsx.js) — CSV import for student list
- Local file:// bypass — skips login when opened from disk

---

## Firestore Structure
```
users/{uid}/trips/{tripId}                  — trip metadata (owner only)
  name, school, fromDate, toDate, area, classes
  leaderName, principalName, coordinatorName
  accommodation, transport, createdAt

users/{uid}/trips/{tripId}/forms/{formId}   — per-appendix data
  formId = appendix_a ... appendix_j, masa, team, security_approval

users/{uid}/shared_trips/{tripId}           — pointer written when user joins a shared trip
  ownerUid, tripId, tripName, school, fromDate, toDate, joinedAt

tiyul_invites/{token}                       — invite docs for trip sharing
  ownerUid, tripId, tripName, school, fromDate, toDate, createdAt

tiyul_signatures/{uid}_{tripId}_{role}      — remote signing docs (public read/write)
  role = coordinator | principal | c_principal | masa_principal
```

---

## Sections — Build Status

| ID | Label | Content | Status |
|----|-------|---------|--------|
| dashboard | דשבורד | Stats cards → class breakdown → food prefs (collapsible) → trip info + appendix completion → medical list | ✅ done |
| security | אישור ביטחוני | Upload a PDF of the security approval. Preview in iframe. Print button. Saves to Firestore `security_approval` doc. | ✅ done |
| א (a) | ביקורת לפני יציאה | Checklist: ~20 rows, each with "week before" + "morning of trip" checkboxes + notes | ✅ done |
| ב (b) | אישור מנהל ורכז | Schedule table + notes + signatures (principal + coordinator) | ✅ done |
| ג (c) | כתב מינוי | Simple appointment letter — classes, dates, area, principal signature | ✅ done |
| ד (d) | לוח זמנים | Dynamic table: day / time / activity+place / notes | ✅ done |
| ה (e) | טלפונים חיוניים | Two parts: essential contacts (fixed rows) + bus crew table (dynamic, per bus) | ✅ done |
| team | צוות הטיול | Table of staff: name / role / phone. Autocomplete feeds into נספח ו מורה fields | ✅ done |
| ו (f) | טבלת שליטה | Bus control table: 3 class dropdowns per bus (split-aware), extra crew rows (מורה/מלווה), auto-computed student+escort counts, red warning >54, class split tool in summary table | ✅ done |
| ז (g) | רשימת תלמידים | Excel import (with gender), display by class with going/not-going toggle, class filter dropdown | ✅ done |
| ח (h) | אישור הורים | Upload a PDF specific to each trip (not a built form) | ✅ done |
| ט (i) | ציוד חובה | Static checklist + option to add custom items | ✅ done |
| י (j) | מגבלות רפואיות | Table: student name / class / medical issue / supervision notes. Autocomplete input. Per-student medical certificate upload (PDF/image). | ✅ done |
| food | העדפות מזון | Select students (autocomplete), mark vegetarian/vegan, gluten-free, food sensitivity + free text detail. Firestore `food_prefs`. Dashboard stat card. | ✅ done |
| הודעת מסע (masa) | הודעת מסע של"ח | 2-page form: general info + schedule table + role holders + signatures | ✅ done |
| rooms | חלוקת חדרים | Room assignment for hostel trips: boys/girls separated, drag-and-drop students between rooms, class filter, manual reorder (↑↓), print/PDF export | ✅ done |
| print | נספחים נוספים להדפסה | 6 static printable forms (no data entry): נספח ט"ו (bus inspection, from Word file, compact mode for 1 page) + נספחי ז', ט', י"א, י"ב, י"ג (from PDF). Print individually or all at once. No Firestore. | ✅ done |
| signs | שילוט אוטובוסים | School logo (URL or file upload), add buses (number + classes text), print A4 landscape sign per bus with giant text. Saves to Firestore under formId 'signs'. | ✅ done |

---

## Shared Trip Data (entered once, auto-filled into all forms)
- שם הטיול, שם ביה"ס, תאריכי יציאה/חזרה, כיתות
- שם אחראי/ת טיול, שם מנהל/ת, שם רכז/ת טיולים
- אזור/יעד, מקום לינה, חברת הסעה

---

## Form Content Reference (from ogdan.pdf)

### נספח א – ביקורת לפני יציאה (4 pages in ogdan, pages 4–5)
Checklist table with columns: מס"ד | נושא הבדיקה | פירוט הנושא | שבוע לפני הטיול | בוקר הטיול | הערות
Categories: 1-תיק אחראי, 2-אוטובוס, 3-ציוד, 4-תדריך, 5-דיווחים
Bottom: טיול/סיור כיתות + שם אחראי/ת טיול + חתימה

### נספח ב – אישור מנהל ורכז (ogdan page 7, called נספח ג there)
Fields: כיתות, אזור, מקום לינה, חברת הסעה, מס' מלווים נושאי נשק, מס' הורים, טלפון אחראי
Schedule table: תאריך | משעה | עד שעה | פירוט הפעילות | הערות
Notes field (אחראי טיול), notes field (מנהל), two signature lines (רכז + מנהל)

### נספח ג – כתב מינוי (ogdan page 6, called נספח ב there)
Fields: תאריך, שם ביה"ס, שם אחראי/ת
Body text (fixed): "הריני ממנה אותך לאחראי/ת טיול לתלמידי כית/ות ___ שיתקיים בתאריכים ___ במקום/באזור ___"
Two signature lines: שם המנהל/ת + חתימת מנהל/ת ביה"ס

### נספח ד – לוח זמנים (separate file: נספח ד לוח זמנים.pdf)
Table: יום הטיול | הזמן | הפעולה והמקום | הערות
Dynamic rows, multiple days

### נספח ה – טלפונים חיוניים (ogdan page 14, called נספח י there)
Part 1 — Essential contacts (fixed rows with pre-filled roles):
  1. אחראי/ת טיול, 2. מנהל/ת ביה"ס, 3. חדר מצב (02-6222211 pre-filled),
  4. מוקד עירוני (106 pre-filled), 5. מנהל מח' ביטחון ובטיחות,
  6. תחנת משטרה באזור, 7. יחידת פיקוח טיולים (57-2258624 pre-filled), 8–10. free
Columns: מס' | תפקיד | שם | טלפון סלולרי | פקס/אחר
Part 2 — Bus crew (per bus): אוטובוס מס' | כתה/ות | מס' תלמידים | תפקיד | שם מלווה | מס' טל"ס
  Roles per bus: נהג, מדריך/ה, מורה אחראי/ת, מורה/הורה/מלווה נוסף, מאבטח/מע"ר/חובש

### נספח ו – טבלת שליטה (separate file: טבלת שליטה יום 1.pdf)
Per bus columns: כיתות | שם אחראי | שם מורה נוסף | שם נהג | מאבטחים וחובשים | מדריך | מספר תלמידים | מספר מלווים | סה"כ
Bottom summary table: כיתה | מס' תלמידים (מתוכנן / בפועל)
Then: count rows: תלמידים / מורים+מלווים+הורים / מאבטחים וחובשים / מדריכים / סה"כ נוכחים

### נספח ז – רשימת תלמידים
Excel import (SheetJS). Supports two formats:
- New: [מספר, ת.ז, שם משפחה, שם פרטי, כיתה, מקבילה, מין(נ/ז), טלפון] — auto-detected when col1 is a 9-digit ID
- Old: [ת.ז, שם משפחה, שם פרטי, כיתה, מקבילה, טלפון]
Stores: id, last, first, class (`כיתה׳מקבילה`), gender (נ/ז), phone, going (bool)
Display: grouped by class, going/not-going toggle per student, gender badge (זכר/נקבה)

### נספח ח – אישור הורים
User uploads their own PDF per trip (not a built form)

### נספח ט – ציוד חובה (separate file)
Static checklist: כובע, חולצת בית ספר, נעלי הליכה סגורות, ארוחת בוקר+צהריים ליום ראשון,
מים ליטר (חובה), פנס, כלי רחצה, בגדים להחלפה, בגדים חמים לערב, שק שינה ומצעים
+ option to add custom items

### נספח י – מגבלות רפואיות (ogdan page 10, called נספח ו there)
Table: מס"ד | שם התלמיד | מכתה | הבעיה הרפואית | דגשים להשגחה על התלמיד
Pulls from student list (appendix ז) — user selects students and fills in medical details

### הודעת מסע של"ח (separate file: הודעת מסע.pdf) — 2 pages
Page 1:
  Header: תאריך, אל: תחום של"ח, מאת: שם בית הספר
  כללי section: תאריכי מסע (from–to), מס' תלמידים, מקום התארגנות,
    מקום פיזור (radio: בביה"ס / בתוך היישוב / פיזור למושבים / אחר),
    שם אחראי, חברת הדרכה + מדריכים
  מינהל והכנות: תאריך תדרוך צוות, הכנת תלמידים (תכנים + מינהלות freetext),
    כמה הוזמנו / מתוכם, תאריך בקשת אישור ביטחוני
Page 2:
  Schedule table: יום | תאריך | זמן יציאה לשטח | ממקום | פרוט מסלול ואתרים |
    זמן סוף מסלול | נקודת פינוי 1 | נקודת פינוי 2 | מקום לינה | תוכנית ערב
  אלטרנטיבות (freetext)
  Role holders table: סדר | תפקיד (אחראי/סגן/חובש/מדריך עזר x3) |
    שם פרטי ומשפחה | מס' זהות | טלפון נייד/מירס | תעודה בתוקף עד
  Signatures: אחראי/ת (canvas, direct) + מנהל/ת ביה"ס (remote link, role=masa_principal)

---

## Decisions Made
- Single HTML file (like Shelach Reports) — simple, no build tools
- Firebase Auth + Firestore — multi-user, cloud storage
- One PDF export per appendix (not one big combined PDF)
- נספח ח (parent permission) — user uploads their own PDF per trip, not a built form
- Student list (נספח ז) — Excel import via SheetJS; supports old and new column formats
- Medical limitations (נספח י) — pulls names from student list, user adds medical details
- Local file:// bypass — skip login when opening from disk for preview
- Canvas signatures — shared `setupCanvas(canvas, onEnd)` function using mouse+touch events with `getBoundingClientRect` scaling factor. Works for both in-form and remote signing.
- Remote signing via shared link — Firestore collection `tiyul_signatures/{uid}_{tripId}_{role}`, public page at `?tiyulsign=DOCID` (no login required). Used in: נספח ב (coordinator + principal), נספח ג (principal), הודעת מסע (principal = masa_principal).
- **הודעת מסע PDF approach:** original PDF pages embedded as base64 JPEG (~90KB each), text overlaid using `position:absolute` divs via `masaCell()`. Print button uses a separate clean HTML table layout (`buildMasaPrintHtml()`), not the background image overlay.
- **צוות הטיול** (formId=`team`) — standalone section between ה and ו. Staff names feed into נספח ו מורה fields via `<datalist>` autocomplete with phone auto-fill.
- **Dashboard** — default landing tab when opening a trip. Loads gStudents, teamMembers, fBuses, jRows and all form docs in parallel for stats + completion status. "יוצאים בסה"כ" = students + staff (teamMembers.length).
- **Trip sharing** — owner clicks "🔗 שתף טיול" → creates `tiyul_invites/{token}` → share link `?jointiyul={token}`. Joiner opens link, logs in, gets written to `users/{uid}/shared_trips/{tripId}`. App then loads their shared trips alongside own trips. `currentTripOwnerUid` tracks whose Firestore path to use; `formRef()` uses it. Collaborators see a "שותף" badge and cannot edit trip metadata.
- **Firestore rules** — production rules deployed (replaced test-mode open rules). Collaborator access gated by existence of `users/{uid}/shared_trips/{tripId}` doc.
- **Delete trip** — 🗑 button on each trip card in the main list. Owned trips: deletes `users/{uid}/trips/{tripId}` doc from Firestore (subcollections/forms become orphaned but are inaccessible). Shared trips: only deletes the `users/{uid}/shared_trips/{tripId}` pointer. Both show a confirmation dialog before acting.
- **חלוקת חדרים** (rooms) — sits after הודעת מסע. Pulls from `gStudents` (auto-loads from Firestore if tab G not yet visited). Auto-assign splits by gender + class into preferred size chunks (4/5/6, no hard limit). Room cards: editable number field, ↑↓ reorder buttons, draggable student chips, ✕ to remove from room. Unassigned students shown as draggable chips (colored by gender: blue/pink/yellow) — drag to any room card to assign. Class filter dropdown reads from gStudents. Empty rooms always visible through filter so new rooms can receive students. Saves to `formRef(tripId, 'rooms')`. Toolbar has: preferred size radio + ⚡ auto-assign only (+ חדר בנים/בנות buttons are below the table, not in toolbar). "נקה הכל" is a separate red button in the top-left corner of the box.
- **Appendix ז — upload UX** — when no file is loaded, shows a blue instructions box explaining how to export from משו"ב (רשימת שמות שכבתית, required fields). Instructions disappear once a file is imported. A red "הסר קובץ" button sits in the top-left corner of the box, visible only when a file is loaded — clears `gStudents`, resets the file input, and saves an empty array to Firestore after confirmation.
- **Gotcha (temporal dead zone):** module-level `let` variables used by boot-time functions must be declared in the `// ─── State ───` section BEFORE the boot code. Declaring them later causes silent async crashes.
- **Print buttons:** every export function takes `printMode = false`. When `true`, calls `doPrint(html)` → `#print-area` + `window.print()`. All appendices follow this pattern.
- **Appendix ז PDF page breaks:** skipped — browser handles page breaks natively via print button.
- **Appendix ז — "לא יוצאים" modal** — button appears only when at least one student has `going === false`. Modal auto-refreshes when `toggleGoing` is called while modal is open. Shared filter+sort logic lives in `getNotGoingStudents()` helper.
- **נספחים נוספים להדפסה** (formId=`print`) — static HTML forms, no Firestore. Source: "נספחים להדפסה.pdf" + "נספח טו...docx". נספח ט"ו uses `printPrintForm(id, compact=true)` — smaller margins/font to fit on one A4 page. Other forms use default (compact=false).
- **שילוט אוטובוסים** (formId=`signs`) — prints via `window.open()` (not `doPrint`) because `@page { size: landscape }` only works in a standalone document. Each sign is a `div.sign-page` with `height:100vh`. Page breaks via `.sign-page + .sign-page { page-break-before: always }` — avoids blank trailing page that `page-break-after: always` would add after the last sign.
- **Student autocomplete (appendix י + food prefs):** `<input list="...">` + `<datalist>` pattern. Options formatted as `"${last} ${first} (${class})"`. On add, match by that exact string. **Gotcha:** JS string args in `onclick`/`oninput` attributes must use single quotes, not `JSON.stringify` — `JSON.stringify` wraps in double quotes which breaks the HTML attribute.
- **Medical certificate upload (appendix י):** Per-student file (PDF/image) read as base64 dataURL and saved to its own Firestore doc `med_cert_{studentId}` (not inside `appendix_j`) to stay well under the 1MB document limit. All certs for existing rows loaded in parallel on tab open.
- **Dashboard button:** Moved from the tab bar into the trip header card (next to ✏️ עריכה and 🔗 שתף). Active state via `#trip-dashboard-btn.active` CSS. `showAppendix` calls `classList.toggle('active', id==='dashboard')` on it.
- **Appendix ו — bus data model:** `fBuses[bi].classSelections = ['','','']` (up to 3 class values) replaces the old free-text `classes` string. Split value format: `"className|partIndex"` (0-indexed). Class splits stored in `fSplits = { [className]: ['count1','count2','count3'] }`. Both fields saved in `appendix_f` Firestore doc alongside `buses` and `actual`. Students and escorts are computed on-the-fly (`calcBusStudents`, `calcBusEscorts`) and shown read-only — not stored. Total shown in bus header; turns red if >54. `updateBusCounts(bi)` refreshes only the 3 count spans (avoids full re-render on every keystroke).
- **Extra crew rows (appendix ו):** `bus.extraTeachers = [{role, name, phone}]`. Role dropdown: מורה / מלווה. Escort count = all filled fixed crew rows where index ≠ 2 (נהג) + all filled extraTeachers rows.
- **Class filter (appendix ז):** `<select id="g-class-filter">` populated dynamically in `renderGTable()` from unique class values in `gStudents`. Preserves selection across re-renders by reading `classFilterEl.value` before rebuilding options.
- **Dashboard section order:** stat cards → class breakdown → food prefs (collapsible `<details>`) → two-column grid (trip info + appendix completion) → medical list. Class breakdown and food prefs were moved above the grid so the most-scanned summary info appears first.
- **אישור ביטחוני** (formId=`security`) — single PDF upload (same pattern as אישור הורים / med cert): read as base64 dataURL, stored in `formRef(tripId, 'security_approval')`. Preview via `<iframe src="dataUrl">`. Print opens a new window with an iframe and calls `win.print()`. Tab button sits to the left of א׳-ביקורת in the tab bar.
