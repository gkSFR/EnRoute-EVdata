# EV Info file naming (source of truth)

Copied from `gray@192.168.1.221:/home/gray/MAP-Based/EV Info`.
The EnRoute app indexes this tree; keep names and layout stable.

## Layout

```
EV Info/
  {make}/                          # manufacturer folder
    {original-nhtsa-or-manual}.pdf
  _nhtsa_erg_manifest.json         # make/model/url catalog used by the app index
  README_NHTSA_DOWNLOADS.md
  nhtsa-err/                       # bulk NHTSA ERR zip; ID filenames, not make/model
  nhtsa/                           # leftover/unmapped NHTSA PDFs
  nfpa/                            # NFPA cards (not make/model)
  _debug/                          # scrape helpers (html/png/js notes)
```

PDFs are **flat inside each make folder**. There is no `model/` or `year/` subdirectory. Model and year are derived at index time.

## Manufacturer folders (`make`)

- Lowercase slug of the NHTSA make string.
- Spaces → underscores (`land_rover`, `volvo_cars`).
- `sanitize()` also replaces `<>:"/\|?*` with `_`.
- Hyphens are kept when they were in the source (`mercedes-benz_cars`, `rolls-royce`, `e-one`).
- One folder contains `&`: `mcneilus_truck_&_manufacturing`.
- Folder `id` is what `/api/ev-rescue/file?make=` uses.
- Display name in the app: `make.replace("_", " ").title()` (so `alfa_romeo` → `Alfa Romeo`).

## PDF filenames

Keep the **remote filename as-is**. The download script takes `Path(url).name` from the NHTSA URL. Do not rename to a local scheme.

Dominant patterns:

| Pattern | Meaning |
|---------|---------|
| `EmergencyResponseGuide-...pdf` | Full ERG (“Guide” in UI) |
| `RescueSheet-...pdf` | Quick rescue sheet (“Rescue Sheet” in UI) |
| `{year}_{model}_...pdf` | Older manual/Tesla/Ford names |
| `DI##-###-####.pdf` | Only inside `nhtsa-err/` (NHTSA document IDs) |

Powertrain suffixes often appear at the end: `_BEV_1`, `_PHEV_1`, `_HEV_1`, `_FCEV_1`.

Label rule in `https_server.py`: if `"rescue"` is in the filename or URL (case-insensitive) → **Rescue Sheet**, else **Guide**.

## Year extraction

From `https_server.py` `_ev_year_from_filename()`:

```text
regex: (20\d{2})(?:[-_](20\d{2}))?
```

- `2013-2014` or `2013_2014` → year range `2013-2014`
- single `2024` → `2024`
- no 20xx match → em dash `—`
- first 20xx in the name wins (so `v0725` is ignored; `2014_Focus_EV_ERG_9-5-2013.pdf` becomes `2014`)

Years 19xx are **not** captured.

## Model names

**Not** parsed from the filename. They come from `_nhtsa_erg_manifest.json`:

```json
{ "make": "acura", "model": "ILX", "url": "https://static.nhtsa.gov/erg/HONDA/EmergencyResponseGuide-Acura-ILX_HEV-2013-2014.pdf" }
```

Index join:

1. `make` → folder `EV Info/{make}/`
2. filename = last path segment of `url`
3. file must exist on disk or the entry is skipped
4. `model` string is used as-is for the UI tree

Files that exist on disk but are **not** in the manifest (manual Tesla/Ford names, `nhtsa-err/`, `nfpa/`) **do not appear** in `/api/ev-rescue/index`.

## API contract

- Index: `GET /api/ev-rescue/index` → `{ manufacturers: [{ id, name, models: [{ name, years: [{ year, pdfs: [{ label, filename }] }] }] }] }`
- File: `GET /api/ev-rescue/file?make={folder}&file={exact-filename}`
- Path on disk: `EV Info/{make}/{filename}`

## When adding files later

1. Put the PDF in `EV Info/{make}/` using the **NHTSA URL basename** (or the existing manual name).
2. If it should show in the app, add a `{ make, model, url }` row to `_nhtsa_erg_manifest.json` whose URL basename matches that file.
3. Do not invent a new folder slug if a make folder already exists.
4. Do not nest by year or model.
