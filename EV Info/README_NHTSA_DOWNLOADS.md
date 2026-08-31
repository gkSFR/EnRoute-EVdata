# NHTSA Emergency Response Guides (EV / Hybrid)

## What’s here

- **`nhtsa-err/`** – 83 PDFs from NHTSA’s bulk ERR zip (same content as [NHTSA file downloads](https://www.nhtsa.gov/file-downloads?p=nhtsa/downloads/ERR/)). Filenames are IDs (e.g. `DI07-044-4713.pdf`), not make/model.
- **Your existing folders** (ford, lexus, tesla, toyota, etc.) – Your manually collected guides by manufacturer.

## Getting the full ~636 guides from the website

The live list at [NHTSA Emergency Response Guides](https://www.nhtsa.gov/emergency-response-guides) is blocked for automated access, so the full set is best collected from your browser:

1. Open: https://www.nhtsa.gov/emergency-response-guides  
2. In the page, select “All” or scroll until all guides are loaded.  
3. Open DevTools (F12) → **Console**.  
4. Paste and run the script in **`_debug/RUN_IN_BROWSER_CONSOLE.txt`**.  
5. The script copies a JSON manifest to your clipboard. Save it as:
   - **`EV Info/_nhtsa_erg_manifest.json`**  
6. From the project root, run:
   ```bash
   .venv-erg/bin/python scripts/download_nhtsa_erg_from_manifest.py
   ```
   PDFs will be downloaded into **`EV Info/{make}/`** (e.g. `EV Info/ford/`, `EV Info/tesla/`).

## Scripts

- **`scripts/download_nhtsa_erg_from_manifest.py`** – Reads `_nhtsa_erg_manifest.json` and downloads each PDF into `EV Info/{make}/`. Skips files that already exist.
- **`scripts/download_nhtsa_erg.py`** – Playwright-based scraper; currently blocked by NHTSA in automated environments. Kept for possible use from a non-blocked host.
