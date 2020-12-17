### Installation of GEOS-Chem on Georgia Tech Phoenix cluster (using spack)

> Contributed By Bai Bin

> Contact Info: baib@mail.sustech.edu.cn

> Last Updated: 2020/12/15

More Info at http://wiki.seas.harvard.edu/geos-chem/index.php/Getting_Started_with_GEOS-Chem

#### 1. download SPACK in home directory

git clone https://github.com/spack/spack.git

#### 2. load SPACK

export SPACK_ROOT=/storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack

source $SPACK_ROOT/share/spack/setup-env.sh

\# or just

\# source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack/share/spack/setup-env.sh

#### 3. download Packages Using 'spack install'

module load intel/19.0.5

time spack install netcdf-fortran %intel@19.0.5.281

time spack install flex %intel@19.0.5.281

time spack install cmake %intel@19.0.5.281

time spack install gmake %intel@19.0.5.281


\####################################################################
\# if you are in Pengfei Liu’s group at Gatech, 
\# just start from here!!###
\####################################################################

#### 4. download Source Code in directory 'GC'

git clone https://github.com/geoschem/geos-chem Code.X.Y.Z

#### 5. download Unit test in directory 'GC'

git clone https://github.com/geoschem/geos-chem-unittest UT.X.Y.Z

#### 6. copy run directory

cd ~/GC/UT.X.Y.Z/perl/

vi CopyRunDirs.input

\# some path need change

\# root path should be /storage/coda1/p-pliu40/0/shared/GEOS-Chem

\# other pathes are based on your own setting

./gcCopyRunDirs

#### 7. set GEOS-Chem task

\# come to the directory you just created, here I show my setting

cd {home}/GC/rundirs/test1/merra2_4x5_standard

vi input.geos # change run time and others

vi HEMCO.Config.rc # shut-down several offline emissions

vi HISTORY.rc #choose preferred output

time spack install ncview %intel@19.0.5.281

spack load texinfo

time spack install cgbd %intel@19.0.5.281

\# see also https://github.com/geoschem/geos-chem-cloud/issues/35
