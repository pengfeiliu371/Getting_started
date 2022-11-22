### Installation of GEOS-Chem on Georgia Tech Phoenix-Slurm cluster (using spack)

> Contributed By Bin Bai & Bingqing Zhang

> Contact Info: bbai32 [at] gatech.edu & bzhang419 [at] gatech.edu

> Last Updated: 2022/11/22

More Info at http://wiki.seas.harvard.edu/geos-chem/index.php/Getting_Started_with_GEOS-Chem

\####################################################################

\# if you are in Pengfei Liu’s group at Gatech 

\# skip step 1-3 and start from step 4 

\# spack is already installed ###

\####################################################################

#### 1. download SPACK in home directory

git clone https://github.com/spack/spack.git

#### 2. load SPACK

export SPACK_ROOT=/storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack

source $SPACK_ROOT/share/spack/setup-env.sh

\# or just

\# source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack/share/spack/setup-env.sh
##### 2.1 make sure your compiler is detected by SPACK
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
#### 3. download Packages Using 'spack install'

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

\# see also https://github.com/geoschem/geos-chem-cloud/issues/35

(Note: if you see this error when installing new packages, " Error: Package 'cray-mpich' not found.", add "^mpich" )

\####################################################################

\# if you are in Pengfei Liu’s group at Gatech 

\# just start from here (spack is already installed)!!###

\####################################################################

#### 4. download Source Code in directory 'GC'

For version 12.9.3 or ealier:

git clone https://github.com/geoschem/geos-chem Code.X.Y.Z

For version 13.0.0 or later

git clone https://github.com/geoschem/GCClassic GCClassic.13.0.0

http://wiki.seas.harvard.edu/geos-chem/index.php/Downloading_GEOS-Chem_source_code_(13.0.0_and_later_versions)

#### 5. download Unit test in directory 'GC'

git clone https://github.com/geoschem/geos-chem-unittest UT.X.Y.Z

#### 6. copy run directory

cd ~/GC/UT.X.Y.Z/perl/

vi CopyRunDirs.input

\# some path need to be changed

\# root path should be /storage/coda1/p-pliu40/0/shared/GEOS-Chem

\# other pathes are based on your own setting

./gcCopyRunDirs

#### 7. set GEOS-Chem task

\# come to the directory you just created, here I show my setting

cd {home}/GC/rundirs/test1/merra2_4x5_standard

vi input.geos # change run time and others

vi HEMCO.Config.rc # shut-down several offline emissions

vi HISTORY.rc #choose preferred output

#### 8. build run environment

\# do below in your run directory, here i.e. {home}/GC/rundirs/merra2_4x5_standard

source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/.bashrc (or you can copy it to your own dir)

source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/.GC (or you can copy it to your own dir)

make realclean

make -j16 build

#### 9. submit task

sbatch xxx.sbatch

\# see template in /storage/coda1/p-pliu40/0/shared/GEOS-Chem/slurm/GC12.9.3/run_GC12.sbatch

\# copy it to your own run dir and modify

#### 10. check your task state

squeue -u <username>

\# for me, it’s squeue -u bbai32


