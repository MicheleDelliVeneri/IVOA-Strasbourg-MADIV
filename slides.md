---
theme: default
title: The MADIV Development Study and AI-Assisted Development at SKA Observatory
info: |
  ## The MADIV Development Study and AI-Assisted Development at SKA Observatory
  IVOA Interoperability Meeting — Strasbourg 2026
  Michele Delli Veneri (SKAO) on behalf of the MADIV team
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
fonts:
  sans: 'Noto Sans'
  mono: 'Noto Sans Mono'
themeConfig:
  primary: '#E70068'
background: '#ffffff'
---

<div class="flex justify-start mb-4">
  <img src="./images/skao_logo_2021_colour_rgb_2500x1058px_transparent (1).png" class="h-10" alt="SKAO Logo" />
</div>

<div class="text-3xl font-bold leading-tight mt-2" style="color:#070068;">
The MADIV Development Study<br/>
and AI-Assisted Development<br/>
at SKA Observatory
</div>

<div class="text-base mt-6" style="color:#333333;">
Deep learning for ALMA imaging — and the petabyte-scale data-access gap it exposes
</div>

<div class="mt-8 text-sm" style="color:#555555;">
Michele Delli Veneri &nbsp;·&nbsp; SKA Observatory<br/>
IVOA Interoperability Meeting — Strasbourg, 7 – 12 June 2026
</div>

<div class="mt-6 mx-auto max-w-2xl border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-2 text-xs text-left" style="color:#333333;">
  Co-funded by <span class="text-[#E70068] font-semibold">ESO</span> and <span class="text-[#E70068] font-semibold">SKAO</span>. Builds on the ESO BRAIN development study (Guglielmetti et al. 2024) and the <span class="text-[#E70068] font-semibold">ALMASim</span> simulator.
</div>

<div class="abs-br m-6 text-xs" style="color:#777777;">
    CSP Plenary Session on AI
</div>

<!--
20-minute slot. Two narrative beats:
1. MADIV — why deep learning, what BRAIN taught us, why training on real ALMA archival data
   is now mandatory, and the petabyte-scale data-infrastructure gap that opens.
2. AI-assisted development at SKAO — coding agents, agentic pipelines, dev practice (TBD).
-->

---

# Outline

1. **Why now** — ALMA's Wideband Sensitivity Upgrade and the imaging bottleneck
2. **What BRAIN taught us** — `DeepFocus`, `ALMASim`, and the simulation-to-real generalisation gap
3. **MADIV in one slide** — ViT + Metadata-Aware Transformer for direct imaging from visibilities
4. **The three work packages** — data, pipeline, validation
5. **ALMASim today** — library-first, TAP + DataLink under the hood
6. **The data-access gap** — DataLink was not built for PB-scale bulk training sets
7. **A call to the IVOA community** — what *ML-ready* products would have to look like
8. **AI-assisted development at SKAO** — coding agents, agentic pipelines, dev practice

---
layout: section
---

# 1 · The ALMA imaging bottleneck

ALMA 2030, the WSU, and why classical CLEAN is running out of runway

---

# ALMA is about to flood its own pipeline

<div class="grid grid-cols-2 gap-8 text-sm">

<div>

The **Wideband Sensitivity Upgrade (WSU)** quadruples ALMA's instantaneous bandwidth and dramatically increases data volume and cube complexity.

- ALMA today: **~1 TB / day** of science data
- ALMA WSU: **at least an order of magnitude more**, with cubes **two orders of magnitude larger** than today's GB-scale cubes
- Looking further out, the **ALMA 2040** vision pushes the trend further still

Image reconstruction is the choke point. Today's products are dominated by **CLEAN** (`tclean` in CASA) — mature, well-understood, but:

1. **Computationally heavy** — channel-by-channel, scales poorly with cube size
2. **Hard to parallelise** — repeated gridding and Fourier inversions
3. **Morphologically biased** — optimised for point sources; struggles with extended, low-surface-brightness emission
4. **Human-intensive** — parameter tuning, masking, QA inspection per dataset

</div>

<div>

<div class="border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">The ALMA Science Archive already feels the strain</div>
  Mitigation steps applied during pipeline imaging — to keep delivery times reasonable — mean the archive is <b>missing many of the images</b> the pipeline would otherwise produce. <span class="opacity-70">— BRAIN final report, §1</span>
</div>

<div class="mt-3 border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">What the community is asking for</div>
  The ALMA 2030 development roadmap calls for <b>faster, more automated imaging</b> with better handling of extended emission and tighter integration with archival reprocessing.
</div>

<div class="mt-3 text-xs opacity-75">
This is the gap MADIV aims at — an AI imaging pipeline that scales with the WSU data rate and recovers extended emission better than classical CLEAN.
</div>

</div>
</div>

---
layout: section
---

# 2 · What BRAIN taught us

