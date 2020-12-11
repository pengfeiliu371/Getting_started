## Running Jupyter jobs on Georgia Tech new HPC environment (Pheonix)
> by Pengfei Liu (pengfei.liu@eas.gatech.edu)

1. Setup Anaconda on PACE cluster

Follow the PACE document for conda, step 1-6:
https://docs.pace.gatech.edu/software/anacondaEnv/

Here I created an evironment named "geo".

2. Install packages

Here I set conda-forge as the default channel, and download all packages from this channel.

Packages installed from different channels can be conflicting with each other.
> conda config --add channels conda-forge 

> conda config --set channel_priority strict
