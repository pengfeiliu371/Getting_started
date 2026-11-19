# GEOS-Chem Classic 12.9.3 — Lifei Yin's Nested_C1 complex-SOA setup

## Contents
1. What the Nested_C1 record is
2. Code and build
3. Run directories and HEMCO_Config generation
4. Segment workflow (chunked monthly jobs, restarts, QC)
5. Emission scaling (NEI2016 + CAMD)
6. Known model issues in the existing record

## 1. What the Nested_C1 record is
- Output (read-only): `/storage/project/r-pliu40-0/lyin61/Climate_Penalty_GC/GCoutput/Nested_C1_complex_coat_NEI16/OutputDir/<year>/`
  - 11 daily collections (AerosolMass, Aerosols, Budget, DryDep, ProdLoss, RxnRates, SpeciesConc, StateChm, StateMet, WetLossConv, WetLossLS) plus HEMCO_diagnostics, 2000-01 to 2022-11.
- Restarts: `.../Nested_C1_complex_coat_NEI16/Restart/1st_day/GEOSChem.Restart.YYYYMM01_0000z.nc4`
  - Every month 2000-01 to 2022-12; no HEMCO restarts.
  - Some are on the 121×161 grid, others 121×145; HEMCO crops on read, so all are usable.
- Global 4×5 BC provider: `.../GCoutput/complex_4_5_NEI16_monthly/`
  - Monthly OutputDir and monthly Restart/ files are kept.
  - The 3-hourly GC_BCs were deleted; only the `GC_BCs/1stday` snapshots remain.
  - Its run directory was deleted; it was rebuilt from UT.12.9.3 (see section 3).
- Paper: Yin et al. 2026 ACP 26:5901.
- Nested domain varied over time (121×161 / 121×145 / 111×145). New runs use 121×145: 10–70N, 140–50W.
- Common analysis domain: 10–65N, 140–50W.

## 2. Code and build
- Production code: `lyin61/GC_ComplexSOA/Code_final`, confirmed by Lifei. Working copy with the LWC/pH diagnostic fix:
  `/storage/project/r-pliu40-0/pliu40/GEOS-Chem/extension_2023_2025/code/Code_final_LY`
  - Fix in `Headers/state_chm_mod.F90`: register `phSav`/`HplusSav`/`WaterSav` for N=1 only (`DO N = 1, 1`). Upstream 12.9.3 registers both categories under the same name, so the output silently holds coarse sea-salt values (pH ~3–6, LWC ~0). This changes diagnostics only, not physics.
  - Lifei's code copies differ functionally (HetRates IEPOX coating area, KPP error handling above level 21, NaN reset in wetscav). See `extension_2023_2025/CODE_VARIANTS.md`.
- Environment: `source /storage/project/r-pliu40-0/pliu40/GEOS-Chem/extension_2023_2025/code/gc1293_intel.env`
  - Provides Intel 2021.9 plus the Intel-built NetCDF 4.9.2 / HDF5 1.14.5 from the old shared spack.
  - Paths are set explicitly because the spack DB and `nf-config` still print `/storage/coda1`. `LIBRARY_PATH` is set so the linker finds `-lnetcdff`.
  - bzhang's `shared/GEOS-Chem/slurm_rh9/.GC` is broken (old coda1 SPACK_ROOT); do not rely on it.
- Build (GNU make, ~15 min, same flags as Lifei: `-O2 -fp-model source -qopenmp`):
  `extension_2023_2025/code/build_gc1293.sh <rundir>`, which runs `make realclean; make -j8 build OMP=y`.
  - The `build` target deletes `*.log` in the run directory. The real log is `compile.log`.
- Grid is chosen at run time (`input.geos`), so one `geos` executable serves both global and nested runs.
- The old RHEL7 `geos` binaries do not run on RHEL9 (missing Intel runtime libs), so always rebuild.

## 3. Run directories and HEMCO_Config generation
- Global skeleton: generated from `lyin61/GC_ComplexSOA/UT.12.9.3` with `perl gcCopyRunDirs <input>`.
  - The input file format is strict: copy Lifei's `perl/CopyRunDirs.input` and edit the values. Comment lines in the runs list must start with `#`.
  - Result: `extension_2023_2025/rundirs/merra2_4x5_complexSOA`.
- `input.geos`: Lifei's nested file and the 4×5 template are identical apart from grid/time/paths (199 species).
- **HEMCO_Config.rc is always generated**, never hand-edited:
  `python extension_2023_2025/rundirs/make_hemco_config.py <global|nested> <MIX on/off> <DICE on/off> <out>`
  - It starts from Lifei's nested config and applies the changes below, then refuses to write if any `coda1`/lyin-home path remains.
  - **Paths:** ROOT/METDIR rewritten. The met-constants (CN) path is hard-coded in her config: the global run uses `$METDIR`, the nested run uses the global 0.5×0.625 CN file. Global runs drop `REG: NA` and the `.$REG` part of met file names.
  - **Year ranges:** every `1980-2020/` range (met fields, BCs; EFY flags would abort after 2020) is extended to 2025.
  - **Offline emissions:** OFFLINE_BIOVOC and SOILNOX use `v2021-12` (verified to match production 2019/2021 emissions to 0.999). OFFLINE_DUST uses `v2021-08`; the old `v2019-01` ended in 2019 and killed global dust after 2019. All offline ranges are extended to 1980-2025.
  - **Fires:** GFED4 points to `v2026-03` (2022–2025 converted from the GFED SFTP; 1997–2021 symlinked to v2020-02). The range is 1997/2003-2025; the flag is RF, i.e. zero outside the range.
  - **Lightning:** range 1980-2025 (flag RFY3, zero outside the range).
  - **Surface CH4:** NOAA_GMD `v2018-01` (1979-2020, RY → abort) is replaced by `v2023-10` 1975-2022 with flag CY, so 2023+ reuse 2022.
  - **Volcano table:** `v2024-04`.
  - **Boundary conditions:** path set to `extension_2023_2025/runs/global_4x5/GC_BCs`.
  - **NEI scale factors:** IDs 265–273 are replaced by `ExtData/HEMCO/NEI2016_YRSCALE/v2026-11/HEMCO_NEI16_scalefactors_2000-2025_optionB.rc`.
