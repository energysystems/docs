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

## Handling non-LSF tasks with long runtime

Sometimes, you may want to perform tasks with long runtime and you do not want or cannot run them on LSF. One such example is Snakemake, whose main process runs on the login nodes of Euler issuing jobs to LSF one after the other. Such Snakemake workflows can easily run for hours or even days.

Long-running tasks can be tricky, as tasks running within a shell session get killed whenever the session gets closed. A session can often be closed unintentionally when your (VPN) connection to Euler drops.

There are two ways of ensuring your long-running tasks survive connection drops. The first is to issue your command with a [no-hangup command](https://en.wikipedia.org/wiki/Nohup). This will ensure your process survives connection drops but you won't be able to easily see command line logs and you also risk loosing any other session-level configuration you made before loosing connection (activating conda environments, navigating to paths, setting environment variables etc).

To ensure the entire session survives, the second option to issue long-running tasks is [screen](https://en.wikipedia.org/wiki/GNU_Screen). Screen keeps the shell session alive as long as either you close the session or the server gets shut down. Screen is pre-installed on Euler. The main commands are the following:

1. `screen -S <name-of-session>`: Start a new shell session.
2. `CTRL + A + D`: Leave the current session but keep it alive.
3. `screen -r <name-of-session>`: Reconnect to an existing shell session.
4. `screen -ls`: List all available shell sessions.

**No matter whether you choose the no-hangup command or screen, be aware of the following.** When you ssh into euler.ethz.ch, you'll get connected to one of ~30 different login nodes (allocation is based on load). Therefore, when you reconnect, you may _not_ end up on the login node where your screen or no-hangup command is running. To overcome this, you can login directly to always the same node (for example eu-login-20.euler.ethz.ch). This has the disadvantage that your chosen login node may be heavy loaded while others aren't and it may not be the preferred option by cluster support.

## Receiving notifications

When executing long-running workflows on Euler, you may want to get notified whenever workflows terminate. For individual jobs, you can use [built-in mechanisms](https://scicomp.ethz.ch/wiki/LSF_mini_reference). For entire workflows, you can send emails to yourself. This feature had been disabled with the [reopening of Euler in May 2020](https://scicomp.ethz.ch/wiki/Reopening_of_Euler_and_Leonhard_(May_2020)) but is available again as of July 2021. You can send an email without body like so:

```bash
mail -s 'mail subject' <receiver-email-address>"
```

Alternatively, and as long as email sending is not possible for whatever reason, you may want to use [IFTTT](https://ifttt.com/my_applets). IFTTT can send you mails, send push notifications to your phone, and do all sorts of much more weird things. The idea is quite simple: you trigger a webhook and that triggers a notification service. Using Python you can do the following to trigger webhooks:

```python
import requests

EVENT_NAME = "my_event"
API_KEY = "my_api_key"

requests.post(
    f'https://maker.ifttt.com/trigger/{EVENT_NAME}/with/key/{API_KEY}'
)

```

In addition, you need an IFTTT applet that listens to the webhook and reacts to it. [Here's a full example](https://pimylifeup.com/using-ifttt-with-the-raspberry-pi/) that shows how to build and trigger the applet.

## Check storage usage

There are upper limits of the number of files and total file size that we are not allowed to exceed. So far, the number of files seems to be more problematic which is caused to a large extent by anaconda. 


`lquota .` will give you an overview of the group-wide quotas in the current directory. 

`find . -type f | cut -d/ -f2 | sort | uniq -c` gives you an overview of the number of files in the subdirectories of the current directory. You can use it to understand how badly you contribute to the overall number of files in the group.  
