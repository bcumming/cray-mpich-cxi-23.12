The issue has to be reproduced on tasnam, because
* it was built with the CSCS uenv mounted at /mch-environment/v5
* rebuilding ICON on a HPE test system is not for the faint hearted

We can use a copy of ICON that has been built by MCH, by first copying it to a path in your SCRATCH and making a few tweaks to the run script.

```
# path where Fabian installed ICON
export ipath=/scratch/mch/fgessler/ICON/master/tasnam
# run script
export runscript=exp.mch_opr_r04b07.run

# grab a local copy of ICON
# use rsync to handle the symlinks in ICON's build
# this takes about 5 minutes to copy everything
rsync -avl $ipath .

# move to the tasnam/run path in your fresh copy
# the job script is exp.mch_opr_rpath where Fabian installed ICON
# we want to modify it to it's new location:
cd tasnam/run
sed -i 's|/scratch/mch/fgessler/ICON/master/tasnam|/scratch/mch/bcumming/icon-test/tasnam|g' $runscript

#Add the following in $runscript:
#SBATCH --reservation=cpe

sbatch exp.mch_opr_r04b07.run
sbatch $runscript

# the output is in LOG.*
```

The error message that we see (just above th bottom of the output):

```
 mo_async_latbc_utils::check_validity_date_and_print_filename:: reading boundary data from file /scratch/mch/jenkins/icon/pool/data/ICON/mch/input/opr_r04b07/efsf00000000_lbc.nc for date: 2020-12-10T06:00:00.000
MPICH ERROR [Rank 5] [job id 40341.0] [Tue Dec 12 17:52:24 2023] [nid001097] - Abort(404347151) (rank 5 in comm 0): Fatal error in PMPI_Send: Other MPI error, error stack:
PMPI_Send(163)................: MPI_Send(buf=0x2857bb8, count=1, MPI_INTEGER, dest=0, tag=2001, comm=0x84000001) failed
MPID_Send(499)................:
MPIDI_send_unsafe(58).........:
MPIDI_OFI_send_lightweight(34): OFI tagged injectdata failed (ofi_send.h:34:MPIDI_OFI_send_lightweight:Invalid argument)

aborting job:
Fatal error in PMPI_Send: Other MPI error, error stack:
PMPI_Send(163)................: MPI_Send(buf=0x2857bb8, count=1, MPI_INTEGER, dest=0, tag=2001, comm=0x84000001) failed
MPID_Send(499)................:
MPIDI_send_unsafe(58).........:
MPIDI_OFI_send_lightweight(34): OFI tagged injectdata failed (ofi_send.h:34:MPIDI_OFI_send_lightweight:Invalid argument)
 mo_util_cdi:deleteInputParameters: Total read statistics for stream ID
           24
        amount:               56153152 bytes
        duration:                0.064 seconds
        bandwidth:             840.793 MiB/s
 mo_grid_distribution_base:scatterPatternPrintStatistics
 : data distribution totals:
        amount:               44378980 bytes
        duration:                0.035 seconds
        bandwidth:            1194.165 MiB/s
 mo_util_cdi:deleteInputParameters: Total read statistics for stream ID
           24
        amount:               12290544 bytes
        duration:                0.012 seconds
        bandwidth:             945.657 MiB/s
 mo_grid_distribution_base:scatterPatternPrintStatistics
 : data distribution totals:
        amount:                9685900 bytes
        duration:                0.007 seconds
        bandwidth:            1286.824 MiB/s
 mo_async_latbc_utils::check_validity_date_and_print_filename:: reading boundary data from file /scratch/mch/jenkins/icon/pool/data/ICON/mch/input/opr_r04b07/efsf00030000_lbc.nc for date: 2020-12-10T09:00:00.000
srun: error: nid001097: task 5: Exited with exit code 255
srun: Terminating StepId=40341.0
slurmstepd: error: *** STEP 40341.0 ON nid001096 CANCELLED AT 2023-12-12T17:52:24 ***
srun: error: nid001097: tasks 1,3: Terminated
srun: error: nid001096: tasks 0,2,4: Terminated
srun: Force Terminated StepId=40341.0
+ exit 1
```

## using a different icon binary

* *NOTE* here we use a binary that was built using `cray-mpich 8.1.26` with the files we copied for the version build with `8.1.18`

Because the ICON binary is built with rpath and static linking, we can directly copy a binary built elsewhere into the `bin` path of the directory that we copied above without having to copy all the other files again.

e.g. if I am in the `run` path:
```
┌ tasna:tasnam-ln002 /scratch/mch/bcumming/icon-test/tasnam/run
└── cp /scratch/mch/fgessler/ICON/master/tasnam_PE/bin/icon ../bin/icon-g2g-mpich8.1.26
```

Make a backup of the old icon executable, and replace it with your new version.

So that the new binary can find the libraries that it was linked against, we have to add a flag to the sbatch file `exp.mch_opr_r04b07.run`:

```
#SBATCH --reservation=cpe
#SBATCH --uenv=/scratch/mch/bcumming/tasna-v5_cray-mpich-8.1.26.sqfs:/mch-environment/v5
```

See the file `uenv.md` in this repository for more information on how this works.
