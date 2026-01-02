### Installation of GEOS-Chem (version 14) on Georgia Tech Phoenix-Slurm cluster (using spack)

> Contributed By Bin Bai & Bingqing Zhang

> Last Updated: 2025/1/25

For more information, visit:[GEOS-Chem Quick Start Guide](https://geos-chem.readthedocs.io/en/latest/getting-started/quick-start.html)

---

### Notes for Pengfei Liu’s Group at Georgia Tech

- **Skip Steps 1–3. Start from Step 4.**
- Spack is already installed.
- Refer to the [Spack Guide](https://geos-chem.readthedocs.io/en/latest/geos-chem-shared-docs/supplemental-guides/spack-guide.html).

---


### **1. Download Spack to Your Home Directory**
```
git clone https://github.com/spack/spack.git
```
### **2. Load Spack**
```
export SPACK_ROOT=/storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack

source $SPACK_ROOT/share/spack/setup-env.sh
```
Alternatively:
```
source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack/share/spack/setup-env.sh
```
#### **2.1 Detect Existing Software**
```
spack external find
```
Spack will list software already available on the cluster and add them to `<Your Home Directory>/.spack/packages.yaml`. For example:

```bash
==> The following specs have been detected on this system and added to ~/.spack/packages.yaml
autoconf@2.69    cmake@3.26.5    findutils@4.8.0  git@2.43.5     libtool@2.4.6  perl@5.32.1    zlib@1.2.11
...
```
#### **2.2 Detect Existing Compilers**
```
spack compilers 
```
Spack should detect the available compilers, e.g.:
```
==> Available compilers
-- gcc rhel9-x86_64 ---------------------------------------------
gcc@11.4.1

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
    spec: intel@=2021.9.0
    paths:
      cc: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/intel-oneapi-compilers-classic-2021.9.0->
      cxx: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/intel-oneapi-compilers-classic-2021.9.0>
      f77: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/intel-oneapi-compilers-classic-2021.9.0>
      fc: /usr/local/pace-apps/spack/packages/linux-rhel9-x86_64_v3/gcc-11.3.1/intel-oneapi-compilers-classic-2021.9.0->    flags: {}
    operating_system: rhel9
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
  ```

Replace paths with the actual locations of the compiler binaries. Verify with:

```bash
spack compilers
```

### **3. Install Required Packages**

```
module load intel/2021.9.0

time spack install mpich%intel@2021.9.0

time spack install openmpi %intel@2021.9.0

time spack install netcdf-fortran %intel@2021.9.0^mpich

time spack install netcdf-c %intel@2021.9.0^mpich

time spack install flex %intel@2021.9.0

time spack install cmake %intel@2021.9.0

time spack install gmake %intel@2021.9.0

time spack install ncview %intel@2021.9.04^python@3.9

time spack install nco %intel@2021.9.04^mpich

spack load texinfo@6.5%intel@2021.9.0

time spack install esmf %intel@2021.9.04^mpich

# time spack install cgdb %intel@20.0.4.304 # This package is not required
```

### Troubleshooting

- **Package Not Found:** If you see this error when installing new packages,  `Error: Package 'cray-mpich' not found.`, add `^mpich` 

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
Use the `.GC` file provided by Dr. Liu's group:
```
source ~/.GC
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

