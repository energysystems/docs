# HOWTO: Basic use of Calliope on the `DelftBlue` cluster

## Installation (basic Calliope + Gurobi)

Having set-up the basic cluster access (see the cluster [Wiki](https://doc.dhpc.tudelft.nl/delftblue/)), you need first of all to load some software modules with the following commands:

```bash
module load 2022r2
module load  miniconda3/4.12
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

A general recommendation is to learn and use [`rsync`](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/Data-transfer-to-DelftBlue#rsync), which is useful beyond DelftBlue and runs on all operating systems.

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

You can confirm that your installation is licensed by running `gurobi_cl --tokens`, which should show you how many licenses are currently in use.

## Setting up a cluster job and running

To run your model, you'll need to set up a cluster job. That's the only additional step required compared to running on your local machine. For generic information of what this means, you can once more have a look at [DelftBlue's documentation on the Slurm scheduler](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/Slurm-scheduler).

In a nutshell, all you need to do is creating an .sbatch file that specifies the requirements of your job.
To do so, create an empty text file and name it, e.g., `test_job.sbatch`. Then, open it with a text editor and make sure that the file contains the following information (see [Slurm scheduler](https://gitlab.tudelft.nl/dhpc/docs/-/wikis/Slurm-scheduler) for the meaning of each line):

```
#!/bin/sh

#SBATCH --partition=compute
#SBATCH --time=01:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu=64G
#SBATCH --mail-type=END
#SBATCH --account=research-tpm-ess ### If you are a MSc student, please use the following account code: --account=education-<faculty>-msc

srun ../my_national_model_run.py
```

```{important}
Do not forget that the memory requirements you specify are per CPU. In the above example, you are requesting two CPUs and 64 GB of memory per CPU, for a total of 128 GB of memory. See further below for information on how to determine how many CPUs and how much memory you need.
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


## Determining the resources to request on the cluster

It is important to request just enough resources on the cluster for your job to run, and no more, in order not to waste computational resources. This applies both to the number of CPUs requested and the amount of memory. Generally, Calliope does not benefit from having a large number of CPUs available since the computationally most intensive part - solving the optimisation problem - is not parallelisable. So it makes sense to request few (e.g. two) CPUs. Memory use is more difficult. Large models require a lot of memory. If you are unsure about the memory required, you can run a first test job and once it has run, look at the resources actually used:

`seff [job_id]`

The output of `seff` will tell you what resources you requested, what resources you actually used, and the resulting resource use efficiency. Ideally you want to be as close to 100% as possible so as not to waste resources that are reserved for you on the cluster but never used.

```{important}
Be a good citizen on the cluster and take your resource use efficiency seriouly! If you consistently request more resources than you need, do not be surprised if you are contacted by cluster support and/or if your access to the cluster is limited.
```


## Running out of space in /home directory

Package managers such as conda are known to install a lot of tiny files locally. These local installations might occupy a lot of space and often use your `/home` directory as their default destination.
No problem! You might want to create these directories on the `/scratch` storage and link to them in your `/home` directory.

You can do that by simply moving your `/home/${USER}/.conda` directory to your `/scratch` and then link it to the `/home` folder. In this way, the `/home` folder will contain only the link to the `/scratch/${USER}/.conda` directory.

```bash
ln -s /scratch/${USER}/.conda $HOME/.conda
```

In case you are setting up the link before creating any enviroment you might want to create an empty `/.conda` folder in your `/scratch`.

```bash
mkdir -p /scratch/${USER}/.conda
```

In any case, you will not need to create the link to any other working folders. You can simply load your working folder (project folder, model-config folder, your script folder, etc.) on the `/scratch` directory.
For additional info check the [DelftBlue docs](https://doc.dhpc.tudelft.nl/delftblue/howtos/conda/).
