### Running GCHP (GEOS-Chem High Performance, v14.8) on the Georgia Tech Phoenix cluster

> Contributed by Pengfei Liu's group
> Last Updated: 2026/11/19

GCHP is the MPI (multi-node) version of GEOS-Chem. It runs on a cubed-sphere grid (C24 ≈ 4°, C48 ≈ 2°, C90 ≈ 1°, C180 ≈ 0.5°)
and supports **stretched grids**: one global simulation with refined resolution over a target region (e.g. CONUS), with no
boundary-condition run needed. This guide uses a **pre-built, group-shared toolchain**, so you do **not** need to install
spack, ESMF, NetCDF or MPI yourself.

Official docs: [GCHP documentation](https://gchp.readthedocs.io/en/latest/) ·
[Stretched-grid guide](https://gchp.readthedocs.io/en/latest/supplement/stretched-grid.html)

---

### Notes for Pengfei Liu's Group at Georgia Tech

- **Toolchain is already installed.** Load it with one line (Step 1).
- Shared input data (ExtData) is at `/storage/ideas/is-pliu40-0/ExtData`.
- This toolchain is **separate** from the GEOS-Chem Classic setup in [Install_GEOS_Chem_on_cluster.md](Install_GEOS_Chem_on_cluster.md).
  It does not touch `~/.spack` or the older shared spack, so both can be used side by side.

| Component | Version | Where it comes from |
|---|---|---|
| Compiler | GCC 12.3.0 | PACE module `gcc/12.3.0` |
| MPI | OpenMPI 4.1.8 (UCX/InfiniBand + Slurm) | PACE module `openmpi/4.1.8` |
| ESMF | 8.8.1 | group spack env `gchp-14.8` |
| HDF5 / NetCDF-C / NetCDF-Fortran | 1.14.6 / 4.10.0 / 4.6.2 | group spack env `gchp-14.8` |
| CMake | 3.31 | group spack env `gchp-14.8` |

> **Why not the old shared spack OpenMPI?** It was built without UCX/InfiniBand and Slurm support, so multi-node runs fall back
> to slow TCP. GCHP needs fast inter-node MPI, so this toolchain uses the PACE OpenMPI module instead.

---

### 1. Load the GCHP environment

```bash
source /storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/gchp.env
```
You should see:
```
GCHP env: gcc 12.3.0 | mpirun (Open MPI) 4.1.8 | ESMF: .../spack-gchp/envs/gchp-14.8/.spack-env/view/lib/esmf.mk
```
This file runs `module purge`, then loads `gcc/12.3.0` and `openmpi/4.1.8`. PACE loads `mvapich2` by default, and that
module **must not** be used with this toolchain. Source it in every new shell and at the top of every job script.

### 2. One-time setup: `~/.geoschem/config`

GEOS-Chem 14 / GCHP reads the data path from `~/.geoschem/config`. It must point to **the ExtData directory itself**:

```bash
mkdir -p ~/.geoschem
cat ~/.geoschem/config        # check what is there
```
The file should contain this line. If an old line points elsewhere (e.g. `.../shared/GEOS-Chem`), edit it:
```bash
export GC_DATA_ROOT=/storage/ideas/is-pliu40-0/ExtData
```
The first time you create a run directory, GCHP asks you to **register** (name, email, institution). Please answer with your own
information. This only happens once and is how the GEOS-Chem team tracks users for support.

### 3. Download the GCHP source code

```bash
mkdir -p /storage/project/r-pliu40-0/${USER}/GCHP && cd $_
git clone --recurse-submodules --branch 14.8.0 https://github.com/geoschem/GCHP.git Code.14.8.0
cd Code.14.8.0 && git log -1 --oneline        # should show "GCHP 14.8.0 release"
```
Use `--recurse-submodules`: GCHP pulls in GEOS-Chem, HEMCO, MAPL, FV3 and others (~560 MB).

### 4. Create a run directory

```bash
cd /storage/project/r-pliu40-0/${USER}/GCHP/Code.14.8.0/run
./createRunDir.sh
```
Answers used for the tested full-chemistry setup:

| Prompt | Answer |
|---|---|
| Simulation type | `1` Full chemistry |
| Additional option | `1` Standard (or `3` Complex SOA, etc.) |
| Meteorology | `1` MERRA-2 |
| Path where the run directory will be created | e.g. `/storage/project/r-pliu40-0/${USER}/GCHP/rundirs` |
| Run directory name | e.g. `fullchem_merra2_c24` |
| Version control (git) | `n` (or `y` if you want it) |

Check that the data links resolve:
```bash
cd /storage/project/r-pliu40-0/${USER}/GCHP/rundirs/fullchem_merra2_c24
ls -l MetDir HcoDir ChemDir CodeDir      # should point into /storage/ideas/is-pliu40-0/ExtData
ls -lL Restarts/                         # restart files must exist (see Troubleshooting if "No such file")
```

### 5. Compile GCHP (once per run directory, ~45 min)

```bash
cd /storage/project/r-pliu40-0/${USER}/GCHP/rundirs/fullchem_merra2_c24
source /storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/gchp.env
mkdir -p build && cd build
cmake ../CodeDir -DRUNDIR=.. -DCMAKE_BUILD_TYPE=Release
make -j 8 install
ls -l ../gchp          # the executable
```
CMake should report `Found ESMF ... version "8.8.1"` and `Found MPI_Fortran: .../openmpi-4.1.8-.../libmpi_usempif08.so`.
Compile on a compute node (an interactive or batch job), not on the login node.

### 6. Configure the simulation

Most settings are in **`setCommonRunSettings.sh`**. GCHP applies them to the `.rc` files at run time. Key entries:

```bash
TOTAL_CORES=48           # must be a multiple of 6
NUM_NODES=2
NUM_CORES_PER_NODE=24    # Phoenix CPU nodes have 24 cores
CS_RES=24                # cubed-sphere resolution: 24, 48, 90, 180 ...
Run_Duration="00000001 000000"   # YYYYMMDD hhmmss -> 1 day
Diag_Monthly="0"         # 1 = monthly-mean output
Diag_Frequency="240000"  # output frequency (hhmmss) when Diag_Monthly=0
Diag_Duration="240000"
STRETCH_GRID=OFF         # see Section 9
```
- Start date: edit `cap_restart` (e.g. `20190701 000000`).
- Diagnostics: `HISTORY.rc`. Emissions: `HEMCO_Config.rc`. Chemistry options: `geoschem_config.yml`.
- Core limits: at least 4×4 grid cells per core. Max cores: C24 216, C48 864, C90 2904, C180 12150.

### 7. Submit a job

Copy the PACE job script from the shared folder and edit `-N`, `--ntasks-per-node`, `-t`, and `-J`:
```bash
cp /storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/examples/gchp.pace.slurm .
sbatch gchp.pace.slurm
```
The script loads `gchp.env` and applies `setCommonRunSettings.sh`. It then runs
`mpirun -np $SLURM_NTASKS --bind-to core ./gchp`, and when the run ends it renames the checkpoint to a dated restart, so the
next job continues from where this one stopped. `#SBATCH -N` × `--ntasks-per-node` must equal `TOTAL_CORES`.

**Tested (2026-11-19):** full chemistry at C24, MERRA-2, 2019-07-01 for 1 day, on 2 nodes × 24 cores. It ran at about
**211 simulated days per wall-clock day**, i.e. ~7 min per simulated day or ~1.7 days per simulated year. Throughput at higher
resolution scales roughly with the number of grid cells (C48 has 4× the cells of C24), so use more cores for C48/C90.

Monitor:
```bash
squeue -u $USER
tail -f gchp.YYYYMMDD_0000z.log      # main GCHP log
ls OutputDir/                         # GEOSChem.*.nc4 diagnostics (cubed-sphere)
./extractPerformance.sh gchp.*.log    # simulated days per wall-clock day
```

### 8. Output on the cubed sphere

GCHP writes diagnostics on the cubed-sphere grid (`nf=6, Ydim, Xdim`), with `lats`/`lons` variables. You have two options:
- Regrid afterwards, e.g. with [GCPy](https://gcpy.readthedocs.io/) or `xesmf`.
- Have GCHP write a lat-lon grid directly. See the instructions at the top of `HISTORY.rc` (`GRID_LABELS`).

### 9. Stretched grid (regional refinement, e.g. over CONUS)

In `setCommonRunSettings.sh`:
```bash
STRETCH_GRID=ON
STRETCH_FACTOR=3.0       # >1; refined-region resolution ≈ 10000 km / (CS_RES × STRETCH_FACTOR)
TARGET_LAT=39.0          # center of the refined face (CONUS)
TARGET_LON=-97.0
```
- Stretched-grid runs **need winds, not mass fluxes**. The MERRA-2 run directory already uses winds.
- Approximate resolution over the target region: C48 with S=3 is about 70 km; C90 with S=3 about 37 km; C180 with S=2 about 28 km.
  MERRA-2 is 0.5°×0.625° (~50 km), so going much finer than that adds little for MERRA-2 runs.

#### 9.1 Make a stretched-grid restart with GCPy (required)

The restart file must be on **exactly** the same stretched grid: same `CS_RES`, `STRETCH_FACTOR`, `TARGET_LAT` and `TARGET_LON`.
Regrid the standard GEOS-Chem 14.8 restart with the group script. It uses the shared GCPy environment, which includes
`gridspec`, `ESMF_RegridWeightGen` and GCPy 1.8.1:

```bash
/storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/examples/regrid_restart_to_stretched.sh \
    /storage/ideas/is-pliu40-0/ExtData/GEOSCHEM_RESTARTS/GC_14.8.0/GEOSChem.Restart.fullchem.20190701_0000z.c24.nc4 \
    24 3.0 39.0 -97.0  ./regrid_work            # N  S  TARGET_LAT  TARGET_LON  workdir

# put it where GCHP looks for it (the name has no stretch info; GCHP checks the file attributes)
rm -f Restarts/GEOSChem.Restart.20190701_0000z.c24.nc4
cp ./regrid_work/new_restart_file.nc Restarts/GEOSChem.Restart.20190701_0000z.c24.nc4
```
C24 takes under a minute. For C90 or C180, run the script inside a job, because the weights file is large.

**Tested (2026-11-19):** C24, S=3.0, target 39°N/97°W, full chemistry, 1 day on 2 nodes × 24 cores, about 212 simulated
days/day. Grid spacing in the output is ~140–155 km over CONUS (Kansas, Atlanta, Los Angeles) versus ~700 km on the
opposite side of the globe; uniform C24 is ~400 km everywhere. For research runs use C90–C180 with S≈2–3.

> Do **not** start a 14.8 run from an older-version restart (e.g. `GC_14.5.0/...c24_s2.0...`) with
> `Require_Species_in_Restart=0`. In our test it ran for 4 h, then crashed with `SIGFPE` in `ucx_mod.F90`, because new state
> fields such as `StatePSC` were missing. Always regrid a restart from the same GEOS-Chem version.

To use GCPy for other work, such as regridding cubed-sphere output to lat-lon or plotting:
```bash
export PATH=/storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/conda/gcpy/bin:$PATH
python -c "import gcpy, xesmf"
```

### Troubleshooting

- **`Restarts/... No such file or directory`.** The run directory links to `ExtData/GEOSCHEM_RESTARTS/GC_14.8.0/`, which may
  be missing for your resolution. Download it into the shared ExtData; see "Getting missing input data" below.
- **Missing emission/met file errors in the log.** GEOS-Chem 14.8 defaults differ from older versions. For example, fires use
  **GFAS v2026-06** and dust uses **OFFLINE_DUST v2025-03 (MERRA2)**. Download the data for your simulation period.
- **`double free or corruption` / `SIGABRT` at the very end of the run (known, harmless so far).** On this toolchain GCHP can
  abort during final cleanup *after* it has written the checkpoint and advanced `cap_restart`. The job script decides success
  from `cap_restart`, not the exit code. If `cap_restart` shows the new date and `Restarts/GEOSChem.Restart.<new date>z.c<N>.nc4`
  exists, the run is complete and the next job continues correctly.
- **`mca_btl_openib ... libosmcomp.so` warnings.** Harmless. `gchp.env` sets `OMPI_MCA_pml=ucx` so InfiniBand goes through UCX.
- **`Could not find field SPC_xxx in gchp_restart.nc4`.** The restart is from an older GEOS-Chem version and lacks species
  added later (e.g. a 14.5 restart in 14.8). Set `Require_Species_in_Restart=0` in `setCommonRunSettings.sh` to start the
  missing species from default background values, and spin up long enough before the period you analyze.
- **`An ORTE daemon has unexpectedly failed after launch` / `srun: fatal: SLURM_MEM_PER_CPU ... mutually exclusive`.** This
  happens when you run `sbatch` **from inside another Slurm job**, such as an interactive session (`salloc`, OnDemand,
  VS Code on a compute node). The parent job's `SLURM_*` variables leak into the new job and break `srun`, which `mpirun`
  uses to start processes on the other nodes. Submit from a login node, or clear the variables when you submit:
  ```bash
  env $(env | grep -oE '^SLURM_[A-Z_0-9]+' | sed 's/^/-u /') sbatch gchp.pace.slurm
  ```
- **Job dies right away with MPI errors.** Make sure `mvapich2` is not loaded (`module list`), and that `TOTAL_CORES` equals
  `-N × --ntasks-per-node`.
- **Input data only exists for some years (e.g. `GT_Chlorine`, 1960–2014).** This is fine. `ExtData.rc` sets
  `Ext_AllowExtrap: .true.`, so when a year's file is missing, GCHP searches year by year and uses the closest file it finds.
  This matches the year-range / `C` flag behavior of HEMCO in GEOS-Chem Classic. In GCHP the year ranges written in
  `HEMCO_Config.rc` are **ignored**: all files are read by MAPL ExtData according to `ExtData.rc`.

### Getting missing input data

All group input data lives in `/storage/ideas/is-pliu40-0/ExtData`. Missing files can be downloaded from the public GEOS-Chem
archive on AWS (`https://gcgrid.s3.amazonaws.com/<same relative path>`). For example:
```bash
cd /storage/ideas/is-pliu40-0/ExtData
sg pace-is-pliu40-0 -c "umask 002; mkdir -p HEMCO/GFAS/v2026-06/2019/07 && \
  curl -sf -o HEMCO/GFAS/v2026-06/2019/07/GFAS-smoke-20190701.nc \
  https://gcgrid.s3.amazonaws.com/HEMCO/GFAS/v2026-06/2019/07/GFAS-smoke-20190701.nc"
```
- If you belong to many Unix groups, writes to ExtData need the `sg pace-is-pliu40-0 -c "..."` prefix. The NFS server only
  sees your first 16 groups.
- Bulk download helpers are in `/storage/project/r-pliu40-0/pliu40/GEOS-Chem/extension_2023_2025/download/`
  (`build_manifest.py`, `run_download.sh`).

---

### For maintainers: how the shared toolchain was built

```
/storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/
├── spack/                 # spack v1.2.2 (site config in spack/etc/spack/{config,packages}.yaml)
├── envs/gchp-14.8/        # spack.yaml for the GCHP environment (+ .spack-env/view used by gchp.env)
├── opt/                   # installed packages
├── user_config/, user_cache/   # isolated spack user scope (keeps ~/.spack untouched)
├── spack-setup.sh         # source to use this spack instance
├── gchp.env               # source to build/run GCHP
└── logs/                  # build logs
```
```bash
source /storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/spack-setup.sh
cd /storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp/envs/gchp-14.8
spack -e . concretize -f && spack -e . install -j 8
```
- `packages.yaml` registers PACE `gcc@12.3.0` and `openmpi@4.1.8 fabrics=ucx schedulers=slurm` as externals (non-buildable).
- Everything is group-writable (setgid, `umask 002`). Use `umask 002` when you add packages.