ESO Internal ALMA Development Study (2020–2024) — Guglielmetti, Delli Veneri, Tychoniec et al.

---

# BRAIN in one slide

<div class="grid grid-cols-2 gap-6 text-sm">

<div>

The ESO **BRAIN** study (*Bayesian Reconstruction with Adaptive Image Notion*) explored two complementary AI techniques for ALMA imaging:

- **`RESOLVE`** — astro-statistics, Bayesian imaging via Information Field Theory (NIFTy), with native uncertainty quantification
- **`DeepFocus`** — astro-informatics, a deep-learning **meta-learner** that finds the best 3D detection/deconvolution architecture for the data at hand

Tested on **simulated** ALMA cubes plus **real** data: HL Tau, BR1202, **DSHARP** and **ALCHEMI** Large Programs, RX J1347.5-1145 (SZ effect).

Companion tool: **`ALMASim`** — open-source ALMA simulator integrated with the archive. **`NOISEMPIRE`** — empirical noise model extracted from real ALMA images.

</div>

<div>

<div class="border-l-4 border-emerald-500 bg-emerald-50/40 dark:bg-emerald-900/15 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">DeepFocus headline numbers</div>
  On 1000 simulated ALMA cubes (256×256×128):
  <ul class="mt-1 ml-4 list-disc">
    <li><b>Blobs Finder</b> reconstruction on the whole test set: <b>23 s</b> (1× NVIDIA Tesla K20)</li>
    <li><b>tclean</b> with 200 iterations: <b>4.3 min/cube</b>, ~1.5 h on 50-cube parallel (400 CPUs)</li>
    <li>Net speed-up: <b>~200×</b> on that hardware</li>
  </ul>
</div>

<div class="mt-3 border-l-4 border-[#E70068] bg-[#E70068]/5 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">Residual quality</div>
  Blobs Finder residuals within <b>±1σ</b>; <code>tclean</code> residuals deviate beyond <b>±5σ</b> on the same test set (BRAIN final report, §5.5).
</div>

<div class="mt-3 text-xs opacity-70">
Conclusion: <b>deep learning works</b> for ALMA imaging — when the training distribution matches the test distribution.
</div>

</div>
</div>

---

# But there is a catch

<div class="text-sm">

BRAIN trained `DeepFocus` primarily on simple **simulated** cubes. The reason is uncomfortable but simple — at the time, training on real archival data at scale was effectively impossible and simulations were not capable of matching real data:

</div>

<div class="mt-3 grid grid-cols-2 gap-6 text-xs">

<div class="border-l-4 border-amber-500 pl-3 py-3">
  <div class="font-semibold mb-1">Archive access at scale</div>
  Retrieving sufficient real data "would require <b>thousands of requests via the helpdesk</b> and extensive processing with CASA, averaging <b>1–2 hours per cube</b>, rendering the process impractical."<br/>
  <span class="opacity-70 mt-1 block">— BRAIN final report, §5.5.2</span>
</div>

<div class="border-l-4 border-amber-500 pl-3 py-3">
  <div class="font-semibold mb-1">Simulation noise ≠ real noise</div>
  Real ALMA noise is <b>spatially correlated, non-Gaussian, baseline-dependent</b>. Simulations available to build groud truths miss those patterns.
</div>

<div class="border-l-4 border-amber-500 pl-3 py-3">
  <div class="font-semibold mb-1">Limited morphological diversity</div>
  Real ALMA sources include complex morphological and spectral structures, which were not simulable.
</div>

<div class="border-l-4 border-amber-500 pl-3 py-3">
  <div class="font-semibold mb-1">Models trained on simulations don't generalise</div>
  Models trained on simulation distributions <b>do not transfer reliably</b> to real ALMA cubes. This is the well-known ML failure mode — and it bites hard in radio interferometry.
</div>

</div>

<div class="mt-4 text-sm">

**Lesson:** the next generation of AI imagers has to **train on real archival data at scale**, augmented with physically-faithful simulations — not the other way round.

</div>

---
layout: section
---

# 3 · MADIV — Metadata-Aware Direct Imaging from Visibilities

co-funded by ESO and SKAO &nbsp;·&nbsp; 36 months &nbsp;·&nbsp; starts September 2026

---

# What MADIV is trying to build

<div class="grid grid-cols-2 gap-6 text-sm">

<div>

A **Visual Transformer (ViT)** that reconstructs **3D clean data cubes directly from calibrated visibilities**, conditioned on a **Metadata-Aware Transformer (MAT)** that ingests:

- ALMA **proposal abstracts**
- Observation **metadata** (band, configuration, integration, source class, …)
- **Linked publications** for that programme

…and turns them into priors the ViT can attend to.

Plus an **anomaly-scoring** head: reconstructions that fall outside the learned feature distribution get flagged for human review — calibration errors, instrumental artefacts, or *new astrophysics*.

</div>

<div>

