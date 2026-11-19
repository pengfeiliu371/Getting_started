# GCHP 14.8 on PACE (group-shared toolchain)

Full user tutorial: https://github.com/pengfeiliu371/Getting_started/blob/main/Run_GCHP_on_cluster.md

## Toolchain
- Location: `/storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/` (group-writable, setgid).
  - spack v1.2.2 with `SPACK_USER_CONFIG_PATH`/`SPACK_USER_CACHE_PATH` kept inside this directory, so it never reads or writes `~/.spack`.
  - Site config lives in `spack/etc/spack/{config,packages}.yaml`.
  - To manage packages: `source spack-setup.sh`; the environment definition is `envs/gchp-14.8/spack.yaml`.
- To build or run GCHP: `source .../spack-gchp/gchp.env`. It does the following:
  - `module purge`, then loads `gcc/12.3.0` and `openmpi/4.1.8`. This is the PACE OpenMPI with UCX/InfiniBand and Slurm; the default mvapich2 must not be used.
  - Puts the env view on PATH: ESMF 8.8.1, HDF5 1.14.6, NetCDF 4.10.0 / 4.6.2, CMake 3.31.
  - Sets `OMPI_MCA_pml=ucx`.
  - The "libosmcomp / openib" warning is harmless.
- Why not bzhang's spack OpenMPI 5.0.5: it was built without UCX/verbs/Slurm, so multi-node runs fall back to TCP.

## Build and run
1. Source code: `git clone --recurse-submodules --branch 14.8.0 https://github.com/geoschem/GCHP.git Code.14.8.0`
2. Run directory: `Code.14.8.0/run/createRunDir.sh`
   - Prompt order: sim type → option → met (1 = MERRA-2) → parent path → name → git (y/n).
   - It reads `GC_DATA_ROOT` from `~/.geoschem/config`. This must be the ExtData directory itself: `/storage/ideas/is-pliu40-0/ExtData`.
   - The first run triggers registration; see SKILL.md.
3. Compile in the run directory: `mkdir build && cd build && cmake ../CodeDir -DRUNDIR=.. -DCMAKE_BUILD_TYPE=Release && make -j 8 install` (~45 min).
4. Configure `setCommonRunSettings.sh`:
   - TOTAL_CORES = nodes × 24, and a multiple of 6.
   - CS_RES, Run_Duration, Diag_*.
   - STRETCH_GRID / STRETCH_FACTOR / TARGET_LAT / TARGET_LON.
   - The start date goes in `cap_restart`.
5. Job script: `spack-gchp/examples/gchp.pace.slurm`. It runs `mpirun -np $SLURM_NTASKS --bind-to core ./gchp` and renames the checkpoint to a dated restart. Submit with the SLURM_*-clean `sbatch` form.
6. Performance: `./extractPerformance.sh gchp.*.log`.

## Stretched grid (CONUS)
- Target-region resolution ≈ 10000 km / (N × S).
  - Tested: C24 with S=3 centered at 39N, -97 gives ~140–155 km over CONUS and ~700 km on the far side.
  - For research use C90–C180 with S≈2–3. MERRA-2 is ~50 km, so going much finer gains little.
- Stretched runs require winds (the MERRA-2 run directory already uses them) and a restart on exactly the same stretched grid.
- Make that restart with GCPy:
  `spack-gchp/examples/regrid_restart_to_stretched.sh <in.cN.nc4> N S LAT LON <workdir>`
  then copy `new_restart_file.nc` to `Restarts/GEOSChem.Restart.<date>_0000z.c<N>.nc4`.
- Never start 14.8 from an older-version restart with `Require_Species_in_Restart=0`. The 14.5→14.8 test crashed after 4 h with SIGFPE in `ucx_mod` because `StatePSC` and other fields were missing.

## GCPy environment
- Location: `spack-gchp/conda/gcpy` (GCPy 1.8.1, esmpy/xesmf/sparselt/gridspec from conda-forge).
  - It was installed with `--no-deps`, because pip GCPy lists a non-existent `esmf==8.8.1` dependency.
  - numpy is pinned below 2 because `sparselt` uses `np.product`.
- Use it via `export PATH=.../spack-gchp/conda/gcpy/bin:$PATH`.

## GCHP-specific behavior
- All inputs are read by MAPL ExtData from `ExtData.rc`. Year ranges in `HEMCO_Config.rc` are ignored.
- `Ext_AllowExtrap: .true.` uses the nearest available year. For example, GT_Chlorine, the group's own product covering 1960–2014, is fine.
- GC 14.8 defaults that differ from older versions:
  - Fires come from GFAS v2026-06 (GFED is off).
  - Offline dust is v2025-03 MERRA2.
  - Restarts come from `GEOSCHEM_RESTARTS/GC_14.8.0/`. Download missing resolutions from S3.
- Known harmless issue: a `double free or corruption` SIGABRT can occur after the final checkpoint and `cap_restart` are written. Success is judged from `cap_restart`.
