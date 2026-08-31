# EV Info file naming (locked for recall)

Source of truth on the server: `/home/gray/MAP-Based/EV Info`  
This repo is a copy of that library. The EnRoute app indexes it; do not invent a new layout.

## Layout

```
EV Info/
  {make}/                  manufacturer folder (flat; no model/year subfolders)
    {original-pdf-name}.pdf
  nhtsa-err/               bulk NHTSA ERR zip; ID filenames, not make/model
  nhtsa/                   leftover NHTSA-site PDFs not yet filed by make
  nfpa/                    NFPA reference cards (not vehicle ERGs)
  _debug/                  scrape helpers (html/png/console script)
  _nhtsa_erg_manifest.json  index source: [{make, model, url}, ...]
  README_NHTSA_DOWNLOADS.md
```

`make` is a sanitized slug: lowercase, spaces → `_`. The downloader also replaces `<>:"/\|?*` with `_`. Display name in the app is that slug with underscores turned into spaces, then title-cased (`ford` → `Ford`, `alfa_romeo` → `Alfa Romeo`).

Folder names are **not** nested by model or year. PDFs sit directly in the make folder.

## How the app finds a PDF

1. Read `_nhtsa_erg_manifest.json`.
2. For each entry, take `make` (slug), `model` (as given), and `filename` = last path segment of `url`.
3. Keep the entry only if `EV Info/{make}/{filename}` exists.
4. Year comes from the **filename**, not the folder:
   - regex: `(20\d{2})(?:[-_](20\d{2}))?`
   - `2013-2014` or `2013_2014` → `2013-2014`
   - `2024` → `2024`
   - no 20xx match → `—`
5. Label:
   - filename or URL contains `rescue` (any case) → **Rescue Sheet**
   - otherwise → **Guide**
6. API serve path: `/api/ev-rescue/file?make={make}&file={filename}`  
   File on disk: `EV Info/{make}/{filename}`

The manifest is the model/year grouping. Scanning folders alone does **not** produce model names.

## Filename families (do not rename)

| Family | Pattern | Example |
|--------|---------|---------|
| NHTSA rescue sheet | `RescueSheet-{Make}-{Model}-{years}_{propulsion}_1.pdf` | `RescueSheet-Kia-EV6-MY2026_BEV_1.pdf` |
| NHTSA ERG | `EmergencyResponseGuide-{Make}-{Model}-{years}.pdf` (optional `_BEV_1` / `_PHEV_1` / `_HEV_1` / `_FCEV_1`) | `EmergencyResponseGuide-Lucid-Air-2022-2026.pdf` |
| Older OEM (Ford/Tesla etc.) | `{year(s)}_{Model}_{...}_Emergency_Response_Guide_...pdf` or `...Rescue_Sheet...` | `2021-2025_Mustang_Mach-E_Rescue_Card_v0725.pdf` |
| NHTSA bulk ERR zip | `nhtsa-err/DI{nn}-{nnn}-{nnnn}.pdf` | `DI07-044-4713.pdf` |

Do not normalize, Title-Case, or replace hyphens/underscores. The filename **is** the lookup key.

## Adding a new NHTSA guide

1. Add `{make, model, url}` to `_nhtsa_erg_manifest.json`.
2. Put the PDF at `EV Info/{make}/{url-basename}` using the exact name from the URL.
3. Do not invent a friendlier filename.

## Special cases

- `mcneilus_truck_&_manufacturing` keeps the `&`.
- Mercedes is split: `mercedes-benz_cars` vs `mercedes_benz_vans`.
- `_debug` and `nhtsa-err` are not in the manufacturer index (they are not make folders with matching manifest entries).
- Two files exceed GitHub’s 100 MB blob limit and must stay on Git LFS: `nhtsa-err/DI12-108-5895.pdf`, `nhtsa-err/DI11-045-5595.pdf`.
