# HOWTO: Basic use of Calliope on the `INSY` cluster

## Installation

To install the latest version of [Calliope](https://www.callio.pe/) and the Gurobi solver into a conda environment called `calliope`.

Ideally, you should be able to do so by using a one-line conda command:

```bash
module use /opt/insy/modulefiles
module load miniconda/3.9
conda create -c conda-forge -c gurobi -n calliope calliope gurobi
```

However, the one-line conda command often fails on the cluster. In that case, you can still use this less-than-ideal fallback:
```bash
module use /opt/insy/modulefiles
module load miniconda/3.9
conda create -n calliope
conda activate calliope
conda install -c gurobi gurobi
conda install -c conda-forge calliope
```

Afterwards, confirm that Calliope is installed and works by typing:

```bash
conda activate calliope
calliope --version
```

This should simply print the installed version, such as "`Version 0.6.6`", and show no errors.

To always enable miniconda each time you log in to the cluster, you should edit the file ``.bash_profile`` in your home directory:

```bash
nano ~/.bash_profile
```

Add the following to the end of the file:

```
module use /opt/insy/modulefiles
module load miniconda/3.9
```

To solve models, you need to set the Gurobi solver to look for the TU Delft license server. Create a file `gurobi.lic` in your home directory:

```bash
nano ~/gurobi.lic
```

Add the following content to the file:

```
TYPE=TOKEN
TOKENSERVER=flexserv-x1.tudelft.nl
PORT=27099
```

## Run a simple Calliope model through the cluster scheduler

Create a sample model:

```bash
calliope new test
```

Modify the sample model configuration:

```bash
nano test/model.yaml
```

Ensure that in `model.yaml`, the solver is set to Gurobi, with the Python interface:

```yaml
solver: gurobi
solver_io: python
```

Create a job specification file, calling it e.g. `jobscript.sbatch`, with this content:

```
#!/bin/sh

# The default partition is the 'general' partition
#SBATCH --partition=general

# The default Quality of Service is the 'short' QoS (maximum run time: 4 hours)
#SBATCH --qos=short

# The default run (wall-clock) time is 1 minute
#SBATCH --time=0:05:00

# The default number of parallel tasks per job is 1
#SBATCH --ntasks=1

# Request 1 CPU per active thread of your program (assume 1 unless you specifically set this)
# The default number of CPUs per task is 1 (note: CPUs are always allocated per 2)
#SBATCH --cpus-per-task=1

# The default memory per node is 1024 megabytes (1GB) (for multiple tasks, specify --mem-per-cpu instead)
#SBATCH --mem=1024

# Set mail type to 'END' to receive a mail when the job finishes
# Do not enable mails when submitting large numbers (>20) of jobs at once
#SBATCH --mail-type=END

# Your job commands go below here

module use /opt/insy/modulefiles
module load miniconda/3.9
conda activate calliope
srun calliope run test/model.yaml --save_netcdf test_results.nc
```

Submit your job to the cluster:

```bash
sbatch jobscript.sbatch
```

A few minutes later you should see an output log file of the with a name like `slurm-[job_id].out`. It should show that Calliope ran successfully.

You should also see the results from the model run in the form of a new file, `test_results.nc`.

## Where to go next

See here for more documentation [about job running on the cluster](https://docs.google.com/presentation/d/10A0_0eNRBYd87E1h1YN6bsIFaZaua5qJkfBbnBKAr6o/present) and [about the cluster more generally](http://insy.ewi.tudelft.nl/content/hpc-cluster) (e.g., how to transfer files).

## Tips & tricks for a smooth use of Calliope

When using Calliope with the Gurobi solver, the latter might lead to unwanted CPU usage and to the job being killed by HPC staff. In fact, by default, Gurobi tries to launch multiple optimisation algorithms in parallel when searching for the problem solution, both to check which one is faster and to have a more robust numeric result. However, this means in practice that Gurobi ignores the requested amount of threads allocated (in the `jobscript.sbatch`) to solve the problem and starts spawning additional threads and processes, which makes CPU usage skyrocket far above what originally requested. 

Trying to increase the number of CPU's in your jobscript is (in general) a bad workaround. Typically, you do not need so many threads for Gurobi to be fast enough in finding a solution; and, after a certain progressive number of parallel threads, the speed of resolution does not change at all.

So, you may want to ask Calliope to use less threads for the Gurobi solver. This is configurable in the YAML config file of Calliope itself, as described on its [documenation](https://calliope.readthedocs.io/en/stable/user/advanced_features.html)
