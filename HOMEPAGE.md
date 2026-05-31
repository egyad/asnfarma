# Homepage (Beranda) — DashboardScreen

**File:** `src/screens/DashboardScreen.tsx`  
**Route:** `Beranda` tab (bottom tab navigator)

---

## Overview

The homepage is the first screen users land on. It gives a full picture of study progress, offers a quick resume button, and keeps the exam countdown front and center. All data comes from Zustand / AsyncStorage — no network call is made on this screen.

---

## Layout (top to bottom)

### 1. Greeting Bar

```
[Avatar]  Selamat pagi,       [Formasi Chip]
          Nama Pengguna
```

- **Avatar** — initials-based circle generated from `s.nama`.
- **Greeting text** — time-based: Pagi / Siang / Sore / Malam.
- **Formasi Chip** — shows short label + icon of the active formasi (e.g. "CPNS", "PPPK"). Sourced from `FORMASI[s.formasi]`.

---

### 2. Overall Progress Card

- Circular `ProgressRing` (72 px, stroke 8) showing `overallProgress(s.progress, f)` as a %.
- Copy changes:
  - `0%` → "Belum ada bab selesai. Yuk, mulai belajar dari bab pertama!"
  - `>0%` → "X dari Y bab selesai. Terus konsisten, kamu pasti bisa."

---

### 3. Resume / Welcome Banner

Shown only when there is recent activity or the user is brand new. Mutually exclusive states:

| State | Condition | Label | Action |
|---|---|---|---|
| **Lanjutkan Belajar** | `s.lastActivity` exists | "LANJUTKAN BELAJAR" + chapter title + progress bar | Navigate to `Reader` for that chapter |
| **Mulai Perjalananmu** | `overall === 0` and no `lastActivity` / `lastTryout` | "MULAI PERJALANANMU" + first TKD chapter | Navigate to first TKD chapter |
| *(hidden)* | Progress > 0 but `lastActivity` cleared | — | Nothing shown |

Background color:
- Lanjutkan = `t.formasiColor` (formasi accent)
- Mulai = `t.brand` (primary brand color)

---

### 4. Module Cards (MODUL BELAJAR)

Three tappable cards stacked vertically:

| Module | Icon | Color |
|---|---|---|
| TKD — Kompetensi Dasar | `school` | `t.brand` |
| TKP — Karakteristik Pribadi | `heart-circle` | `t.brand` |
| SKB — Kompetensi Bidang | Formasi-specific icon | `t.formasiColor` |

Each card shows:
- Module label + full name
- `ProgressBar` with % completion
- `moduleProgress(s.progress, key, formasi)` value

Tapping navigates to `Materi` screen with `{ module }` param.

> **Memoisation:** the `mods` array is wrapped in `useMemo` with deps `[s.progress, f, t]` so module cards re-render only when progress, formasi, or theme changes.

---

### 5. Exam Countdown Card

- Shows days remaining (`daysUntil(s.examDate)`) in large bold text.
- Special copy when `days === 0`: "Hari ujian telah tiba!"
- Tapping opens `ExamDateSheet` (bottom sheet with a `DateTimePicker`).

**ExamDateSheet behaviour:**
- Android: renders a native date-picker dialog directly (no Sheet wrapper).
- iOS: renders `DateTimePicker` in spinner mode inside a `Sheet`.
- Minimum selectable date is tomorrow.
- Saves ISO date string to `s.setExamDate()` on confirm.

---

### 6. Last Tryout Card

Shown only when `s.lastTryout` exists.

- Mini `ProgressRing` (52 px) coloured green (`t.success`) if passed, red (`t.danger`) if not.
- Score display: `score / max`
- Status `Chip`: "Lolos PG" or "Di bawah PG"
- Date of the last tryout.
- Tapping navigates to the `Tryout` tab.

---

## State & Store Dependencies

| Store field | Used for |
|---|---|
| `s.nama` | Avatar initials + greeting name |
| `s.formasi` | Module colors, chapter list filtering |
| `s.progress` | `overallProgress`, `moduleProgress`, `chapterPct` |
| `s.lastActivity` | Resume banner (module, chapter no, title, id) |
| `s.lastTryout` | Last tryout card (score, max, passed, date) |
| `s.examDate` | Countdown days |

No Firebase / network calls — all reads from Zustand (backed by AsyncStorage).

---

## Navigation Actions

| Target | Trigger |
|---|---|
| `Reader` (module, no, id, title) | Resume banner tap / welcome banner tap |
| `Materi` (module) | Any module card tap |
| `ExamDateSheet` (local modal) | Countdown card tap |
| `Tryout` tab | Last tryout card tap |

---

## Related Files

- [src/screens/DashboardScreen.tsx](../src/screens/DashboardScreen.tsx) — screen implementation
- [src/store/useAppStore.ts](../src/store/useAppStore.ts) — `overallProgress`, `moduleProgress`, `daysUntil`, store state
- [src/data/content.ts](../src/data/content.ts) — `getChapters`, `chapterId`
- [src/theme/tokens.ts](../src/theme/tokens.ts) — `FORMASI`, `RADIUS`
- [src/components/index.ts](../src/components/index.ts) — `Card`, `Chip`, `Avatar`, `ProgressBar`, `ProgressRing`, `Sheet`