<div class="border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">Why a transformer, not a U-Net?</div>
  <ul class="mt-1 ml-4 list-disc">
    <li>Real spectral structures persist coherently <b>across many channels</b>; noise does not — 3D attention can exploit that, channel-by-channel CNNs cannot</li>
    <li>Adjacent channels are <b>redundant</b>; treating them independently wastes compute</li>
    <li>Self-attention naturally handles <b>long-range</b> spatial-spectral dependencies — extended emission, mosaics, broad lines</li>
  </ul>
</div>

<div class="mt-3 border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">Targets</div>
  Cycle 7 onward, plus Large Programs: <b>DSHARP, ACES, ALMAGAL, ALMA-IMF, ALCHEMI</b>. Public CLEAN products exist for all of these — straightforward A/B comparison.
</div>

</div>
</div>

---

# Simulating Realistic Calibrated Visibilties

```mermaid {scale: 0.45}
flowchart LR
  A[ALMA Archive<br/>Cycle 7+ & LPs] --> B[ALMASim v2]
  B --> D[Calibrated Visibilities]
  D --> C[WSClean]
  C --> W[Real Clean Cubes]
  W --> E[ALMASim]
  E --> F[Simulated Calibrated Visibilities]
  F --> G[Diffusion Model<br/>vis refinement]
  D --> G[Diffusion Model<br/>vis refinement]
  G --> H[Simulated Calibrated Visibilities]
```
<div class="grid grid-cols-[2fr_1fr] gap-6 items-start">

<div>

# Data Enhancement

```mermaid {scale: 0.48}
flowchart LR
  A[Simulated Skymodel] --> B[ALMASim]
  B --> D[Calibrated Visibilities]
  D --> C[Diffusion Model]
  C --> W[Realistic Calibrated Visibilities]
```

# Training the Visual Transformer

```mermaid {scale: 0.48}
flowchart LR
 A[ALMA Archive<br/>Cycle 7+ & LPs] --> I[MAT corpus<br/>abstracts · metadata<br/>· publications]
  A --> B[ALMASim]
  B --> F[Simulated Calibrated Visibilities]
  B --> H[Calibrated Visibilties]
  H --> J((ViT + MAT))
  I --> J
  F --> J
  J --> K[Clean 3D cubes<br/>+ anomaly score]
```

</div>

<div class="text-xs space-y-3 mt-4">

<div class="border-l-2 border-emerald-500 pl-2">
<div class="font-semibold">ALMASim v2</div>
Fetches raw vis, calibration, abstracts, metadata, publications — not just synthetic skies.
</div>

<div class="border-l-2 border-violet-500 pl-2">
<div class="font-semibold">Diffusion refinement</div>
DM makes simulated vis **indistinguishable** from real — physically-plausible augmentation.
</div>

<div class="border-l-2 border-amber-500 pl-2">
<div class="font-semibold">ViT + MAT</div>
Augmented vis + MAT priors feed ViT → clean cubes + anomaly score.
</div>

</div>

</div>

---

# Three work packages, 36 months

<div class="grid grid-cols-3 gap-3 text-xs">

<div class="border-l-4 border-emerald-500 bg-emerald-50/40 dark:bg-emerald-900/15 pl-3 py-3">
  <div class="font-semibold mb-1">WP1 · M1–12</div>
  <div class="opacity-90 mb-2">Enhanced ALMASim + dataset</div>
  <ul class="ml-4 list-disc">
    <li>Archive integration (raw vis, calib, metadata, abstracts, papers)</li>
    <li>Web app (FastAPI + Svelte)</li>
    <li>CLI and Slurm optimization</li>
    <li>Diffusion model refinement</li>
    <li>Physical data augmentation</li>
    <li><b>D1.1</b> ALMASim v2 release</li>
    <li><b>D1.2</b> Public multi-modal dataset</li>
  </ul>
</div>

<div class="border-l-4 border-[#E70068] bg-[#E70068]/5 pl-3 py-3">
  <div class="font-semibold mb-1">WP2 · M10–24</div>
  <div class="opacity-90 mb-2">ViT + MAT pipeline</div>
  <ul class="ml-4 list-disc">
    <li>ViT architecture for visibility-domain input</li>
    <li>3D spatial-spectral attention</li>
    <li>MAT ingestion of abstracts + metadata + papers</li>
    <li>Anomaly scoring module</li>
    <li>End-to-end automated pipeline</li>
    <li><b>D2.1–D2.4</b> components &amp; integrated pipeline</li>
  </ul>
</div>

