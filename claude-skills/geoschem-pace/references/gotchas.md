# Symptom → cause → fix (seen on PACE, 2026-11)

| Symptom | Cause | Fix |
|---|---|---|
| `An ORTE daemon has unexpectedly failed after launch` / `srun: fatal: SLURM_MEM_PER_CPU ... mutually exclusive` / `cpus-per-task set by two different environment variables` | `sbatch` was run from inside another Slurm job, and the parent's `SLURM_*` variables leaked into the new job | `env $(env \| grep -oE '^SLURM_[A-Z_0-9]+' \| sed 's/^/-u /') sbatch ...` |
| `Permission denied` writing into ExtData even though `[ -w ]` says writable | NFS 16-group limit (pliu40 has 41 groups) | `sg pace-is-pliu40-0 -c "umask 002; ..."` |
| `spack: command not found`, empty `NETCDF_HOME`, `ld: cannot find -lnetcdff` | `/storage/coda1/...` paths in old env scripts, the spack DB and `nf-config` | use `extension_2023_2025/code/gc1293_intel.env` (explicit paths plus `LIBRARY_PATH`) |
| Old `geos` binary: `libifcoremt.so.5 ... not found` | built on RHEL7 | rebuild with `build_gc1293.sh` |
| HEMCO: `Cannot find file ... CN.4x5` with a coda1 path | CN path hard-coded in Lifei's config | regenerate the config with `make_hemco_config.py` (fixed there) |
| `SET_CH4: Cannot evaluate NOAA_GMD_CH4` | NOAA GMD v2018-01 ends in 2020 with an `RY` flag | v2023-10 with `CY` (in the generator) |
| Missing fires/lightning after 2019–2020 with no error | `RF`/`RFY3` flags give zero outside the year range | extend the ranges (in the generator); audit R/E-flagged entries capped before the end year |
| Global BC dust ~70× low in 2020+ | OFFLINE_DUST v2019-01 ends in 2019 | use v2021-08 |
| NO3 ~0 and NH4 ≫ SO4+NO3 for weeks (ion ratio 3–6) | thermodynamic breakdown in some segments (2013/2018/2019/2022) | monthly QC catches it; rerun from the last good restart |
| Liquid water / pH output (`Chem_WaterSav`, `Chem_phSav`) looks like sea salt (pH 3–6, LWC ~0 over land) | upstream 12.9.3 registers both size categories under one name | `DO N = 1, 1` fix (in `Code_final_LY`); bzhang's GC12_SOA has corrected hourly LWC for 2000/05/10/15/20 |
| GCHP `Could not find field SPC_xxx in gchp_restart.nc4` | restart from an older version | regrid the same-version restart with GCPy (don't set `Require_Species_in_Restart=0` across versions) |
| GCHP SIGFPE in `ucx_mod.F90 calc_strat_aer` hours into a run | old-version restart missing `StatePSC` and other state fields | same as above |
| GCHP `double free or corruption` after `cap_restart` is written | finalize-time issue | harmless; judge success by `cap_restart` |
| `createRunDir.sh` asks for name/email | first-time GEOS-Chem registration | the user registers themselves; for testing, set `GC_USER_REGISTERED=true` in a temporary HOME |
| GC 12.9.3 `make build` log missing | the `build` target deletes `*.log` | read `compile.log` |
| `pkill -f <pattern>` killed your own shell (exit 144) | the pattern matched the running command line | kill by PID |
