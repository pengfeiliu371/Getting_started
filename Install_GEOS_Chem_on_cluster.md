### Installation of GEOS-Chem (version 14) on Georgia Tech Phoenix-Slurm cluster (using spack)

> Contributed Bingqing Zhang & By Bin Bai 
> Last Updated: 2026/08/08

For more information, visit:[GEOS-Chem Quick Start Guide](https://geos-chem.readthedocs.io/en/latest/getting-started/quick-start.html)

---

### Notes for Pengfei Liu’s Group at Georgia Tech

- **Skip Steps 1–3. Start from Step 4.**
- Spack is already installed.
- Refer to the [Spack Guide](https://geos-chem.readthedocs.io/en/latest/geos-chem-shared-docs/supplemental-guides/spack-guide.html).

---
### 0. Default module in GT-Phoenix cluster
```
----------------------------------------------------
Currently Loaded Modules:
  1) xalt/3.2.3       3) mpfr/4.2.0 (H)   5) zlib/1.2.13 (H)   7) libpciaccess/0.17 (H)   9) slurm/23.11.1    (H)
  2) gmp/6.2.1  (H)   4) mpc/1.3.1  (H)   6) gcc/12.3.0        8) pmix/4.2.6        (H)  10) mvapich2/2.3.7-1

  Where:
   H:  Hidden Module
----------------------------------------------------
```

### **1. Download Spack to Your Home Directory**
```
git clone https://github.com/spack/spack.git
```
### **2. Load Spack**
```
module purge

module load gcc/12.3.0

export SPACK_ROOT=/storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack

source $SPACK_ROOT/share/spack/setup-env.sh
```
Alternatively:
```
source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack/share/spack/setup-env.sh
```

#### **2.1 Put Spack’s build stage on an exec-enabled project path (not /tmp)**
```
mkdir -p /storage/home/hcoda1/0/${USER}/spack-stage
spack config add "config:build_stage:[/storage/home/hcoda1/0/${USER}/spack-stage]"
spack config get config # verify the build_stage shows your project path
```
#### 2.2 Start with a clean slate and clean env
```
spack clean -a
unset CC CXX FC F77 F90 CFLAGS CXXFLAGS FFLAGS LDFLAGS LIBS CPPFLAGS
```

#### **2.3 Detect Existing Compilers**
Here I choose to compile GEOS-Chem with gcc. First load the compiler:
```
module purge
module load gcc/12.3.0
spack compilers 
```
Spack should detect the available compilers, e.g.:
```
==> Available compilers
-- gcc rhel9-x86_64 ---------------------------------------------
gcc@12.3.0  gcc@11.5.0  gcc@11.4.1

-- intel rhel9-x86_64 -------------------------------------------
intel@2023.1.0  intel@2021.9.0

-- oneapi rhel9-x86_64 ------------------------------------------
oneapi@2023.1.0
```
If your desired compiler is not listed, add it:

- **Automatically:**
  ```bash
  spack compiler find
  ```
- **Manually:** Edit `<home directory>/.spack/linux/compilers.yaml` to include:

  ```yaml
  - compiler:
    spec: gcc@=12.3.0
    paths:
      cc: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/gcc-12.3.0-ukkkutsxfl5kpnnaxflpkq2jtliwt>
      cxx: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/gcc-12.3.0-ukkkutsxfl5kpnnaxflpkq2jtliw>
      f77: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/gcc-12.3.0-ukkkutsxfl5kpnnaxflpkq2jtliw>
      fc: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/gcc-12.3.0-ukkkutsxfl5kpnnaxflpkq2jtliwt>
    flags: {}
    operating_system: rhel9
    target: x86_64
    modules: [gcc/12.3.0] #<== add this
    environment: {}
    extra_rpaths: []
  ```

Replace paths with the actual locations of the compiler binaries. Verify with:

```bash
spack compilers
```
### **3. Detect externals**
```
spack external find
```
Spack will list software already available on the cluster and add them to `<Your Home Directory>/.spack/packages.yaml`. For example:
```bash
==> The following specs have been detected on this system and added to /storage/home/hcoda1/0/{USER}/.spack/packages.yaml
git@2.47.3  git-lfs@3.6.1  openssl@3.2.2  zlib@1.2.13
```

### **4. Install Required Packages (consistent with openmpi)**
```

spack env create geoschem-gcc
spack env activate geoschem-gcc
spack config add "packages:all:target:[x86_64_v3]"
# add OpenMPI first
spack add openmpi@5.0.5 %gcc@12.3.0
spack concretize --fresh
spack install

# after OpenMPI is done, add NetCDF stack
spack add hdf5+fortran+hl+mpi %gcc@12.3.0 ^openmpi@5.0.5
spack add netcdf-c+mpi %gcc@12.3.0 ^openmpi@5.0.5
spack add netcdf-fortran %gcc@12.3.0 ^openmpi@5.0.5

spack concretize --fresh
spack install

# (GCHP only, but I also install this)
spack add parallelio %gcc@12.3.0 ^openmpi@5.0.5
spack add esmf+mpi %gcc@12.3.0 ^openmpi@5.0.5
spack concretize 
spack install --jobs 4

If there is an ESMF memory issue, 
spack clean -s esmf
spack install --jobs 2 esmf


# Utilities
spack add curl %gcc@12.3.0
spack add nco %gcc@12.3.0
spack add cdo %gcc@12.3.0 ^proj~curl

spack concretize 
spack install
```

### Troubleshooting

- **Cache Conflicts:** If you encounter this error:```==> Error: Name``` , the reason might be two versions of spack caches conflicting with each other, you can solve this by renaming (or deleting) the original cache file in ```{Home directory}/.spack/cache```

- **Build Errors:** Uninstall the failing package and try a different version. Example:
```
>> 25    configure: error: perl >= 5.8.1 with Encode, Data::Dumper and Unicode::Normalize required by Texinfo.
```


\####################################################################

\# if you are in Pengfei Liu’s group at Gatech 

\# just start from here (spack is already installed)!!###

\####################################################################

### **4. Download GEOS-Chem Source Code** 

For version 14.0.0 or later, create a new directory (e.g. GC14) and download the source code here
```
mkdir GC14
cd GC14
```
- **Using GitHub:**
  - Fork the `GCClassic` repository to your GitHub account.
  - Clone your fork:
    ```bash
    git clone --recurse-submodules https://github.com/<your_github_account>/GCClassic.git Code.X.Y.Z
    ```
- **Official Repository:**
  ```bash
  git clone --recurse-submodules https://github.com/geoschem/GCClassic.git Code.X.Y.Z
  ```

Replace `Code.X.Y.Z` with the version, e.g., `Code.14.3.0`. Confirm the version:

```bash
git log -n 1
```
### **5. (Optional) Link Local and Central Repositories**
Only if you want to track your code change with Github
```
git remote add upstream <Central repo URL>
```
##e.g., for me, it is: `git remote add upstream https://github.com/bzhang419/GCClassic.git`

Then if you type `git remote -v`, the origin should point to your remote repo and the upstream should point to the central repo like this:
```
origin  https://github.com/bzhang419/GCClassic.git (fetch)
origin  https://github.com/bzhang419/GCClassic.git (push)
upstream        https://github.com/bzhang419/GCClassic.git (fetch)
upstream        https://github.com/bzhang419/GCClassic.git (push)
```
### **6. (Optional) Commit and Push Changes**
1) Make some changes.
2) Stage and commit:
   ```
   git add .
   git commit -m "info of your change"
   git push origin main
   ```
   Then you will be asked to enter your username and password (Note that from 2021-08-13, GitHub no longer accepts account passwords when authenticating Git operations. Instead of entering your GitHub password, you need to enter your PAT (Personal Access Token). You can generate one by navigating your account -> settings -> developer settings -> personal access tokens -> tokens (classic) -> generate new token. Enter this instead of your GitHub password).
   
   Every time you perform a git action, you will need to enter the user name and token. If you want to avoid this, you can store your credentials in cache by entering `git config --global --replace-all credential.helper cache`
   