<div class="border-l-4 border-violet-500 bg-violet-50/40 dark:bg-violet-900/15 pl-3 py-3">
  <div class="font-semibold mb-1">WP3 · M20–36</div>
  <div class="opacity-90 mb-2">Validation &amp; science demos</div>
  <ul class="ml-4 list-disc">
    <li>Training/validation on Cycle 7+ archival data</li>
    <li>Benchmarking vs <code>tclean</code> &amp; other baselines</li>
    <li>Science demonstrations (PPDs, high-z galaxies)</li>
    <li>Readiness for archival mining + RADPS integration</li>
    <li><b>D3.1</b> benchmarks, <b>D3.2</b> demo papers, <b>D3.3</b> prototype</li>
  </ul>
</div>

</div>

<div class="mt-4 text-xs opacity-80">

**Compute backbone.** STILES ADHOC (UniNa Federico II) — 2 PFLOPS steady-state, 12 PB hot disk, 22 × dual-H100 + 12 × dual-L40 GPU nodes

</div>

<div class="mt-3 text-xs opacity-70">

**Why this matters beyond ALMA.** The same architecture, dataset patterns, and AI workflows are transferable to <b>SKA</b> and <b>ngVLA</b> — both will need scalable, automated imaging at PB/yr data rates.

</div>

---
layout: section
---

# 4 · ALMASim today

What we already have — and what WP1 has to upgrade

---

# ALMASim V2

<div class="grid grid-cols-2 gap-6 text-sm">

<div>

ALMASim has been refactored from "GUI + scripts" into a **library-first** package with reusable services:

```
src/almasim/
  services/
    metadata/       ← ALMA TAP query + normalisation
    download/       ← DataLink resolve + parallel pull
    archive/        ← ASDM unpack, applycal
    interferometry/ ← UV sampling, baselines, noise, TP
    imaging/        ← deconvolution, TP+INT combination
    products/       ← MS export, HDF5 shards, cubes
    astro/          ← spectral lines, redshift
    compute/        ← sync / local / Dask / Slurm / k8s
  skymodels/        ← Point, Gaussian, Extended,
                      Galaxy Zoo, Hubble100, Molecular,
                      Diffuse, Illustris TNG
  cli.py            ← typer entrypoint
```

Same staged API drives the **CLI**, a **FastAPI** backend, and Jupyter notebooks.

</div>

<div>

<div class="border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-3 text-xs mb-3">
  <div class="font-semibold mb-1">Why library-first</div>
  WP1 needs ALMASim to be <b>callable</b> from training loops, Dask graphs, Slurm jobs, and a browser-based UI — without rewriting the simulation logic four times.
</div>

<img src="./images/almasim-frontend.png" class="rounded shadow w-full" alt="ALMASim v2 web frontend" />

</div>
</div>

---

# The CLI mirrors the staged pipeline

<div class="text-sm mb-2">

Four sub-apps, one for each stage of the *real-data* workflow. They allow to query, download, calibrate and clean real ALMA data and use it as input for training and for simulating further data with the simulation function.

</div>

```bash
almasim --help

almasim metadata query \
  --science-keyword "Galaxies" --band 6 --array-type 12m \
  --observation-date-range 2019-10-01 2024-09-30 \
  --qa2-status Pass --public-only \
  --out-csv metadata.csv

almasim products resolve   --metadata-csv metadata.csv --out-csv products.csv

almasim products download  --products-csv products.csv --type raw --parallel 8

almasim products unpack    --products-csv products.csv  # ASDM → MS

almasim products calibrate --products-csv products.csv  # applycal (casatasks)

almasim simulation run     --skymodel point --backend local --save-format h5
```

<div class="mt-3 grid grid-cols-2 gap-3 text-xs">
  <div class="border-l-4 border-emerald-500 pl-3 py-2">
    <b>metadata</b> → <b>products</b> → <b>simulation</b> / <b>image</b> mirrors what BRAIN had to do by hand: query, locate, fetch, unpack, calibrate, then use.
  </div>
  <div class="border-l-4 border-[#E70068] pl-3 py-2">
    <code>products resolve</code> is the one that takes us into <b>IVOA DataLink</b> — and where the trouble starts at PB scale.
  </div>
</div>

---

# Under the hood — ALMASim talks TAP and DataLink

<div class="grid grid-cols-[1.4fr_1fr] gap-6 items-start">

<div>

```mermaid {scale: 0.36}
sequenceDiagram
  participant U as User / training loop
  participant AS as ALMASim
  participant TAP as ALMA TAP<br/>(ESO / NRAO / NAOJ)
  participant DL as ALMA DataLink<br/>(per-mirror)
  participant RSE as ALMA mirror<br/>(HTTP / FTP)
  participant CASA as CASA tooling

  U->>AS: metadata query (Inclusion/Exclusion filters)
  AS->>TAP: ADQL via pyvo
  TAP-->>AS: rows (member_ous_uid, freq, beam, …)
  Note over AS,DL: products resolve
  loop one UID at a time
    AS->>DL: GET /datalink/sync?ID=<uid>
    DL-->>AS: VOTable rows (access_url, semantics, size)
  end
  AS->>AS: classify + filter (raw / cont / cube / cal / …)
  U->>AS: products download
  AS->>RSE: parallel pulls (cap per mirror)
  AS->>CASA: importasdm → applycal
  CASA-->>U: calibrated MeasurementSet
```

