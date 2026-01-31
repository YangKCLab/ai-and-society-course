# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 + Vite course website for CS 415/515 "Social Media Data Science Pipelines" at Binghamton University. The site supports multiple semester versions with isolated content per semester.

## Common Commands

```bash
# Frontend development
cd frontend && npm install    # Install dependencies
cd frontend && npm run dev    # Start dev server (auto-opens browser)
cd frontend && npm run build  # Build for production

# Semester management
make new-semester SEMESTER=Spring2026    # Create new semester from template
make validate-versions                   # Validate version config (requires uv)
make copy-syllabus VERSION=Fall2025      # Copy compiled syllabus PDF
```

## Architecture

### Version System
- `versions/config.json` - Registry of all semesters with default, active status, and per-version navigation settings
- `versions/{Semester}/content/` - JSON data files (schedule.json, staff.json, resources.json, home.json, syllabus.pdf)
- `versions/{Semester}/components/Home.vue` - Optional custom home page (falls back to shared component if absent)
- `versions/{Semester}/demos/` - Jupyter notebooks

### Frontend Structure
- `frontend/src/composables/useVersion.js` - Version-aware data loading; provides `loadVersionData(filename)` and `currentVersion`
- `frontend/src/wrappers/HomeWrapper.vue` - Async loader for custom vs default Home components
- `frontend/src/router/` - Dynamic routing with version param (e.g., `/Fall2025/schedule`)
- `frontend/vite.config.js` - Custom plugin copies `versions/` to `dist/` during build

### Key Constants
- Default version is hardcoded in `useVersion.js` (keep in sync with `versions/config.json`)
- GitHub Pages base path: `/ai-and-society-course/`

## Adding a New Semester

1. `make new-semester SEMESTER=Spring2026`
2. Update `versions/config.json` with new version entry
3. Edit JSON files in `versions/Spring2026/content/`
4. Optionally create custom `versions/Spring2026/components/Home.vue`
5. Copy demos and syllabus
6. `make validate-versions`