- HEMCO time-flag meaning matters: `C` = closest year; `R` = zero outside the range; `E` = must match exactly (else error or skip). After any change, audit the active entries capped before your end year that carry an R or E flag.
- HISTORY: segments use daily time-averaged collections (nested: the 11 Nested_C1 collections) and `Restart` 'End'.
  - Global runs also write BoundaryConditions every 3 h, daily files, `LON_RANGE -140 -40`, `LAT_RANGE 6 70` (199 species), the same as the old BC files.

## 4. Segment workflow
Scripts in `extension_2023_2025/rundirs/`:
- `setup_segment.py <global|nested> <name> <run_start> <keep_start> <end> <restart> [--retime] [--cores N] [--months-per-job M]` builds `runs/{global_4x5|nested}/<name>/` (configs, geos, `seg.conf`, `cap`, `Restarts/`).
  - `--retime` copies a borrowed restart and rewrites its time to `run_start`. Species are read with flag C, but met state (TMPU1, SPHU1, PS1DRY) uses EY and would otherwise be skipped.
- `run_chunks.sh` is the Slurm job. Submit it from the segment directory:
  `env $(env | grep -oE '^SLURM_[A-Z_0-9]+' | sed 's/^/-u /') sbatch -J <name> --cpus-per-task=24 run_chunks.sh`
  - Runs ~5-day chunks to month end (12.9.3 cannot write both interval and end-of-run restarts, so chunking gives both).
  - Moves restarts to `Restarts/`. Global runs move BC files to the shared GC_BCs directory.
  - Nested jobs wait (resubmit with `--begin=now+4hours`) until the BCs for the chunk exist.
  - Runs `qc_month.py` at month end. On PASS it prunes intra-month restarts older than a month and resubmits; on FAIL it writes `STOP`.
  - The log is `logs/driver.log`. To resume after fixing a problem: delete `STOP`, check that `cap` equals the last good restart date, and resubmit.
- Spin-up evidence: across 49 run joins in Nested_C1, jump frequency was at chance level even with 0-day spin-up when starting from same-date restarts.
  - Use 1 month of spin-up from real restarts and 2 months from borrowed (other-year) restarts.
- Plan used for the 2020–2025 rerun/extension:
  - Global runs per year in parallel from the old global restart at (Y-1)-12-01.
  - Nested runs in 3-month segments in parallel; 2023–25 segments borrow the same-month 2022 restart.
  - Cost ~123k core-hours (~$840).

## 5. Emission scaling
- NEI2016 monthly emissions are scaled to the year as follows:
  - **Non-EGU:** national annual factors (IDs 265–271).
  - **EGU SO2/NOx:** CAMD regional monthly factors (IDs 272/273), files `ExtData/HEMCO/CAMD_EGU/v2026-11/EGU_regional.monthly.1x1.YYYY.nc` (2000–2025).
- CAMD data:
  - Source: CAMPD bulk API `https://api.epa.gov/easey/bulk-files/emissions/daily/quarter/emissions-daily-YYYY-qN.csv`; the old gaftp path is gone.
  - Scripts: `extension_2023_2025/camd/` (`camd_regional_monthly.py`, `make_egu_sf_files.py`). These reproduce Lifei's 2016 values exactly.
- Non-EGU NOx/SO2 factors for 2018–2022 were inferred (`extension_2023_2025/infer_nonEGU_factors.py`):
  - Method: model HEMCO totals minus NEI16 ptegu × CAMD.
  - Validated on 2017: 0.957/0.898 versus the config's 0.958/0.903.
- 2023–2025 non-EGU factors are held at the 2022 values (user's "option B"), because EPA Trends 2024/2025 reuse 2023 and 2022→2023 mixes in NEI method changes.

## 6. Known issues in the existing record (2000–2022)
- **Nitrate collapse** (NO3 ~0, NH4 unbalanced; also in SpeciesConc):
  - Periods: 2013-01-01..02-15, 2018-08-30..11-09, 2019-01-01..01-20, 2022-02-23..03-12.
  - In the extracted files these days carry `qc_day_flag=3`.
- **Global BC dust is ~70× too low in 2020–2022** because OFFLINE_DUST v2019-01 ended in 2019. This lowers nested PM25 by ~0.8 µg/m³.
- **2022 offline biogenic and soil NOx equal 2021**; the 2022 files were missing when it ran.
- **Numerical blow-ups** near the Greenland edge (outside 10–65N, 140–50W).
- **2021-01-10 output file is empty**; December 2022 was not simulated.