</div>

<div class="text-xs space-y-3">

<div>
<div class="font-semibold text-slate-800 dark:text-slate-100">1 · Discovery via TAP</div>
<div class="opacity-80 mt-0.5"><code>pyvo.dal.TAPService</code> across three mirrors. ALMASim retries each. Filters (science keyword, band, array type, FOV, QA2, date) are translated to ADQL and normalised against stable application fields.</div>
</div>

<div class="border-l-2 border-[#E70068] pl-2">
<div class="font-semibold">2 · Resolution via DataLink</div>
<div class="opacity-80 mt-0.5">For every <code>member_ous_uid</code>: <code>GET /datalink/sync?ID=&lt;uid&gt;</code>. Parse the VOTable. Each row → <code>(access_url, content_length, content_type, semantics)</code>. Classify into <code>raw / continuum / cube / calibration / preview / other</code>.</div>
</div>

<div class="border-l-2 border-emerald-500 pl-2">
<div class="font-semibold">3 · Bulk download</div>
<div class="opacity-80 mt-0.5">Parallel pulls across mirrors, with per-mirror and global concurrency caps (rclone-style backpressure).</div>
</div>

<div class="border-l-2 border-amber-500 pl-2">
<div class="font-semibold">4 · ASDM → MS → applycal</div>
<div class="opacity-80 mt-0.5">CASA <code>importasdm</code>, then <code>applycal</code> with delivered calibration tables. <b>This step is where the wheels start coming off.</b></div>
</div>

</div>

</div>

---
layout: section
---

# 5 · The data-access gap

DataLink was not designed for tranfering TBs of data for DL training — it was designed for *retrieval* of observations 

---

# What works, what doesn't

<div class="grid grid-cols-2 gap-6 text-xs">

<div>

**What works well today**

- **TAP / ObsCore** discovery — `pyvo` + ADQL covers science-keyword-filtered cohort builds
- **DataLink** resolves an OUS UID to its file list — VOTable, `access_url`, `semantics`, `content_length`
- File-level HTTP / FTP transport at single-file scale

**What stops working at MADIV scale**

- Need **thousands** of OUSs from Cycle 7+ — not one dataset
- **Tens to hundreds of TB** of raw vis + cal tables; **PB-scale** across all Cycles
- Sustained bulk transfer is **not what DataLink was designed for**

</div>

<div>

<div class="border-l-4 border-amber-500 bg-amber-50/40 dark:bg-amber-900/15 pl-4 py-2 text-xs">
  <div class="font-semibold mb-1">Three concrete pain points</div>
  <ol class="mt-1 ml-4 list-decimal space-y-0.5">
    <li><b>Per-product round trips.</b> One <code>datalink/sync</code> call per UID; one fetch per file. Thousands of OUSs × dozens of files = massive round-trip count against a small mirror set.</li>
    <li><b>No bulk-transfer mode.</b> No standard way to say "give me all raw vis for this OUS list in one negotiated transfer."</li>
    <li><b>Mirror-side rate caps.</b> ALMASim caps per-mirror concurrency — mirrors are <b>shared infrastructure</b>; we cannot saturate them.</li>
  </ol>
</div>


</div>
</div>

---

# Raw visibilities are not training data

<div class="grid grid-cols-2 gap-6 text-xs">

<div>

<div class="text-sm mb-2">Even if we transfer everything, what we get is <b>not</b> ML-ready. ALMASim runs this for every OUS, every time:</div>

```mermaid {scale: 0.6}
flowchart LR
  A[ASDM<br/>raw] --> B[importasdm]
  B --> C[MS<br/>uncalibrated]
  C --> D[applycal]
  D --> E[Calibrated MS]
  E --> F[UV grid / FFT]
  F --> G[Dirty cube +<br/>vis + mask]
  G --> H[HDF5 shard<br/>ML-ready]
```

</div>

</div>

<div class="space-y-2">

<div class="border-l-4 border-[#E70068] pl-3 py-1.5">
  <b>importasdm.</b> CASA-only. Linux x86-64 only. 10s–100s GB per OUS for big LPs.
</div>

<div class="border-l-4 border-[#E70068] pl-3 py-1.5">
  <b>applycal.</b> Cal tables ship with the delivery. CASA-only. Failure modes are dataset-specific.
</div>

<div class="border-l-4 border-[#E70068] pl-3 py-1.5">
  <b>UV gridding + FFT.</b> Memory scales with Nx · Ny · Nch; mosaics multiply by Nfields.
</div>

<div class="border-l-4 border-amber-500 pl-3 py-1.5">
  <b>Domain expertise required.</b> All steps above are radio-interferometric specialist work — most ML groups don't have it, and don't need it if products arrive ML-ready.
