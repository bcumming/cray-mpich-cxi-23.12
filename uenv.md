# Using user environments

First things first, install a command line tool that will make your life easier:

```bash
# log into tasnam and run the following
# type yes when prompted
git clone git@github.com:eth-cscs/uenv.git && ./uenv/install

# log out and back in again, the following should tell you that no environment is loaded
uenv status
```

```bash
# start a new shell with the new environment mounted:
uenv start /scratch/mch/bcumming/tasna-v5_cray-mpich-8.1.26.sqfs:/mch-environment/v5

# check that it has been mounted (it should generate some information)
uenv status

# make the modules available
uenv modules use
module avail

# load one of the views, of which there are two options:
#   the gcc-serial env provides cray-mpich, osu, cmake, etc all built with gcc
uenv view prgenv-gcc-serial
#   the icon env provides cray-mpich, osu built with nvhpc, and some other tools built with gcc
uenv view prgenv-icon
```

By default the images are mounted at `/user-environment`, hence why we have to add the `:/mch-environment/v5` after the squashfs file to specify that we want a non-standard mounte point.

If Simon provides you with a simpler testing env, you don't need that part (`uenv status` will print a warning message if an image has been mounted at a location it wasn't built to be mounted at).

If you have a workflow like the following:

```
uenv start image.sqfs
srun -n2 ...
```

We have a slurm plugin that will detect that you had an image mounted in your session, and automatically mount it on the compute nodes.

You can specificy which image to mount (and optionally where) using the `--uenv` flag to srun or sbatch:

```bash
srun --uenv=/scratch/mch/bcumming/tasna-v5_cray-mpich-8.1.26.sqfs:/mch-environment/v5 ...
```

or in an sbatch script:
```
#SBATCH --uenv=/scratch/mch/bcumming/tasna-v5_cray-mpich-8.1.26.sqfs:/mch-environment/v5
```
