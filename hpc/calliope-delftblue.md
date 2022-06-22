# HOWTO: Basic use of Calliope on the `DelftBlue` cluster

## Installation (basic Calliope + Gurobi)

Having set-up the basic cluster access (see the cluster [Wiki](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/home)), you need first of all to load some software modules with the following commands:

```bash
module load 2022r2
module load  miniconda3/4.10.3-eyq4jvx
```

For a basic Calliope+Gurobi installation (i.e., you can skip this if you plan to use Euro-Calliope or similar software in which Calliope is already sub-package) you should then be able to run:

```bash
conda create -c conda-forge -c gurobi -n calliope calliope gurobi
```
However, as it happened for INSY, the one-line conda command may fail on the cluster. In that case, you can still use this less-than-ideal fallback:

```bash
conda create -n calliope
conda activate calliope
conda install -c gurobi gurobi
conda install -c conda-forge calliope
```

## File transfer from local machine to cluster

Most default Linux file managers support SFTP out of the box. By simply putting `sftp://<netid>@login.delftblue.tudelft.nl` in the address bar you can access the cluster on your local machine and easily transfer files. 

For different OS, check out the cluster [Wiki/Data-transfer](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/Data-transfer-to-DelftBlue).


## Installation (specific Calliope environment + Gurobi)

If you want to install a different version of Calliope, say one that runs within a broader Euro-Calliope environment, you can avoid installing the basic Calliope and install the desired version directly. 

For North-Sea Calliope (but the same applies to other Euro-Calliope pre-built models), for instance, you need to first move the `requirements.yml` to your home or proejct directory on the cluster. Then, move to such a directory and run the following:

```bash
conda env create -f requirements.yml
``` 

One the environment is created, activate it (beware: you may need to log off and log in again to be able to do so): 

```bash
conda activate northseacalliope_2022_05_20
```

and install Gurobi via: 

```bash
conda install -c gurobi gurobi
```

## Loading modules by default

If you do not want to load the default software modules any time you log in, you can set them as default modules.

Look for: `/home/<netid>/.bashrc`. The file might be hidden if you have set up file transfer from your local machine, in which case you need to unlock the possibility to view hidden files first.

Edit this file (e.g., with a text editor) and add the following lines to the end of the file:

```bash
module load 2022r2
module load  miniconda3/4.10.3-eyq4jvx 
```

This will make sure that such modules are always loaded whenever you login.

## Adding a Gurobi license

To solve models, you need to set the Gurobi solver to look for the TU Delft license server. Create a file gurobi.lic in your home directory:

```bash
nano ~/gurobi.lic
```

Add the following content to the file:

```
TYPE=TOKEN
TOKENSERVER=flexserv-x1.tudelft.nl
PORT=27099
```

## Seting up a cluster job and running

To run your model, you'll need to set up a cluster job. That's the only additional step required compared to running on your local machine. For generic information of what this means, you can once more have a look at the cluster [Wiki/SLURM-scheduler](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/Slurm-scheduler).

In a nutshell, all you need to do is creating an .sbatch file that specifies the requirements of your job.
To do so, create an empty text file and name it, e.g., `test_job.sbatch`. Then, open it with a text editor and make sure that the file contains the following information (see [Wiki/SLURM-scheduler](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/Slurm-scheduler) for the meaning of each line):

```
#!/bin/sh

#SBATCH --partition=compute
#SBATCH --time=01:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem-per-cpu=8G
#SBATCH --mail-type=END
#SBATCH --account=research-tpm-ess ### If you are a MSc student, please use the following account code: --account=education-<faculty>-msc

srun ../my_national_model_run.py
```

In this case, the example file is pointing to a python script (`my_national_model_run.py`) that is what you are asking to run. Please notice that the path to the file is preceded by `../`. This means that the path is one directory level up compared to where this example `test_job.sbatch` is stored.

To be more precise, in this example the `test_job.sbatch` is contained in a folder called `run_on_cluster` within the main `2022-05_20_lite_north-sea/` directory. By going one level up, the run command moves from `2022-05_20_lite_north-sea/run_on_cluster/`, where you need to be to run `test_job.sbatch`, back to the main `2022-05_20_lite_north-sea/` directory.

So, assuming you are in your main model folder, to run this example model you'd need to do:

```bash
cd run_on_cluster
sbatch test_job.sbatch
```

A few minutes later you should see an output log file of the with a name like `slurm-[job_id].out`. It should show that Calliope ran successfully.

You should also see the results from the model run in your results folder.

To check the status of a job while it is in the queue, you can type: `squeue -u <your-username>`