</div>

<div class="border-l-4 border-slate-400 pl-3 py-1.5">
  <b>The cost is duplicated.</b> Every group repeats the same calibration, gridding, and HDF5 sharding against the same archive. Compute and bandwidth paid many times over.
</div>


</div>

---
layout: section
---

# 6 · A call to the IVOA community

What would *ML-ready* delivery look like — and why it benefits more than MADIV

---

# Bulk + processed + standardised

<div class="grid grid-cols-2 gap-6 text-sm">

<div>

**The asks, concretely**

1. **A bulk-transfer profile** for DataLink-resolved products — one negotiated session, many files, with backpressure and resume. Whether that's an extension of DataLink, a SODA shape, or a sibling service is a question for this room.
2. **A processed-products tier**. Calibrated MS, dirty cubes, UV masks — *delivered*, not reconstructed by every consumer. The archive runs the pipeline once, the community trains many models. Maybe a new standard service for processed products that can introduced within Datalink Service Descriptor?
3. **A standardised "ML-ready" container**. HDF5 (or Zarr) shards with explicit conventions fore.

</div>

<div>

<div class="border-l-4 border-emerald-500 bg-emerald-50/40 dark:bg-emerald-900/15 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">Why this is bigger than MADIV</div>
  <ul class="mt-1 ml-4 list-disc">
    <li><b>SKA Regional Centres</b> will face exactly this — only at <b>700 PB/yr</b> rather than 1 TB/day</li>
    <li><b>ngVLA</b> sits in the same regime</li>
    <li>Every <b>ML-for-radio</b> group is building a private version of the ALMASim pipeline today; that work is fungible</li>
    <li>This is the IVOA <b>natural problem space</b> — discovery, access, transport, semantics — applied to a use case (large-scale training) the standards weren't originally written for</li>
  </ul>
</div>

<div class="mt-3 border-l-4 border-emerald-500 bg-emerald-50/40 dark:bg-emerald-900/15 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">Sister work</div>
  The <b>SRCNet Product Streamer</b> (separate IVOA session, DSP WG) shows one way to negotiate dataset-level transport over DataLink for SKA. The MADIV ask is the same shape — just upstream, at ALMA.
</div>

</div>
</div>

---

# What we hope to bring back from this meeting


- **A signal** on whether a *bulk transfer + processed tier* could be shipped alongside Datalink as a new IVOA standardized service.
- **A community-curated, standards-conformant tier** — even at modest scale, this would change what BRAIN-style studies can realistically attempt.


<div class="mt-6 p-4 bg-[#E70068]/8 border-l-4 border-[#E70068] text-sm">
Providing ML ready data products and services is fundamentally  a <b>community infrastructure problem</b>. MADIV is one of the projects that has to solve it for itself — but every group will hit it. Let's solve it once.
</div>

---
layout: section
---

# 7 · AI-assisted development at SKAO

How we are building MADIV — coding agents, agentic pipelines, dev practice

---

# SKAO AI Policy — the governance frame

<div class="grid grid-cols-2 gap-6 text-xs">

<div>

SKAO's AI use is governed by **SKAO-GOV-0000166** (Rev 01, 2024). Key rules for development teams:

<div class="mt-2 space-y-2">

<div class="border-l-4 border-emerald-500 pl-3 py-1.5">
  <b>Science data excluded.</b> §3.3 — AI solutions used solely for science data are out of scope. MADIV abides to this regulation and infact it is exploring only on ALMA data.
</div>

<div class="border-l-4 border-[#E70068] pl-3 py-1.5">
  <b>Only Unrestricted data in AI tools.</b> §5.5 privacy control — no restricted, personal, or special-category data may enter any AI system.
</div>

<div class="border-l-4 border-amber-500 pl-3 py-1.5">
  <b>Code generation = Moderate risk.</b> §5.3.7 — all AI-generated code must follow SKA software quality assurance: human review + testing before merge.
</div>

<div class="border-l-4 border-violet-500 pl-3 py-1.5">
  <b>New tool → IT helpdesk ticket.</b> §5.2.2 — any third-party AI tool that processes SKAO data must be assessed by IT + InfoSec before use.
</div>

</div>

</div>

<div>

<img src="./images/ai-dashboard.png" class="rounded shadow w-full" alt="SKAO AI governance dashboard" />

<div class="mt-2 text-xs opacity-70 text-center">SKAO AI adoption governance dashboard — budget, approvals, lifecycle tracking</div>

</div>

</div>

---

# AI Tools employed at the SKA Observatory

<div class="grid grid-cols-2 gap-6 text-sm">

<div>

