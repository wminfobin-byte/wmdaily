# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A browser-based daily sales performance dashboard (`index.html`) for a multi-group sales organization. Users upload three types of CMS Excel files (roster, DB count, contract count), and the dashboard generates group/individual KPI tables with filtering, sorting, and export — all client-side with no backend.

## Architecture

Single-file web app (`index.html`, ~3300 lines) with no build step or framework:
- **SheetJS (xlsx-js-style)** via CDN for Excel parsing and export
- **html2canvas** via CDN for PNG screenshot export
- **IndexedDB** (`dbPerfDashboard`, version 6) for persistent storage of uploaded data across sessions
- **GitHub API** for optional cloud sync (token stored in localStorage)
- All data processing happens in-browser; sensitive customer data never leaves the browser

## Data Model

Three input files, each with its own column mapping constants:

- **Roster file** (`WM_근태보고_그룹_yyyymm.xls`) — `ROSTER_COL`: role(0), group(1), center(2), team(3), name(4), hireDate(5)
- **DB file** (`WM_고객관리_yyyy-mm-dd.xlsx`) — `DB_COL`: customerKey(6), mediaCode(19), inflowDate(8), obName(7), customerStatus(27)
- **Contract file** (`신청자관리_yyyymm.xlsx`) — `CT_COL`: customerKey(3), manager(13), returnDate(42), mainGift(21), undeliveredGift(22), gift(23), brand(71)

Key logic:
- **Join**: LEFT JOIN on customerKey (DB col 6 ↔ Contract col 3)
- **Cancel logic**: contract row where returnDate (col 42) has any date value
- **DB accumulation**: DB files are accumulated by date in IndexedDB; roster and contract overwrite
- **Holiday handling**: `KOREAN_HOLIDAYS` set + weekend check in `isHoliday()`

## Key Code Sections (in index.html)

| Line Range | Section |
|---|---|
| 1–230 | HTML head, CSS styles |
| 230–450 | HTML body (modals, filters, tab panels) |
| 450–500 | Global state variables and column constants |
| 500–570 | Utility functions (date parsing, formatting) |
| 570–620 | Auth system (simple credential check, sessionStorage) |
| 620–750 | Tab switching, KPI display, Excel reading |
| 750–1050 | IndexedDB operations (open, load, accumulate, clear) |
| 1050–1430 | File metadata, archive, file list management |
| 1430–1500 | Roster parsing |
| 1500–1770 | Filter system (date range picker, calendar) |
| 1770–1930 | Dropdown filters (group, center, team, person) |
| 1930–2360 | `tryBuildTab4()` — main dashboard builder (builds grouped performance tables) |
| 2360–2520 | Export functions (PNG, HTML, Excel download) |
| 2520–2860 | Accumulated data storage and export |
| 2860–3130 | GitHub sync (save/load shared data via GitHub API) |
| 3130–3300 | Initialization, favicon, DOMContentLoaded |

## Running

Open `index.html` in a browser, or use VSCode Live Server (right-click → Open with Live Server).

## Tab System

- **뇌새김 (main)**: Default tab, shows 뇌새김 group performance (W/E/N/어학 groups)
- **톡이즈 (talkis)**: Shows 톡이즈 group performance
- Quick-nav pills scroll to specific group sections within each tab

## Language

The project owner works in Korean. All UI labels, data, and comments are in Korean. Communicate in Korean when the user writes in Korean.
