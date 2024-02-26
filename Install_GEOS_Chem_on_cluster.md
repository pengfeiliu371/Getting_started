### Installation of GEOS-Chem (version 14) on Georgia Tech Phoenix-Slurm cluster (using spack)

> Contributed By Bin Bai & Bingqing Zhang

> Last Updated: 2024/2/25

More Info at https://geos-chem.readthedocs.io/en/stable/getting-started/quick-start.html

\####################################################################

\# if you are in Pengfei Liu’s group at Gatech 

\# skip step 1-3 and start from step 4 

\# spack is already installed ###

\####################################################################

#### 1. Download SPACK in home directory
```
git clone https://github.com/spack/spack.git
```
#### 2. Load SPACK
```
export SPACK_ROOT=/storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack

source $SPACK_ROOT/share/spack/setup-env.sh
```
\# or just
```
source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack/share/spack/setup-env.sh
```
##### 2.1 Make sure your compiler is detected by SPACK
In this cluster we use intel20.0.4, thus type:
```
spack compilers 
```
You should be able to see the compiler (intel 20.0.4) in the list, the result should be similar to the following: 
```
==> Available compilers
-- gcc rhel7-x86_64 ---------------------------------------------
gcc@10.3.0  gcc@10.1.0  gcc@8.3.0  gcc@4.8.5  gcc@4.4.7

-- intel rhel7-x86_64 -------------------------------------------
intel@20.0.4.304  intel@19.1.3.304  intel@19.0.5.281
```
If you cannot see the compiler, you can add it to the spack compiler list 
  - automatically, by typing:
    spack compiler find
  - manually, by adding the following lines to this file \<home directory\>/.spack/linux/compilers.yaml
  ```
  - compiler:
    spec: intel@20.0.4.304
    paths:
      cc: /usr/local/pace-apps/spack/packages/linux-rhel7-x86_64/gcc-4.8.5/intel-parallel-studio-cluster.2020.4-5mxdw276vo2p6wtdkoaghj5h2zkmozjt/compilers_and_libraries_2020.4.304/linux/bin/intel64/icc
      cxx: /usr/local/pace-apps/spack/packages/linux-rhel7-x86_64/gcc-4.8.5/intel-parallel-studio-cluster.2020.4-5mxdw276vo2p6wtdkoaghj5h2zkmozjt/compilers_and_libraries_2020.4.304/linux/bin/intel64/icpc
      f77: /usr/local/pace-apps/spack/packages/linux-rhel7-x86_64/gcc-4.8.5/intel-parallel-studio-cluster.2020.4-5mxdw276vo2p6wtdkoaghj5h2zkmozjt/compilers_and_libraries_2020.4.304/linux/bin/intel64/ifort
      fc: /usr/local/pace-apps/spack/packages/linux-rhel7-x86_64/gcc-4.8.5/intel-parallel-studio-cluster.2020.4-5mxdw276vo2p6wtdkoaghj5h2zkmozjt/compilers_and_libraries_2020.4.304/linux/bin/intel64/ifort
    flags: {}
    operating_system: rhel7
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
  ```
  You should change the paths based on the file locations in your system. 
  Then you will be able to see the compiler added to the list by typing: spack compilers
#### 3. Download Packages Using 'spack install'
```
module load intel/20.0.4

time spack install mpich%intel@20.0.4.304

time spack install openmpi %intel@20.0.4.304

time spack install netcdf-fortran %intel@20.0.4.304^mpich

time spack install netcdf-c %intel@20.0.4.304^mpich

time spack install flex %intel@20.0.4.304

time spack install cmake %intel@20.0.4.304

time spack install gmake %intel@20.0.4.304

time spack install ncview %intel@20.0.4.304^mpich

spack load texinfo@6.5%intel@20.0.4.304

time spack install cgbd %intel@20.0.4.304
```

\# see also https://github.com/geoschem/geos-chem-cloud/issues/35