<div class="border-l-4 border-[#E70068] bg-[#E70068]/8 pl-4 py-3 text-xs mb-3">
  <div class="font-semibold mb-1">Claude Code / GPT  — agentic coding assistant</div>
  <ul class="ml-4 list-disc space-y-0.5">
    <li>Used for most software development at the SKA Observatory</li>
    <li>Used for ALMASim v2 refactoring — library-first architecture, service decomposition</li>
    <li>FastAPI + Svelte UI scaffolding and iteration</li>
    <li>Simulation pipeline code, HDF5 shard format, CLI wiring</li>
    <li>Slides and documentation drafting (this talk!)</li>
  </ul>
</div>

<div class="border-l-4 border-emerald-500 bg-emerald-50/40 dark:bg-emerald-900/15 pl-4 py-3 text-xs">
  <div class="font-semibold mb-1">Greptile — AI code review agent</div>
  <ul class="ml-4 list-disc space-y-0.5">
    <li>Reviews PRs automatically across all SKAO repositories</li>
    <li>Flags critical bugs, security issues, and regressions before human review</li>
    <li>Complements — does not replace — mandatory human review (per §5.3.7)</li>
  </ul>
</div>

</div>

<div>

<div class="text-xs font-semibold mb-1 opacity-70">Top models by premium requests — SKAO</div>
<img src="./images/image.png" class="rounded shadow w-full" alt="Top AI models at SKAO" />

</div>

</div>

---

# Greptile in numbers — one month across 15 repos

<div class="grid grid-cols-[1fr_1.2fr] gap-6 items-start">

<div class="text-xs space-y-3">

<div class="border-l-4 border-emerald-500 bg-emerald-50/40 dark:bg-emerald-900/15 pl-3 py-2">
  <div class="text-2xl font-bold text-emerald-600">668</div>
  <div class="font-semibold">PRs reviewed by Greptile</div>
  <div class="opacity-70">+5467% vs baseline period — adoption ramped fast once tool was approved</div>
</div>

<div class="border-l-4 border-[#E70068] bg-[#E70068]/8 pl-3 py-2">
  <div class="text-2xl font-bold" style="color:#E70068">233</div>
  <div class="font-semibold">Critical bugs caught</div>
  <div class="opacity-70">Across 15 repos — before human review, before merge</div>
</div>

<div class="border-l-4 border-amber-500 bg-amber-50/40 dark:bg-amber-900/15 pl-3 py-2">
  <div class="text-2xl font-bold text-amber-600">5.4 d</div>
  <div class="font-semibold">Avg merge time</div>
  <div class="opacity-70">−71.52% — faster review cycle with AI first pass</div>
</div>

<div class="border-l-4 border-slate-400 pl-3 py-2">
  <div class="font-semibold">Addressed rate: 30%</div>
  <div class="opacity-70">Of Greptile comments acted on — selective, not mechanical. Engineers remain in the loop.</div>
</div>

</div>

<div>
<img src="./images/greptile.png" class="rounded shadow w-full" alt="Greptile analytics dashboard" />
</div>

</div>

---

# Where the AI suggestions land — by language

<div class="grid grid-cols-[1.2fr_1fr] gap-6 items-start">

<div>
<img src="./images/top-languages.png" class="rounded shadow w-full" alt="Top languages by accepted AI suggestions" />
</div>

<div class="text-xs space-y-3">

<div class="border-l-4 border-[#E70068] pl-3 py-2">
  <div class="font-semibold">Python — 18.4k accepted suggestions</div>
  SKAO's primary science and pipeline language. Control software, data pipelines — all Python. AI suggestions accepted at very high rate.
</div>

<div class="border-l-4 border-emerald-500 pl-3 py-2">
  <div class="font-semibold">C++ — 2.1k</div>
  Low-level telescope control and signal processing. AI still useful for boilerplate, but expert review is heavier here.
</div>

<div class="border-l-4 border-amber-500 pl-3 py-2">
  <div class="font-semibold">Shell, RST, YAML, Dockerfile — 500–1.2k each</div>
  Infrastructure, CI, docs — AI is very effective for these. Low domain-knowledge threshold = high acceptance rate.
</div>

<div class="border-l-4 border-violet-500 pl-3 py-2">
  <div class="font-semibold">TypeScript / React — 635 combined</div>
  Growing fast as the browser-based pipeline interface matures.
</div>

</div>

</div>

---

# What works, what doesn't — honest assessment

<div class="grid grid-cols-2 gap-6 text-xs">

<div>

**Works well**

<div class="space-y-2 mt-2">
<div class="border-l-2 border-emerald-500 pl-2">
  <b>Refactoring to patterns.</b> Turning monolithic scripts into library-first services — AI excels when the target structure is clear and the code is Unrestricted.
</div>
<div class="border-l-2 border-emerald-500 pl-2">
  <b>Test and CLI scaffolding.</b> Generating pytest fixtures, typer CLI wiring, FastAPI route stubs. High acceptance, low risk.
