## Interactive geodata analysis on Georgia Tech Phoenix HPC 
> by Pengfei Liu (pengfei.liu@eas.gatech.edu)

This tutorial is for the new GaTech phoenix cluster. The Anaconda software has been installed, and a "pace-jupyter-notebook" script has been develped by PACE support team. If you are interested to install Anaconda from scratch, the tutorial in the link below might be useful:
https://github.com/Environment-and-Seniors-Health-Emory/Getting_started/blob/main/HPC_cluster_GUI_Jupyter_Guide.md

### 1. Setup Anaconda environment on PACE cluster

Follow the PACE document for conda, step 1-6:
https://docs.pace.gatech.edu/software/anacondaEnv/

In step 4, I created an evironment named "geo".

### 2. Install packages

Here I set conda-forge as the default channel, and download all packages from this channel.
Packages installed from different channels can be conflicting with each other.

> conda config --add channels conda-forge 

> conda config --set channel_priority strict

Install packages for geo data analysis:

> conda install esmpy

(The esmpy package is needed for regridding. Currently, it is not working on windows computers. It is highly recommended to use on cluster. If you really want to use it on a windows computer, a workaround is to use docker. https://xesmf.readthedocs.io/en/latest/installation.html)

> conda install regionmask geopandas

> conda install xesmf dask matplotlib cartopy netcdf4 descartes

> conda install jupyter

Install any other packages as needed.

These packages should be adequate for analysis and visaulization of NETCDF data. xESMF is the regridding package. GeoPandas is used to handle .shp files. I will later post some examples. 

If you are familiar with NCL, you may check out these packages:
http://www.pyngl.ucar.edu/

If you work with GEOS-Chem output, GCPy might be useful:
https://github.com/geoschem/gcpy


\###############################################################

\# start from here once environment is successfully configured

\###############################################################

### 3. Use jupyter notebook on PACE

load Anaconda:

> module load anaconda3

Run the following command to request for 1 node, 4 CPUs, 32gb memory, 4 hours wall time: 

> pace-jupyter-notebook -q inferno -A GT-pliu40 -l nodes=1:ppn=4,mem=32gb,walltime=4:00:00 --conda-env=geo

You will need an account to be charged on for using the new phoenix cluster (here GT-pliu40 is the account for my group). To check what accounts you can use, type:
> mam-balance

Once the requested computational resource is allocated, follow the instruction on the screen. For mac users, you will need to press shift+~, shift+C (quickly) to get > ssh, and add ssh port forwarding option to your running ssh session. (I need to test if it works for windows users...)

Then, copy and paste the sever address to your browser on your local computer, Jupyter should be ready to use.

For more informaiton, refer to the PACE document below:
https://docs.pace.gatech.edu/interactiveJobs/jupyterInt/

### 4. How to kill a job

Once you finish the interactive session, kill the job:

> qdel job-id

You will find your job_id by typing:

> qstat -u username

You may also find files like "pace-jupyter-notebook.pbs.oXXXXXXXX" in your home directory. The number in the most recent file should be your job_id. 