(Note: if you see this error when installing new packages,  `Error: Package 'cray-mpich' not found.`, add `^mpich` )

\####################################################################

\# if you are in Pengfei Liu’s group at Gatech 

\# just start from here (spack is already installed)!!###

\####################################################################

#### 4. Download the source code 

For version 14.0.0 or later

generate a new directory (e.g. GC14) and download the source code here
```
mkdir GC14
cd GC14
```
\# If you want to track your code change with Github, you will first need to fork the repository "GCClassic" to your Github account, then clone the repo from your account: 
```
git clone --recurse-submodules https://github.com/<your github account>/GCClassic.git Code.X.Y.Z
```
or you can simply download the source code from the official git repository
```
git clone --recurse-submodules https://github.com/geoschem/GCClassic.git Code.X.Y.Z
```
Here I will name the code folder with the code version, i.e., The name of my folder is Code.14.3.0. By default you will download the most recent version. You can check the version by:
```
git log -n 1
```
#### 5. (Optional) Link the repo on the cluster (remote repo) to the repo on your GitHub account (central repo)
Only if you want to track your code change with Github
```
git remote add upstream <Central repo URL>
```
##e.g., for me, it is: `git remote add upstream https://github.com/bzhang419/GCClassic.git`

Then if you type `git remote -v`, origin should point to your remote repo and upstream should point to the central repo like this:
```
origin  https://github.com/bzhang419/GCClassic.git (fetch)
origin  https://github.com/bzhang419/GCClassic.git (push)
upstream        https://github.com/bzhang419/GCClassic.git (fetch)
upstream        https://github.com/bzhang419/GCClassic.git (push)
```
#### 6. (Optional) Make changes on your remote repo (origin) and update the change on your central repo (main)
1) make some change
2) 
   ```
   git add .
   git commit -m "info of your change"
   git push origin main
   ```
   Then you will be asked to enter your username and password (Note that from 2021-08-13, GitHub no longer accepts account passwords when authenticating Git operations. Instead of entering your GitHub password, you need to enter your PAT (Personal Access Token). You can generate one by navigating your account -> settings -> developer settings -> personal access tokens -> tokens (classic) -> generate new token. Enter this instead of your GitHub password).
   
   Every time you perform a git action, you will need to enter the user name and token. If you want to avoid this, you can store your credentials in cache by entering `git config --global --replace-all credential.helper cache`
   
#### 7. Load your environment
Before you compile or run GEOS-Chem, you will need to set up the environment. The configuration files ".GC" can be found in the shared folder from Dr. Liu's group. Copy that in your own directory and execute
```
source ~/.GC
```
#### 8. Create a run directory
```
cd run/
./createRunDir.sh
```
This step will create a run directory that contains files for your simulation.
#### 9. Configure your build, compile and install
```
# navigate to the run directory you just created
cd /path/to/gc_4x5_merra2_fullchem  # Skip if you are already here
cd build
# configure your build
cmake ../../Code14.3.0 -DRUNDIR=..
# compile and install
make -j
make install
```
If the compilation is successful, you will see an executable `gcclassic` in your run directory
#### 10. Configure your simulation
Navigate to your run directory '/path/to/gc_4x5_merra2_fullchem', review the following files before your simulations and edit them based on your simulation: 
```
vi geoschem_config.yml
vi HISTORY.rc
vi HEMCO_Diagn.rc
vi HEMCO_Config.rc
vi HEMCO_Config.rc.gmao_metfields
```
For more info of the configuration files above, refer to GEOS-Chem wiki.
#### 11. Submit a job to run GEOS-Chem
You can find a sample sbatch script in the Liu group shared folder: `/storage/coda1/p-pliu40/0/shared/GEOS-Chem/slurm/GC12.9.3/run_GC12.sbatch`, copy one to your run directory, make some edits, and submit it by:
```
sbatch xxx.sbatch
```

