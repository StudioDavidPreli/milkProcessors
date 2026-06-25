# MilkPEP Video Personalisation Tool

A self-service web tool that lets milk processors take a ready-made milk recipe video or image, add their own branded elements, and download a finished promotional asset. Built for MilkPEP (the Milk Processor Education Program).

**Live:** [customizer.milkpep.org](https://customizer.milkpep.org)
**Studio David Preli** — [davidpreli.com](https://davidpreli.com)

> Tagline in the product: *Simply select a milk video or image, add branded elements and download your new promotional asset. It's that easy.*

---

## What it does

MilkPEP members browse a catalog of recipe-based promotional assets (Arepas de Leche y Queso, Brownie Fries, Cereal Milk Latte, Chamomile Moon Milk, and others), pick one, personalise it with their own branding, and export a render they can post. Each asset is backed by an After Effects template; the user's choices are merged into the template and rendered to a downloadable file.

The point is to give processors broadcast-quality, on-brand video without touching After Effects themselves.

## User flow

| Step | Page | What happens |
|------|------|--------------|
| 1. Browse | **Custom Assets** (gallery) | Grid of recipe assets, each with run time, file size, and asset type (Video / Image) |
| 2. Inspect | **Asset Details** | A single asset's preview and metadata before committing to customise |
| 3. Personalise | **Customize Asset** | Branded elements applied to the chosen template |
| 4. Export | (render + download) | The template is rendered and the finished file is delivered |

## Architecture

### Frontend
- **Next.js (App Router)** + React. The build serves from `/_next/static/chunks/app/*`, with standard `layout`, `page`, `error`, and `not-found` route segments.
- Responsive: ships the standard `width=device-width` viewport and a mobile menu.

### Assets and rendering
- Source templates and thumbnails live in **AWS S3** (`s-milkpep-vd-assets`) served over **S3 Transfer Acceleration**.
- Templates are organised as `media/ae_files/templates/<template_id>/`, each with a thumbnail.
- Thumbnails are pre-rendered into sized, quality-tuned variants (for example `Arepas.png.360x360_q85`), so the gallery loads lightweight images rather than full media.
- Final videos are produced from **After Effects** templates, rendered headlessly via **NexRender**. <!-- TODO: confirm render-trigger path (queue, lambda, worker) -->

### Catalog model
Each catalog entry carries:
- `name` (recipe title)
- `runTime` (seconds)
- `fileSize`
- `assetType` (Video / Image)
- `templateId` (maps to the S3 `ae_files/templates/<id>` directory)

## Tech stack

| Layer | Detail |
|-------|--------|
| Framework | Next.js (App Router), React |
| Asset CDN | AWS S3 + Transfer Acceleration (`s-milkpep-vd-assets`) |
| Templates | After Effects project files (`.aep` / template bundles) |
| Render | NexRender (headless After Effects) <!-- TODO: confirm --> |
| Backend / API | _TODO: document (framework, hosting, endpoints)_ |
| Auth | _TODO: document (how members sign in / are scoped)_ |
| Deploy | _TODO: document (host, CI, build command, env)_ |

## Local development

> _TODO: fill in once repo internals are confirmed. Likely shape for a Next.js app:_

```bash
# install
npm install

# run dev server
npm run dev        # http://localhost:3000

# production build
npm run build && npm run start
```

### Environment

> _TODO: list required env vars. Expected, based on the deployed app:_

```
# AWS / asset storage
AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_ASSET_BUCKET=s-milkpep-vd-assets

# render pipeline (NexRender)
# TODO: render worker / queue config
```

## Repo structure

> _TODO: add the real tree. Skeleton for orientation:_

```
app/                 # Next.js App Router routes
  page.tsx           # Custom Assets gallery
  assets/[id]/       # Asset Details
  customize/[id]/    # Customize Asset
components/
lib/                 # S3 / catalog / render helpers
public/
```

## Notes for maintainers

- The gallery is the heaviest surface: it lists every template with a thumbnail. Thumbnails are already served as sized S3 variants, so keep new templates consistent with the `<name>.png.<W>x<H>_q85` convention to avoid loading full-resolution stills.
- Render jobs are AE-driven, so a template change in After Effects propagates to every personalised export of that asset. Version templates deliberately.
- Footer and brand chrome are MilkPEP's (1250 H Street NW, Washington D.C.); update copyright year and policy links there.

---

Studio David Preli — [davidpreli.com](https://davidpreli.com)
© MilkPEP, 2026
