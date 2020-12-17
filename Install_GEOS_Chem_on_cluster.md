### Installation of GEOS-Chem on Georgia Tech Phoenix cluster

> Contributed By Bai Bin

> Contact Info: baib@mail.sustech.edu.cn

> Last Updated: 2020/12/15

More Info at http://wiki.seas.harvard.edu/geos-chem/index.php/Getting_Started_with_GEOS-Chem

1. download SPACK in home directory

git clone https://github.com/spack/spack.git

2. load SPACK

export SPACK_ROOT=/storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack

source $SPACK_ROOT/share/spack/setup-env.sh

\# or just

\# source /storage/coda1/p-pliu40/0/shared/GEOS-Chem/spack/share/spack/setup-env.sh

3. download Packages Using 'spack install'
module load intel/19.0.5
time spack install netcdf-fortran %intel@19.0.5.281
time spack install flex %intel@19.0.5.281
time spack install cmake %intel@19.0.5.281
time spack install gmake %intel@19.0.5.281
time spack install ncview %intel@19.0.5.281
spack load texinfo
time spack install cgbd %intel@19.0.5.281
\# see also https://github.com/geoschem/geos-chem-cloud/issues/35