### **7. Load Environment**
Use the `GCenvgcc` file provided by Dr. Liu's group:
```
source ~/GCenvgcc
```
A sample GC environment config file is as follows:

```
#set -euo pipefail

echo "=== Reset the environment ==="

# Helper: unset all variables that start with a given prefix
unset_prefix() {
  local prefix="$1"
  local name
  while IFS= read -r name; do
    unset "$name"
  done < <(compgen -v | awk -v p="$prefix" 'index($0,p)==1')
}

# Unset specific known vars if set
for v in PERL_HOME CC CXX FC F77 F90 F90FLAGS FFLAGS CFLAGS CXXFLAGS LDFLAGS \
         ESMFMKFILE OMP_NUM_THREADS OMP_STACKSIZE; do
  unset "$v" 2>/dev/null || true
done

unset_prefix NETCDF_
unset_prefix GC_
unset_prefix HDF5_

module purge
module load gcc/12.3.0

echo "=== Set up Spack ==="
export SPACK_ROOT=/storage/project/r-pliu40-0/shared/GEOS-Chem/spack
source $SPACK_ROOT/share/spack/setup-env.sh

echo "=== Activate Spack environment ==="
spack env activate geoschem-gcc

echo "=== Load exact GEOS-Chem/GCHP toolchain by hash ==="

# Hashes from current geoschem-gcc environment
OMPI_HASH=tmnxxce      # openmpi@5.0.5%gcc@12.3.0
HDF5_HASH=gafqxk4      # hdf5@1.14.5%gcc@12.3.0 +fortran+hl+mpi
NCC_HASH=y2lhyty       # netcdf-c@4.9.2%gcc@12.3.0 +mpi
NCF_HASH=arsv2g4       # netcdf-fortran@4.6.1%gcc@12.3.0
ESMF_HASH=duszk3p      # esmf@8.8.0%gcc@12.3.0 +mpi
PIO_HASH=ozlp56j       # parallelio@2.6.3%gcc@12.3.0
NCO_HASH=i5uz2jq       # nco@5.2.4%gcc@12.3.0
CDO_HASH=jshwm5b       # cdo@2.4.4%gcc@12.3.0

# Core MPI / I/O stack
spack load "/$OMPI_HASH"
spack load "/$HDF5_HASH"
spack load "/$NCC_HASH"
spack load "/$NCF_HASH"

# GCHP dependencies
spack load "/$PIO_HASH"
spack load "/$ESMF_HASH"

# Utilities
spack load "/$NCO_HASH"
spack load "/$CDO_HASH"

echo "=== Compilers: use MPI wrappers ==="
export CC=mpicc
export CXX=mpicxx
export FC=mpifort

echo "=== Discover NetCDF prefixes from config tools ==="
export NETCDF_C_ROOT="$(nc-config --prefix)"
export NETCDF_FORTRAN_ROOT="$(nf-config --prefix)"

# Traditional GEOS-Chem variables
export NETCDF_HOME="$NETCDF_C_ROOT"
export NETCDF_BIN="$NETCDF_C_ROOT/bin"
export NETCDF_INCLUDE="$NETCDF_C_ROOT/include"
export NETCDF_LIB="$NETCDF_C_ROOT/lib"
export GC_BIN="$NETCDF_BIN"
export GC_INCLUDE="$NETCDF_INCLUDE"
export GC_LIB="$NETCDF_LIB"

export NETCDF_FORTRAN_HOME="$NETCDF_FORTRAN_ROOT"
export NETCDF_FORTRAN_BIN="$NETCDF_FORTRAN_ROOT/bin"
export NETCDF_FORTRAN_INCLUDE="$NETCDF_FORTRAN_ROOT/include"
export NETCDF_FORTRAN_LIB="$NETCDF_FORTRAN_ROOT/lib"
export GC_F_BIN="$NETCDF_FORTRAN_BIN"
export GC_F_INCLUDE="$NETCDF_FORTRAN_INCLUDE"
export GC_F_LIB="$NETCDF_FORTRAN_LIB"

echo "=== HDF5 and ESMF prefixes ==="
export HDF5_ROOT="$(spack location -i "/$HDF5_HASH")"
export HDF5_DIR="$HDF5_ROOT"

export ESMF_ROOT="$(spack location -i "/$ESMF_HASH")"
export ESMFMKFILE="${ESMF_ROOT}/lib/esmf.mk"

export PIO_ROOT="$(spack location -i "/$PIO_HASH")"

echo "=== Runtime and discovery paths ==="
export LD_LIBRARY_PATH="${NETCDF_FORTRAN_LIB}:${NETCDF_LIB}:${HDF5_ROOT}/lib:${PIO_ROOT}/lib:${ESMF_ROOT}/lib:${LD_LIBRARY_PATH:-}"

export CMAKE_PREFIX_PATH="${NETCDF_C_ROOT}:${NETCDF_FORTRAN_ROOT}:${HDF5_ROOT}:${PIO_ROOT}:${ESMF_ROOT}:${CMAKE_PREFIX_PATH:-}"
export PKG_CONFIG_PATH="${NETCDF_C_ROOT}/lib/pkgconfig:${HDF5_ROOT}/lib/pkgconfig:${PIO_ROOT}/lib/pkgconfig:${PKG_CONFIG_PATH:-}"

# Add runtime search paths parsed from *-config
for p in $(nf-config --flibs | tr ' ' '\n' | grep '^-L' | sed 's/^-L//'); do
  [[ ":$LD_LIBRARY_PATH:" == *":$p:"* ]] || export LD_LIBRARY_PATH="$p:${LD_LIBRARY_PATH:-}"
done

for p in $(nc-config --libs | tr ' ' '\n' | grep '^-L' | sed 's/^-L//'); do
  [[ ":$LD_LIBRARY_PATH:" == *":$p:"* ]] || export LD_LIBRARY_PATH="$p:${LD_LIBRARY_PATH:-}"
done

echo "=== ulimit settings ==="
ulimit -c unlimited
ulimit -s unlimited

echo "=== Sanity checks ==="
which mpicc
which mpifort
which nc-config
which nf-config
which ncks
which cdo
echo "NetCDF-C  : $(nc-config --version) @ ${NETCDF_C_ROOT}"
echo "NetCDF-For: $(nf-config --version) @ ${NETCDF_FORTRAN_ROOT}"
echo "ESMFMKFILE: ${ESMFMKFILE}"

echo "GC/GCHP toolchain ready: GCC 12.3.0 + OpenMPI 5.0.5 + HDF5 + NetCDF + ESMF + PIO."

```
### **8. Create a Run Directory**
```
cd run/
./createRunDir.sh
```
This step will create a run directory that contains files for your simulation.
### **9. Configure, Compile, and Install**
```
# Navigate to the run directory you just created
cd /path/to/gc_4x5_merra2_fullchem  # Skip if you are already here
cd build
# configure your build
cmake ../CodeDir -DRUNDIR=..
# compile and install
make -j
make install
```
If successful, the executable `gcclassic` will appear in the run directory.
### **10. Configure Simulation Settings**
Edit the following files in your run directory:

- `geoschem_config.yml`
- `HISTORY.rc`
- `HEMCO_Diagn.rc`
- `HEMCO_Config.rc`
- `HEMCO_Config.rc.gmao_metfields`
Refer to the [[GEOS-Chem Wiki]([https://geos-chem.readthedocs.io](https://geos-chem.readthedocs.io/en/stable/getting-started/quick-start.html#configure-your-run-directory))] for configuration details.
### **11. Submit a Job**
Use the sample Slurm script provided in Dr. Liu’s shared folder:
```bash
/storage/coda1/p-pliu40/0/shared/GEOS-Chem/slurm_rh9/run_GC.sbatch
```

Copy it to your run directory, make necessary edits, and submit:

```bash
sbatch run_GC.sbatch
```

