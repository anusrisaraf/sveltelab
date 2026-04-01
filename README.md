# Portfolio (Vis & Society Labs 4–7)

SvelteKit site with shared layout, dark mode, GitHub stats, project scrollytelling, D3 bar charts, and **Lab 7** commit scatterplot + filtered LOC bar chart (`@floating-ui/dom` for tooltips).

## Run locally

```bash
npm install   # installs @floating-ui/dom (Lab 7) and other deps
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

## Meta page (lines of code chart)

The Meta page reads `static/loc.csv`, which is **not** in the repo (it’s in `.gitignore`). Generate it locally when you need it:

```bash
npx elocuent -d static,src -o static/loc.csv
```

On GitHub Actions, `loc.csv` is generated automatically before each build (see `.github/workflows/deploy.yml`), so the deployed Meta page works without committing the file.

## Lab 7 notes

- **Commit links** in `src/routes/meta/+page.svelte`: set `COMMIT_URL_BASE` to your GitHub repo’s commit URL prefix (the handout uses `https://github.com/vis-society/lab-7/commit/`).
- **CI / single commit**: if every line shows the same commit in the deployed chart, uncomment the “Configure git” step in `.github/workflows/deploy.yml` and set your `user.email` / `user.name` (see course handout).

## Build & deploy

```bash
npm run build
npm run preview   # optional: preview production build
```

Deploys to GitHub Pages via the workflow in `.github/workflows/deploy.yml` (push to `main`).

## Project structure

| Path | Purpose |
|------|--------|
| `src/routes/+layout.svelte` | Nav, theme switcher, layout CSS |
| `src/routes/+page.svelte` | Home: bio, GitHub stats, reading list, latest projects |
| `src/routes/projects/+page.svelte` | Projects: bar chart (per year), scrollytelling, project grid |
| `src/routes/meta/+page.svelte` | Meta: commit scatterplot + horizontal bar chart (LOC by language from `loc.csv`) |
| `src/lib/CommitScatter.svelte` | Lab 7: D3 scatter (axes, grid, sqrt radius, tooltip, click selection) |
| `src/lib/Project.svelte` | Project card (title, year, image, description) |
| `src/lib/ProjectNarrative.svelte` | Scrollytelling narrative + sticky project viz |
| `src/lib/Bar.svelte` | Vertical D3 bar chart (projects per year) |
| `src/lib/BarHorizontal.svelte` | Horizontal D3 bar chart (LOC by language) |
| `src/lib/projects.json` | Project data (title, year, image, description, story, url) |
| `static/style.css` | Global styles |

## Files you can delete (optional)

- **`static/assignments/report.css`** — Remove if you don’t use it (e.g. for assignment writeups).
- **`src/routes/a2/+page.svelte`** and **`src/routes/a3/+page.svelte`** — Remove if you no longer need the assignment iframe pages; if you keep them, ensure the layout nav only links to pages you want (nav is in `+layout.svelte`).
