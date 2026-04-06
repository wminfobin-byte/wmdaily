# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A browser-based daily sales performance dashboard (`index.html`) for a multi-group sales organization. Users upload three types of CMS Excel files (roster, DB count, contract count), and the dashboard generates group/individual KPI tables with filtering, sorting, and export — all client-side with no backend.

## Architecture

Single-file web app (`index.html`, ~3345 lines) with no build step or framework:
- **SheetJS (xlsx-js-style)** via CDN for Excel parsing and styled export
- **html2canvas** via CDN for PNG screenshot export
- **IndexedDB** (`dbPerfDashboard`, version 6) with 6 object stores: `dates`, `keyMeta`, `roster`, `contract`, `fileMeta`, `fileArchive`
- **localStorage** for accumulated daily snapshots and GitHub token
- **GitHub API** for optional cloud sync (repo: `wminfobin-byte/wm_daily`, file: `data.json`)
- All data processing happens in-browser; sensitive customer data never leaves the browser

## Data Model

Three input files, each with its own column mapping constants (0-based indices):

- **Roster file** (`WM_근태보고_그룹_yyyymm.xls`) — `ROSTER_COL`: role(0), group(1), center(2), team(3), name(4), hireDate(5)
- **DB file** (`WM_고객관리_yyyy-mm-dd.xlsx`) — `DB_COL`: customerKey(6), obName(7), distributeDate(8), inflowDate(9), dbInfo(15), mediaCode(19), package(20), orderDate(23), mainGift(27), customerStatus(29)
- **Contract file** (`신청자관리_yyyymm.xlsx`) — `CT_COL`: customerKey(3), manager(13), dbDate(14), applicationDate(15), category(16), mainGift(21), undeliveredGift(22), gift(23), distributeDate(41), returnDate(42), brand(71)

Key logic:
- **Join**: LEFT JOIN on customerKey (DB col 6 ↔ Contract col 3)
- **Cancel logic**: contract row where returnDate (col 42) has any date value
- **DB accumulation**: DB files are accumulated by date in IndexedDB; roster and contract overwrite
- **Holiday handling**: `KOREAN_HOLIDAYS` set + weekend check in `isHoliday()`
- **Roster exclusion**: `ROSTER_EXCLUDE` set of names to skip

## Key Code Sections (in index.html)

| Line Range | Section |
|---|---|
| 1–230 | HTML head, CSS styles |
| 230–460 | HTML body (modals, filters, tab panels, download buttons) |
| 462–524 | Global state variables, column constants (`DB_COL`, `CT_COL`, `ROSTER_COL`), holidays |
| 525–575 | Utility functions (date parsing, formatting) |
| 577–630 | Auth system (credential check, sessionStorage) |
| 633–710 | Tab switching, quick-nav (click + scroll spy) |
| 710–758 | Excel file reading (`readExcel()`) |
| 760–1450 | IndexedDB operations (open, load, accumulate, clear, file metadata, archive) |
| 1453–1490 | Roster parsing |
| 1491–1614 | Status messages, last update display, drag & drop, file upload handlers |
| 1615–1950 | Filter system (dropdowns, custom calendar, date range picker) |
| 1950–1970 | Roster selection by date (`getRosterForDate()`) |
| 1972–2408 | `tryBuildTab4()` — main dashboard builder (grouping, KPI rows, table rendering) |
| 2410–2555 | Export functions (PNG, HTML, Excel download) |
| 2557–2898 | Accumulated data storage/export (localStorage) |
| 2900–3160 | GitHub sync (save/load shared data via GitHub API) |
| 3163–3345 | Favicon, initialization (`DOMContentLoaded`), floating FAB (group nav + TOP button) |

## Running

Open `index.html` in a browser, or use VSCode Live Server (right-click → Open with Live Server).

## Tab System

- **뇌새김 (main)**: Default tab, shows 뇌새김 group performance (W/E/N/제2외국어 groups)
- **톡이즈 (talkis)**: Shows 톡이즈 group performance
- Quick-nav pills scroll to specific group sections within each tab; scroll spy highlights active group

## Language

The project owner works in Korean. All UI labels, data, and comments are in Korean. Communicate in Korean when the user writes in Korean.
