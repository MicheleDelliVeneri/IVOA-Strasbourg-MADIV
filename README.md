[![Deploy Slidev to GitHub Pages](https://github.com/MicheleDelliVeneri/IVOA-Strasbourg-MADIV/actions/workflows/deploy.yml/badge.svg?branch=main)](https://github.com/MicheleDelliVeneri/IVOA-Strasbourg-MADIV/actions/workflows/deploy.yml)

# IVOA Strasbourg 2026 — MADIV

Slides for the IVOA Interoperability Meeting in Strasbourg, May 2026.

Talk: **The MADIV Development Study and AI-Assisted Development at SKA Observatory** — how the ESO/SKAO MADIV study uses deep learning for ALMA imaging, why training on real archival data exposes a petabyte-scale data-access gap for the community, and how SKAO is responding through AI-assisted development practices.

Built with [Slidev](https://sli.dev).

## Run locally

```bash
npm install
npm run dev
```

Open <http://localhost:3030>.

## Build a static site

```bash
npm run build
```

Output lands in `dist/`.

## Deploy

`.github/workflows/deploy.yml` builds the deck on every push to `main` and publishes it to GitHub Pages. Enable Pages with **Source: GitHub Actions** in repo settings; the deck will be served from:

<https://micheledelliveneri.github.io/IVOA-Strasbourg-MADIV/>

## Export to PDF

```bash
npm run export
```

Produces `slides-export.pdf`.

## Contents

- `slides.md` — the deck (Markdown + Slidev directives)
- `Abstract.md` — talk abstract submitted to the IVOA Interoperability Meeting
- `package.json` — Slidev dependencies and scripts
- `.github/workflows/deploy.yml` — CI for GitHub Pages

## References

- MADIV proposal — ESO/CFP/129249/AMA ALMA Development Studies 2025
- ESO BRAIN final report — *Bayesian Adaptive Interferometric Image Reconstruction Methods*, Guglielmetti et al. (2024)
- ALMASim — <https://github.com/MicheleDelliVeneri/ALMASim>
- IVOA TAP — <https://www.ivoa.net/documents/TAP/>
- IVOA DataLink 1.1 — <https://www.ivoa.net/documents/DataLink/>