</div>
<div class="border-l-2 border-emerald-500 pl-2">
  <b>PR first-pass review.</b> Greptile catches regressions and security issues that reviewers miss under time pressure.
</div>
<div class="border-l-2 border-emerald-500 pl-2">
  <b>Docs and RST.</b> API docs, Sphinx stubs, README updates — large time saving, easy to verify.
</div>
</div>

</div>
<div>

**Doesn't work / guardrails**

<div class="space-y-2 mt-2">
<div class="border-l-2 border-[#E70068] pl-2">
  <b>Domain physics.</b> AI cannot check that UV gridding parameters are scientifically correct. Expert review mandatory.
</div>
<div class="border-l-2 border-[#E70068] pl-2">
  <b>Radio Interferometric Software internals.</b> Specialized software and edge cases require hands-on debugging — AI suggestions are often plausible but wrong.
</div>
<div class="border-l-2 border-amber-500 pl-2">
  <b>Never use restricted data.</b> SKAO policy §5.5 — only Unrestricted data enters any AI tool. Science archive credentials, proprietary metadata: never in prompts.
</div>
<div class="border-l-2 border-amber-500 pl-2">
  <b>All generated code is reviewed.</b> Per §5.3.7 — Moderate risk means mandatory QA, not optional. No AI code merges without a human sign-off.
</div>
</div>

</div>

</div>

<div class="mt-3 p-3 bg-[#E70068]/8 border-l-4 border-[#E70068] text-xs">
  Net verdict: AI coding assistance multiplies throughput on well-understood engineering tasks. It does not substitute for domain expertise in radio interferometry, and SKAO's policy framework makes the boundary explicit.
</div>

---

# A Radio-Expert LLM for SKAO — the vision

<div class="grid grid-cols-2 gap-6 text-xs">

<div>

**What we are planning to build**

<div class="space-y-2 mt-2">
<div class="border-l-2 border-[#E70068] pl-2">
  <b>A domain-fine-tuned LLM</b> trained on radio-interferometric literature, SKAO software repositories, and ALMA/SKA pipeline code — a model that "speaks" radio astronomy natively.
</div>
<div class="border-l-2 border-[#E70068] pl-2">
  <b>Code assistant.</b> Suggest and generate code and fixes with observatory-level context a general model lacks.
</div>
<div class="border-l-2 border-[#E70068] pl-2">
  <b>ADQL / TAP copilot.</b> Natural-language → ADQL translation, context-aware query validation against ObsCore and schemas, interactive refinement before submission.
</div>
<div class="border-l-2 border-[#E70068] pl-2">
  <b>SRCNet & data-access interface.</b> Abstract DataLink resolution, bulk-transfer negotiation, and product classification behind a conversational layer — lower the barrier for non-expert users.
</div>
</div>

</div>

<div>

**What works / what doesn't**

<div class="space-y-2 mt-2">
<div class="border-l-2 border-emerald-500 pl-2">
  <b>Training corpus exists.</b> SKAO GitLab repos, arXiv radio-astro papers — all public, all Unrestricted. Enough text to fine-tune a mid-size open model.
</div>
<div class="border-l-2 border-emerald-500 pl-2">
  <b>ADQL generation is tractable.</b> Schema-constrained generation + retrieval-augmented prompting over TAP metadata already works well with current general LLMs — a domain model would improve reliability further.
</div>
<div class="border-l-2 border-amber-500 pl-2">
  <b>Evaluation is hard.</b> No benchmark exists for "correct radio-interferometric advice." We would have to build one — expert-curated Q&A pairs, ADQL correctness harness, code execution tests.
</div>
<div class="border-l-2 border-slate-400 pl-2">
  <b>Governance.</b> Deploying an LLM as an operational tool inside SRCNet requires SKAO IT/InfoSec assessment — same §5.2.2 path as any third-party tool, even if self-hosted.
</div>
</div>

</div>

</div>

<div class="mt-3 p-3 bg-[#E70068]/8 border-l-4 border-[#E70068] text-xs">
  Long-term goal: a self-hosted Radio Expert LLM that lowers the expertise barrier for TAP, DataLink, and SRCNet — making archival science accessible to non-specialist users without compromising data governance.
</div>

---
layout: center
class: text-center
---

# Thank you

<div class="text-base opacity-80 mt-6">
MADIV proposal &nbsp;·&nbsp; <code>ESO/CFP/129249/AMA</code><br/>
ALMASim &nbsp;·&nbsp; <code>github.com/MicheleDelliVeneri/ALMASim</code><br/>
BRAIN final report &nbsp;·&nbsp; Guglielmetti et al. 2024<br/>
Slides &nbsp;·&nbsp; <code>github.com/MicheleDelliVeneri/IVOA-Strasbourg-MADIV</code>
</div>

<div class="mt-10 text-sm opacity-60">
michele.delliveneri@skao.int &nbsp;·&nbsp; SKA Observatory
</div>
