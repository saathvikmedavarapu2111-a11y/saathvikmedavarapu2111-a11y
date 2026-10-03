# Implementation & Audit Report: GitHub Profile README

## 1. Executive Summary
This repository contains the animated, terminal-styled GitHub profile README project for **Medavarapu Saathvik** (`saathvikmedavarapu2111-a11y`). A comprehensive local audit and engineering polish were conducted across all Python source scripts, JSON configurations, generated SVG assets, GitHub Actions workflows, and the profile README.

All reference-author identity leaks were eradicated from the source generators, the contribution data pipeline was investigated, fixed, and verified with live GitHub data, the visual hierarchy of the README was unified into a cohesive terminal aesthetic, and validation tests passed across all components.

---

## 2. Reference Identity Leakage Audit & Fix

### Root Cause
The Python source generators (`scripts/fetch_contributions.py`, `scripts/make_ascii_svg.py`, `scripts/make_info_card.py`) contained hardcoded fallback usernames defaulting to `AiyzoxX` (`os.environ.get("GH_PROFILE_USER", "AiyzoxX")`). When scripts were executed locally or when environment variables were not explicitly injected, strings such as `aiyzoxx@github: ~$ ./portrait.sh`, `aiyzoxx@github:~$ whoami AiyzoxX`, and reference username values were directly baked into the generated SVG assets (`ascii-portrait.svg`, `contrib-heatmap.svg`).

### Source Scripts Modified
1. **`scripts/fetch_contributions.py`**:
   - Replaced fallback default `AiyzoxX` with `saathvikmedavarapu2111-a11y`.
   - Added user-agent header identifying `saathvikmedavarapu2111-a11y`.
2. **`scripts/make_ascii_svg.py`**:
   - Replaced fallback default with `TERMINAL_USER = "saathvik"` and `DISPLAY_NAME = "Medavarapu Saathvik"`.
   - Updated terminal title bar: `saathvik@github: ~$ ./portrait.sh`.
   - Updated bottom status bar prompt: `saathvik@github:~$ whoami Medavarapu Saathvik` with properly positioned blinking cursor.
3. **`scripts/make_info_card.py`**:
   - Replaced fallback default with `TERMINAL_USER = "saathvik"` and `DISPLAY_NAME = "Medavarapu Saathvik"`.
   - Updated terminal title bar: `saathvik@github: ~$ neofetch`.
   - Updated host row: `saathvik@github`.
   - Replaced generic fallback stack with accurate student developer stack (Java, Python, JS, Spring Boot, FastAPI, PostgreSQL, pgvector, AI/RAG/DSA).
4. **`scripts/render_heatmap_svg.py`**:
   - Updated terminal title bar to use `GH_TERMINAL_USER` (`saathvik`): `saathvik@github: ~/contributions --graph`.
   - Added grid integrity verification comparing cell sums to reported totals.
5. **`data/profile_info.json`**:
   - Formatted with explicit `"name": "Medavarapu Saathvik"` and `"terminal_user": "saathvik"`.

---

## 3. Contribution Data Mismatch Investigation & Resolution

### Root Cause Analysis (74 vs 65 vs 67)
1. **Stale Local Snapshot**: The previous `data/contributions.json` in the workspace was generated at `2026-10-03T17:33:16Z` with 65 total contributions.
2. **Public vs Authenticated Visibility**: When a logged-in user views their own GitHub profile on github.com with "Include private contributions" enabled in profile settings, GitHub computes and displays combined public + private contributions (74). However, public unauthenticated requests to `https://github.com/users/saathvikmedavarapu2111-a11y/contributions` exclusively expose public contributions (which reached 67 after recent activity).
3. **Pipeline Verification**: An unauthenticated scraper cannot invent or scrape private contributions without user tokens (which must never be embedded or committed). Live scraping of `https://github.com/users/saathvikmedavarapu2111-a11y/contributions` returns exactly 67 public contributions across 371 calendar cells (53 weeks from 2025-09-28 to 2026-10-03).

### Data Pipeline Enhancements
- **H2 Header Cross-Validation**: `scripts/fetch_contributions.py` now parses GitHub's rendered `<h2>` summary (`"67 contributions in the last year"`) and validates that `sum(day['count'] for day in days) == h2_total`.
- **Tooltip Regex Hardening**: Enhanced regex to accurately capture `N contribution(s)` and `no contributions` across localized or updated GitHub markup.
- **Renderer Cell Assertion**: `scripts/render_heatmap_svg.py` asserts that the exact sum of rendered grid boxes matches `data['total_contributions']`.
- **Dynamic Calculation**: Zero hardcoded totals or fake metrics. Streaks, active days, and best days are computed strictly from real daily contribution data.

---

## 4. Visual Hierarchy & README Design Polish

### Visual Improvements
- **Intro Section**: Added a clean, compact terminal identity block (`saathvik@github ~ $ whoami`) establishing name and focus without becoming an oversized hero element.
- **Contribution Graph**: Terminal command `saathvik@github ~ $ ./contributions.sh` leading into the custom animated SVG heatmap with 860px container width.
- **Side-by-Side Profile**: Terminal command `saathvik@github ~ $ neofetch` leading into balanced 2-column layout:
  - `ascii-portrait.svg`: 370px width &times; 385px height (ratio matched to card).
  - `info-card.svg`: 490px width &times; 385px height.
  - Combined width: `370px + 490px = 860px` (seamlessly aligned with heatmap).
- **Contact Section**: Terminal-styled `saathvik@github ~ $ ls contact/` featuring subtle terminal code links (`GitHub • LinkedIn • Email`) rather than generic third-party badges.
- **Cleanliness**: Zero debugging artifacts, `cat` commands, or EOF markers in `README.md`.

---

## 5. GitHub Actions & CI Discipline
- **`.github/workflows/update-profile-art.yml`**:
  - Sets `GH_PROFILE_USER: saathvikmedavarapu2111-a11y` and `GH_TERMINAL_USER: saathvik`.
  - Installs only lightweight dependencies (`requests`, `beautifulsoup4`) without large ML/vision models (`rembg`, `opencv-python`, `torch`).
  - Commits only `data/contributions.json` and `contrib-heatmap.svg`.

---

## 6. Repository Cleanliness & Security
- **`.gitignore`**: Created comprehensive `.gitignore` preventing `.venv/`, `__pycache__/`, `.DS_Store`, and ML model weights from being tracked.
- **Index Cleanup**: Removed `.DS_Store` and temporary scratch files (`README.C`) from git index.

---

## 7. Validation Results
- [x] Python syntax compilation: All scripts in `scripts/*.py` compile without errors (`python -m py_compile`).
- [x] JSON validation: `data/profile_info.json` and `data/contributions.json` format validated (`json.tool`).
- [x] SVG XML validity: ElementTree parse confirmed valid XML for `ascii-portrait.svg`, `info-card.svg`, and `contrib-heatmap.svg`.
- [x] Calculation check: `sum(d["count"]) == 67 == total_contributions`.
- [x] Global string audit: `0` occurrences of reference-author identities (`aiyzox*`) across the entire repository.
- [x] Sizing & alignment: Verified proportional alignment of ASCII portrait (370x385) and Info Card (490x385) matching Heatmap (860px).
- [x] Local only: No changes pushed to remote git origin.
