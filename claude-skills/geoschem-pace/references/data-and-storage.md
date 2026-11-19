# Input data, downloads and storage

## Where things are
- **ExtData:** `/storage/ideas/is-pliu40-0/ExtData` (31 TB free in late 2026).
  - Inventory and caveats for 2022–2025: `extension_2023_2025/INPUT_DATA_STATUS.md`.
- **Met fields:**
  - `GEOS_0.5x0.625_NA/MERRA2` (nested NA, complete 2000–2025).
  - `GEOS_4x5/MERRA2` (global, complete 2000–2025).
  - `GEOS_0.5x0.625/MERRA2` (global 0.5°, 2010–2019 only; used by GCHP).
- **Group-added HEMCO data:**
  - `GFED4/v2026-03` (2022–2025 converted from GFED4.1s SFTP; earlier years symlinked to v2020-02).
  - `CAMD_EGU/v2026-11` (EGU monthly scale factors 2000–2025, plus `source_campd_daily/`).
  - `NEI2016_YRSCALE/v2026-11` (annual factor block plus EPA Trends sources).
- **2025 fills** (from the same dates in 2024, listed in `README_FILLED_2025.txt` in each directory):
  - OFFLINE_DUST v2021-08: September–December 2025.
  - OFFLINE_LIGHTNING v2020-03: all of 2025 (leap day removed, times shifted).
  - VOLCANO v2024-04: after 2025-03-18.
  - When the official data appear, delete the filled files and download the real ones.

## Downloading
- **GEOS-Chem archive:** use the AWS S3 mirror `https://gcgrid.s3.amazonaws.com/<ExtData-relative path>` (list with `?list-type=2&prefix=`, which pages).
  - The WashU portal (geoschemdata.wustl.edu) lacks most 2023–2025 nested met files.
  - S3 contains a few stray files, e.g. `2023/08/MERRA2.20230501.A1.*`. Delete misplaced files after a bulk download.
- **Bulk downloader:** `extension_2023_2025/download/`.
  - `build_manifest.py` lists S3 and skips files already present with the same size.
  - `run_download.sh` runs 8 parallel streams, writes `.part` files, checks sizes, renames, and retries.
  - Run it under `sg pace-is-pliu40-0 -c "umask 002; ..."` when the targets are group-only.
  - Kill a stuck run before rebuilding the manifest. Files that keep failing usually mean an mkdir permission problem, not the network.
- **GFED4.1s:**
  - The public HTTP site (geo.vu.nl) stops at 2023.
  - 2024+ are on the GFED SFTP (`ftp.prd.dip.wur.nl:1022`, directory `GFED4s/`; credentials are posted on globalfiredata.org/data.html). The user authorizes access.
  - Convert with `GFED4/v2026-03/gfed41s_to_hemco.py YYYY beta <dir>` (bit-identical to the official 2023 files).
- **EPA:**
  - CAMPD daily EGU data: `https://api.epa.gov/easey/bulk-files/...` (listing: `api.epa.gov/easey/camd-services/bulk-files?API_KEY=DEMO_KEY`).
  - EPA Air Pollutant Emissions Trends xlsx: openpyxl is not installed, so use `extension_2023_2025/nei_trends/xlsx_read.py`.

## Permissions and ownership
- ExtData contains files owned by many current and former group members. If a directory is not group-writable, ask its owner or the PI to fix the permissions. In the meantime, write with `sg pace-is-pliu40-0 -c "..."` (see SKILL.md).
- Before overwriting an existing file of a different size, compare the contents. In the past these differences were cosmetic (metadata or header whitespace), and the existing file was kept (`rsync --ignore-existing`).
- Shared project directory: `/storage/project/r-pliu40-0/shared` (group `pace-pliu40`, which is pliu40's primary group, so no NFS group issue there).

## Surface PM extraction (for data fusion)
- Script: `/storage/project/r-pliu40-0/pliu40/GEOS-Chem/PM_surface_extract/extract_surface_pm.py`.
  - Output: yearly `GC_surface_PM_YYYY.nc`, with QC flags. The README there documents the variables, QC, nitrate-collapse days and dust caveats.
  - QA tools: `qa_*.py`, `conus_mask.nc` (CONUS plus 8 regions).
