# HOWTO: Basic use of Calliope on the `euler` cluster

## Installation

To install the latest version of [Calliope](https://www.callio.pe/), you need to run three commands in order:

```bash
module load python/3.7.1
export PATH=~/.local/bin:$PATH
pip install --user --upgrade calliope
````

To install a specific version, for example ``0.6.3``, change ``calliope`` to ``calliope==0.6.3`` in the third command above.

Afterwards, confirm that Calliope is installed and works by typing:

```bash
calliope --version
```

This should simply print the installed version, such as "`Version 0.6.3`", and show no errors.

If the version shown does not match the version you installed, make sure that you ran all three of the commands above.

To automatically activate the correct Python version and make Calliope accessible each time you log in to the cluster, you should edit the file ``.bash_profile`` in your home directory.

An easy way to achieve that is with the `nano` editor by typing:

```bash
nano ~/.bash_profile
```

Add the following two lines to the very end of the file:

```
module load python/3.7.1
export PATH=~/.local/bin:$PATH
```

To solve models, you should use the Gurobi solver, which requires loading it:

```
module load new gurobi/8.1.1
```

It is easiest to add this line to `~/.bash_profile` as well to ensure it is always loaded on login.

## Running

At its most basic, a Calliope model can be submitted like this:

```
bsub -W 06:00 -n 1 -R "rusage[mem=10G]" -o calliope.log calliope run model.yaml --save_netcdf=result.nc
```
This assumes the command is run from an environment where calliope is available, and from a path that contains the model `model.yaml`. It requests a run time of 6 hours, a single CPU, and 10 GB of RAM. `-o calliope.log` means that all output (standard out and standard error) is logged into calliope.log (appending if the file exists).

Calliope also comes with a tool to automate the generation and submission of many models at once, which is [described in the documentation](https://calliope.readthedocs.io/en/stable/user/advanced_features.html#generating-scripts-to-run-a-model-many-times).
