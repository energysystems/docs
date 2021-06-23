# Euler cluster

## Getting up and running

- Its easiest to use coda for Python, installing it into `/cluster/work/apatt/$username.Download 64-bit linux miniconda installer here: [https://conda.io/miniconda.html](https://conda.io/miniconda.html)
- Modules that may be useful — possibly always load them by putting them into `.bashrc` or similar:
    - Gurobi commercial optimisation solver for Calliope: `module load gurobi[/<version>]`
    - NetCDF operators (NCO): `module load nco`
    - Ncview (a simple netcdf viewer): `module load ncview`
    - Ncdump (lets you inspect netcdf metadata): `module load netcdf`

## Cluster-driven Jupyter lab

- Grab the files from here: [https://gist.github.com/brynpickering/78ed2b32792cfd73997ff148b941a265](https://gist.github.com/brynpickering/78ed2b32792cfd73997ff148b941a265)
- To set a password, run `jupyter notebook password` once
- Deploy `run_jupyter.sh` and `[jupyter.sh](http://jupyters.sh)` (see gists below) to euler
- Execute `run_jupyter.sh` from a login node and follow the instructions printed out to connect

## Running jobs

**Running a job in one line:**
`bsub [LSF options] "python my_python_script.py"`
where [LSF options] can be:
    -n: number of cores
    -I: interactive
    -o: output file
    -R: memory allocation per requested core, "rusage[mem=XX]"
    -W: time, int if minutes, or as HH:MM(?)
e.g.
`bsub -n 1 -W 4:00 -oo output.txt 'python training.py --njobs 4 --draws 10000'`
`bsub -n 4 -R "rusage[mem=30G]" -o low_cost.log -W 600 'calliope run model.yaml --output-file=output.nc --scenario=low-cost'`



**Interactive jobs:**

Can be useful if you want/need to work interactively on the cluster (e.g., running IPython, pdb). Interactive jobs are handled similarly to normal jobs (same job categories and syntax).

As a prerequisite you need to enable X11 forward on your euler account. To do so, ssh into a login node and run */cluster/apps/local/setup_ssh.sh*

Example:

```bash
bsub -Is -XF -R "rusage[mem=3000]" bash
```

where -Is stands for interactive, -XF enables X11 forwarding that is needed sometime (e.g., when using paste or %paste in IPython) and -R "rusage[mem=3000] allocations 3GB RAM

**Accessing output of running jobs:**

```bash
bpeek $JOBID
```

The JOBID of submitted jobs can be found using bbjobs

**Modify resources of already running jobs**

If you find that you did not allocate enough memory or computing time in the submission of the job, you can modify this will the job is running. This can only happen within certain bounds. If, for example, your job is submitted to the queue normal.4h, you cannot increase the runtime beyond 4 hours.

Example

```bash
bmod -W 04:00 $JOBID
```

## Job Arrays

To submit a set of individual jobs that share the same code but use different parameters, jobs can be submitted as an array.

```bash
#BSUB -J job_name[0-14]
```

To restart some of the jobs (say, numbers 0,4,6,7,8), the expression in brackets is to be changed:

```bash
#BSUB -J job_name[0,4,6-8]
```

## Overcoming the Firewall

Since Euler was 'hacked', it has become more cautious with connection to external servers. This include those needed to install Python packages with e.g. conda.

To allow conda to connect to the Anaconda servers, add the following to `.condarc` in your home directory

```bash
proxy_servers:
    http: [http://proxy.ethz.ch:3128](http://proxy.ethz.ch:3128/)
    https: [http://proxy.ethz.ch:3128](http://proxy.ethz.ch:3128/)
```

For git:

```bash
git config --global http.proxy http://proxy.ethz.ch:3128
git config --global https.proxy http://proxy.ethz.ch:3128
```

For pip and curl, add the following to your `.bash_profile`:

```bash
export https_proxy=http://proxy.ethz.ch:3128
export http_proxy=http://proxy.ethz.ch:3128
```

## Receiving notifications

When executing long-running workflows on Euler, you may want to get notified whenever workflows terminate. For individual jobs, you can use [built-in mechanisms](https://scicomp.ethz.ch/wiki/LSF_mini_reference). For entire workflows, you can, in principle, send emails to yourself. However, this feature has been disabled with the [reopening of Euler in May 2020](https://scicomp.ethz.ch/wiki/Reopening_of_Euler_and_Leonhard_(May_2020)).

Alternatively, and as long as email sending is not possible, you may want to use [IFTTT](https://ifttt.com/my_applets). IFTTT can send you mails, send push notifications to your phone, and do all sorts of much more weird things. The idea is quite simple: you trigger a webhook and that triggers a notification service. Using Python you can do the following to trigger webhooks:

```python
import requests

EVENT_NAME = "my_event"
API_KEY = "my_api_key"

requests.post(
    f'https://maker.ifttt.com/trigger/{EVENT_NAME}/with/key/{API_KEY}'
)

```

In addition, you need an IFTTT applet that listens to the webhook and reacts to it. [Here's a full example](https://pimylifeup.com/using-ifttt-with-the-raspberry-pi/) that shows how to build and trigger the applet.
