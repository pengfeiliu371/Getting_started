---
name: geoschem-pace
description: Compile, configure and run GEOS-Chem on the Georgia Tech PACE Phoenix cluster for Pengfei Liu's group. Covers GEOS-Chem Classic 12.9.3 (Lifei Yin's complex-SOA Nested_C1 setup: global 4x5 boundary-condition runs + 0.5x0.625 North America nested runs) and GCHP 14.8 (multi-node, stretched grid over CONUS), plus the shared input data (ExtData), restarts, Slurm job chaining, month-end QC and the many cluster-specific pitfalls. Use this skill whenever the user wants to build or rebuild GEOS-Chem/GCHP, create or edit a run directory, HEMCO_Config.rc, HISTORY.rc or input.geos, extend or rerun the Nested_C1 simulation, submit or debug GEOS-Chem Slurm jobs, download or fill ExtData inputs (MERRA-2, GFED, CAMD/NEI scaling, offline emissions), or plan cost/time for model runs on PACE — even if they just say "跑一下模式", "续跑", "rundir", "GCHP", or paste a GEOS-Chem error log.
---

# GEOS-Chem on PACE Phoenix (Liu group)

Two supported setups. Pick by the task:

| Task | Setup | Read next |
|---|---|---|
| Continue / rerun / modify Lifei's Nested_C1 complex-SOA record (2000–2025), anything that must stay consistent with it | **GC-Classic 12.9.3**, Intel, single node OpenMP | `references/classic-1293.md` |
| New multi-year studies, stretched grid over CONUS, multi-node runs | **GCHP 14.8**, GCC 12.3 + OpenMPI 4.1.8, group spack | `references/gchp.md` |
| Finding / downloading / filling input data, write permissions on ExtData | either | `references/data-and-storage.md` |
| A job failed or numbers look wrong | either | `references/gotchas.md` first — most failures seen so far are listed there with the fix |

## Facts that apply everywhere

- **Shared input data (ExtData):** `/storage/ideas/is-pliu40-0/ExtData` (= `/storage/project/r-pliu40-0/shared/GEOS-Chem/data/ExtData`). The old prefix `/storage/coda1/p-pliu40/0/...` no longer exists, but it is still baked into old scripts, spack databases, `nf-config` output and Lifei's configs. Treat any `coda1` path as stale and rewrite it.
- **Writing into ExtData:** pliu40 belongs to >16 Unix groups and NFS only sees the first 16, so writes into group-only directories fail with "Permission denied" even though `[ -w dir ]` says yes. Use `sg pace-is-pliu40-0 -c "umask 002; <cmd>"`, never `chmod 777`. Test permissions by actually writing.
- **Slurm:** account `gts-pliu40`, QOS `inferno` (≤6000 cores per user, ≤21-day jobs). CPU nodes have 24 cores (cascadelake). Intel-built libraries target cascadelake, so add `--constraint=cascadelake` to GC-Classic jobs.
- **Submitting from inside a job** (interactive session, a running job, or this agent's own session) leaks `SLURM_*` variables. That breaks `srun`/`mpirun` ("ORTE daemon failed", "SLURM_MEM_PER_CPU ... mutually exclusive"). Always submit with
  `env $(env | grep -oE '^SLURM_[A-Z_0-9]+' | sed 's/^/-u /') sbatch ...`
- **Cost:** PACE internal rate was $0.0068/CPU-hour (2022 posted rate). Measured speeds:
  - Nested 0.5°×0.625° NA complex SOA on 16 cores: ~2.2 wall-hours per simulated day, i.e. ~33 days and ~12.7k core-hours per simulated year.
  - Global 4°×5° on 16 cores: ~13 min per simulated day, ~3.5 days per simulated year.
  - GCHP C24 on 48 cores: ~210 simulated days per day.
- **Don't touch:** bzhang419's spack (`shared/GEOS-Chem/spack`), anyone's `~/.spack`, `~/.bashrc`, or `~/.geoschem/config` without asking. Read-only sources: `lyin61/...`, `bzhang419/...`.
- **GEOS-Chem first-time registration** (`createRunDir.sh`) POSTs name/email to gc-dashboard.org. Never fill it with the user's information. For your own testing, put `export GC_USER_REGISTERED=true` in a temporary `HOME=.../.geoschem/config`.

## Working style that has paid off

1. **Validate before production.** Rebuild, then reproduce a month of an existing output: compare emissions with HEMCO diagnostics and compare concentrations. Only then launch long runs.
2. **Short jobs, frequent restarts.** For long nested runs, run one month per Slurm job in ~5-day chunks, keep both GEOS-Chem and HEMCO restarts after each chunk, and run QC at month end before submitting the next month. A failure then costs ≤5 days instead of a whole segment.
3. **QC every month** for numerical blow-ups (PM25 > 1e5, non-finite values), nitrate collapse (median CONUS NH4/(2SO4+NO3) equivalent ratio > 1.3; the normal value is ~0.97) and O3 range. The Nested_C1 record already has collapse episodes in 2013, 2018, 2019 and 2022.
4. **Write down every data substitution** (filled years, version swaps, held-constant factors) in a README next to the data and in the extension notes, so later users know what is real and what is filled.

Project notes with full history:
- `/storage/project/r-pliu40-0/pliu40/GEOS-Chem/extension_2023_2025/` (INPUT_DATA_STATUS.md, NEI_SCALING_METHOD.md, CODE_VARIANTS.md)
- Group tutorial: https://github.com/pengfeiliu371/Getting_started (Run_GCHP_on_cluster.md)
