# Website Plan — Maisha Farzana

## What I have

- **CV** (extracted from DOCX): BEng EE (MMU, 2023–2026, predicted 1st), internship at BYK Additives, three projects.
- **Dissertation** (`An electro-thermal model...`): Full 10-section report with LaTeX equations, a physics-based battery model solved with forward Euler, three aging metrics.
- **Python source** (`battery.py`): Pure NumPy — CC-CV current profile, Euler solver, analytical solver, SEI calendar-aging metric, peak temperature, time above threshold.
- **Demo notebook** (`demo.ipynb`): Shows the plots she generates: current profiles, temperature profiles, aging vs C-rate sensitivity over h.

---

## Proposed website structure

Five pages (or sections in a single-page layout):

| Page | Content |
|------|---------|
| **Home / About** | Short bio, photo placeholder, nav links |
| **CV** | Education, experience, projects, skills — a clean web version of the CV |
| **Dissertation** | Written summary of the dissertation + two interactive simulation widgets |
| **Contact** | Email + LinkedIn |

Single-page with anchor navigation is cleaner for a personal site of this size, but separate HTML files are easier to maintain. I'll go with **separate HTML files** (`index.html`, `cv.html`, `dissertation.html`, `contact.html`) sharing a common header/footer.

---

## Tech stack

| Layer | Choice | Reason |
|-------|--------|--------|
| HTML/CSS/JS | Vanilla (no framework) | Simplest for GitHub Pages; no build step |
| Styling | Hand-written CSS with CSS variables | Avoids CDN dependencies; small file size |
| Plots | **Plotly.js** (CDN) | Publication-quality interactive plots; good mobile support |
| Python runtime | **Pyodide** (CDN) | Runs battery.py in-browser via WebAssembly; supports NumPy; no server needed |
| Math rendering | **KaTeX** (CDN) | Lightweight; renders the key equations from the dissertation |
| Hosting | **GitHub Pages** | Free static hosting |

Pyodide is the central design decision — it lets the site run the actual Python simulation code without any backend. The tradeoff is a ~30 MB first load. This is mitigated by: (1) only loading Pyodide on the dissertation page; (2) showing a clear "Loading simulation..." progress indicator; (3) Pyodide is CDN-cached after the first visit.

---

## Dissertation page — interactive widgets

### Widget 1: Charging explorer
User sets **charging rate C** (slider, 0.1–5.0 C) and optionally **heat transfer coefficient h** (slider, 7–56 W/m²K). Clicking "Run" calls into Pyodide:

```
cc_cv_profile → euler_solve_battery → [current, temperature, aging] → Plotly
```

Outputs: 
- Left panel: current vs time (CC and CV phases clearly annotated)
- Right panel: temperature vs time
- Metric cards below: peak temperature (°C), time above 35°C (min), calendar capacity loss (dimensionless)

### Widget 2: Aging vs C-rate sweep
User selects **h** from a dropdown (4 preset values: 7, 14, 28, 56 W/m²K) and clicks "Run sweep". Runs 40 C-rate values and plots **calendar capacity loss vs C-rate**. This reproduces Figure 6 from the dissertation interactively.

The sweep takes ~1–2 seconds in Pyodide because it loops over 40 C-rates — acceptable with a spinner.

---

## GitHub Pages setup

GitHub Pages serves a static site from a repository. Steps:
1. Initialise a git repo in `maisha_website/`
2. Push to a GitHub repository (see authorship note below)
3. Enable GitHub Pages in the repo settings (serve from `main` branch, root `/`)
4. Site will be live at `https://<username>.github.io/<repo-name>/`

Optionally add a custom domain (e.g. `maishafarzana.com`) via a `CNAME` file if desired later — no code changes needed.

---

## Questions for you

1. **GitHub account**: Whose account should the repo live under? Options:
   - Create a new GitHub account for Maisha (cleanest — URL is `maisha-farzana.github.io`)
   - Use an existing account (URL will contain that username unless a custom domain is added)
   - A repo named `<username>.github.io` gives a clean root URL; any other name appends `/repo-name/`

2. **Photo**: Does she have a photo she'd like on the homepage? If not I'll use a clean placeholder.

3. **LinkedIn URL**: The CV lists "LinkedIn: Maisha Farzana" but not the URL slug. What is her actual LinkedIn URL?

4. **Colour palette**: Any preference? I'll default to a clean, professional scheme (navy/slate + a warm accent).

5. **Dissertation title treatment**: The dissertation is still "ongoing" per the CV (May 2026 deadline). Should the page present it as a completed work or note it's a final-year project?

---

## Anticipated complications and solutions

### 1. Pyodide load time (~5–10 s on first visit)
**Problem**: First-time visitors will stare at a blank widget for several seconds.  
**Solution**: Show a progress bar with status messages ("Loading Python environment… Loading NumPy…"). Lock the Run button until ready. Cache kicks in on second visit.

### 2. battery.py needs to be fetchable by Pyodide
**Problem**: Pyodide fetches Python files via HTTP. In local development (`file://`) this fails due to CORS.  
**Solution**: (a) Inline the battery.py source as a JS string that Pyodide executes — avoids the fetch entirely, keeps everything in one file. (b) Or use `python -m http.server` for local dev. I'll go with (a) for robustness.

### 3. The aging sweep loop (Widget 2)
**Problem**: 40 C-rate × Euler integration loop runs on the main thread and may block the UI for 2–3 seconds.  
**Solution**: Run it inside a Pyodide Web Worker. This keeps the page responsive. If that turns out complex, a simpler fallback is to just show a spinner — the total time is short enough.

### 4. No mention of Faisal Alam
**Problem**: Git commits record `user.name` and `user.email` from global git config.  
**Solution**: Before the first commit, set repo-local git config:
```bash
git config user.name "Maisha Farzana"
git config user.email "maishafarzana7430@gmail.com"
```
This overrides global config for this repo only. The website content itself has no attribution.

### 5. Mobile responsiveness of Plotly plots
**Problem**: Plotly defaults can be wide and don't always resize on mobile.  
**Solution**: Set `responsive: true` in Plotly config; use CSS `max-width: 100%` on plot containers; stack panels vertically on small screens via media queries.

### 6. KaTeX rendering of dissertation equations
**Problem**: The dissertation has complex multi-line equations. KaTeX supports most of LaTeX but not all macros.  
**Solution**: The key equations (Eqs. 18, 21, 23–25) are standard and KaTeX-compatible. I'll render only the central equations on the dissertation page, not the full appendix.

---

## What I'll build (in order)

1. `css/style.css` — shared design tokens, layout, responsive grid
2. `index.html` — home/about page
3. `cv.html` — full CV layout
4. `dissertation.html` — summary + both widgets (Pyodide + Plotly)
5. `contact.html` — simple contact page
6. Git init + README for GitHub Pages deployment instructions

I can start building as soon as you answer Q1 (GitHub account) and Q3 (LinkedIn URL). Q2 and Q4 are optional — I'll use sensible defaults if unanswered.
