# PROJECT.md — תיק טיול

## Project Goal
A single HTML file app for building a complete "trip folder" (תיק טיול) before each annual school trip.
Nir fills in all required forms online, then exports a PDF per appendix — formatted to Ministry of Education standards.

---

## Files
- **Main file:** `/Users/nirozari/Projects/Tik Tiyul/index.html`
- **Reference PDFs:** same folder (ogdan.pdf + individual appendix files + הודעת מסע.pdf)
- GitHub / Vercel: not yet set up

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
users/{uid}/trips/{tripId}          — trip metadata
  name, school, fromDate, toDate, area, classes
  leaderName, principalName, coordinatorName
  accommodation, transport
  createdAt

users/{uid}/trips/{tripId}/forms/{formId}   — per-appendix data
  formId = appendix_a, appendix_b, ... appendix_j, masa
```

---

## Appendices — Build Status

| ID | Label | Content | Status |
|----|-------|---------|--------|
| א (a) | ביקורת לפני יציאה | Checklist: ~20 rows, each with "week before" + "morning of trip" checkboxes + notes | 🔧 next |
| ב (b) | אישור מנהל ורכז | Schedule table + notes + signatures (principal + coordinator) | pending |
| ג (c) | כתב מינוי | Simple appointment letter — classes, dates, area, principal signature | pending |
| ד (d) | לוח זמנים | Dynamic table: day / time / activity+place / notes | pending |
| ה (e) | טלפונים חיוניים | Two parts: essential contacts (fixed rows) + bus crew table (dynamic, per bus) | pending |
| ו (f) | טבלת שליטה | Bus control table: classes, teacher, driver, security, guide, counts | pending |
| ז (g) | רשימת תלמידים | CSV import, display by class | pending |
| ח (h) | אישור הורים | Upload a PDF specific to each trip (not a built form) | pending |
| ט (i) | ציוד חובה | Static checklist + option to add custom items | pending |
| י (j) | מגבלות רפואיות | Table: student name / class / medical issue / supervision notes | pending |
| הודעת מסע (masa) | הודעת מסע של"ח | 2-page form: general info + schedule table + role holders | pending |

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
CSV import, display grouped by class, print-ready list

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
  Header: תאריך, אל: תחום של"ח, מאת: מנהל/ת ביה"ס
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

---

## Decisions Made
- Single HTML file (like Shelach Reports) — simple, no build tools
- Firebase Auth + Firestore — multi-user, cloud storage
- One PDF export per appendix (not one big combined PDF)
- נספח ח (parent permission) — user uploads their own PDF per trip, not a built form
- Student list (נספח ז) — CSV import
- Medical limitations (נספח י) — pulls names from student list, user adds medical details
- Local file:// bypass — skip login when opening from disk for preview
