# HOWTO: Basic use of the Sector-Coupled Euro-Calliope model

## Premise
This short wiki is intended for users that already have a basic knowledge of the Calliope modelling framework. If you don't, please have a look at [Calliope's documentation](https://calliope.readthedocs.io/en/stable/index.html) first. It is especially advised to have at least tried to run "[Tutorial #1](https://calliope.readthedocs.io/en/stable/user/tutorials.html)" successfully.

In particular, you must already have installed the [Anaconda Python distribution](https://www.anaconda.com/products/distribution).

In fact, the Sector-Coupled Euro-Calliope model is nothing but a set of text (.yaml) and .csv files that can be interpreted and run within a Calliope environment. No different from the tutorials suggested above, only much larger and more detailed.

## Download and setup

Before continuing, if you are on Windows, download and run the installer for the [git version control system]([https://git-scm.com/](https://git-scm.com/download/win)), which is not available on Windows by default.

In addition, to speed up the various installation steps below, we will install "mamba", a faster version of the "conda" package manager for the Anaconda distribution. To do so, run this command in a terminal window:

`conda install -c conda-forge mamba`

There are various ways to download the Sector-Coupled Euro-Calliope model and set up a suitable Calliope environment for using and running it. 

1. You can work with the version of the model files used in the [Joule publication](https://doi.org/10.1016/j.joule.2022.05.009) that marked the first peer-reviewed use of the model. We provide an [updated version of that model for you to download here](https://edu.nl/epupe). Download the zip file and unzip it on your computer (note that this is a slightly different version than the one stored on Zenodo as associated with the publication as "[Pre-built Sector-coupled Euro-Calliope Model](https://zenodo.org/record/5774988#.Y8eyVv7MKUl)"). After unzipping, also unzip the `2050.zip` file inside the unzipped folder (you can leave aside 2030 for now, as we will focus on the 2050 version). This is the suggested option if you do not require radical changes to the model, such as a complete revisitation of the model nodes and grid topology.

2. You can also download a customised version of the above pre-built model that subsets the problem to the North-Sea region with a spatial resolution of one node per country only. The latter is accessible [here](https://surfdrive.surf.nl/files/index.php/s/mZE64jCJamytBZt) and is meant for students that do not have access to enough computational power to run the full model.

3. In principle, it is possible to download, install and run the whole workflow that *generates* the very model files for complete customisation and adaptation. The workflow is based on Snakemake and is available on [GitHub](https://github.com/calliope-project/sector-coupled-euro-calliope). However, we are still in the process of cleaning and perfecting the workflow towards user-friendliness. It is not guaranteed that it will work seamlessy on any machine. This is the ***least recommended*** option for non-expert users.

Having obtained the model files by one of the above methods, you need to setup a dedicated Calliope environment for reading and running those. As also explained in the `README.md` file that accompanies the model files, you can do so by: 

- activating your Anaconda distribution (e.g., opening a terminal and typing `conda activate`)
- moving to the directory where the model files are stored (e.g., `../2022-02-08/` for the case of the Zenodo release)
- and typing: `mamba env create -f requirements.yml`

This will create a new Anaconda environment based on the requirements specified in the .yml file that accompanies the pre-built model. The name of the environment is given in the same file. By default, it is `eurocalliope_[version]`. For example, `eurocalliope_2022_02_08`.

Activate the environment from your anaconda terminal by typing: `conda activate eurocalliope_2022_02_08` (or your environment name, if different). You are all set!


## Understanding the model structure

### Projection year
The model directory is divided into two main sub-directories based on the model projection year: 2030 or 2050. The difference between the two lies in technology cost projections and which technologies and demand types are considered. Some key aspects to keep in mind:

- Technology costs in 2050 are, by default, cost projections for 2050. The same applies to 2030. This implies that the technologies are deployed very close to the optimisation horizon. If a more conservative approach is desired, one might run 2050 based on 2030 cost overrides (e.g., those in the folder `/2022-02-08_custom/2050/model/overrides-2030`). But no alternative cost assumption is available for 2030 itself)
- Emissions in 2030 do not include industrial feedstock emissions. This is because the demand for methanol itself (`demand_industry_methanol`, the fuel that synthesises all industrial feedstock in the model) is removed from the model.

### Spatial resolution
Within each projection-year directory, there are: a `model` folder and two folders named after the possible options for the spatial resolution. By default, `eurospores` and `national`. 
1. `eurospores` is the model resolution used in the Joule paper cited above and corresponding to a grid topology of 98 nodes across all the European countries covered by the model. Such a grid topology is based on the e-Higway project and represents a "realistic" approximation of the key existing transmission bottlenecks;
2. `national` is the resolution in which the sub-national nodes existing in the `eurospores` topology are merged, leading to a one-node-per-country topology.

Each of these two folders only contain a file with the absolute yearly values for all energy demand types and for all years between 2000 and 2018, which are used for scaling the non-dimensional hourly demand profiles.

The `model` folder is the key one. Within the latter, there is a series of .yaml file defining model-wide technology parameters and constraints that do not depend on the chosen model resolution. For example, the file `storage-techs.yaml` defines all the storage technologies in the model (except heat-storage ones, which are in the heat-specific file) with their costs, lifetime, efficiencies, etc. 

In addition, the `model` folder includes two sub-folders that distinguish again between `national` and `eurospores` resolutions (in the North-Sea version of the model described above, the `eurospores` directory has been removed). Each folder contains the same set of files: 
1. additional .yaml files, which define location-specific technology parameters (e.g., the `cap_max` for a given technology in each location) or constraints (e.g., the maximum available supply of biofuels in any given location).\;
2. and .csv files containing hourly timeseries of demand, capacity factors or other parameters.

It is worth noting that most of the .yaml files in this sub-folders exist in multiple copies, named after the corresponding weather year. This occurs whenever weathers-specific parameters exist. The .csv files, instead, include all the possible weather years within themselves.

### Overrides and scenarios
It is worth noting that the .yaml files discussed above are separated by energy sector as much as possible. For instance, heat-specific or mobility-specific technologies and constraints are defined in distinct files, and they are defined as `overrides` to the basic model setup. This means that, to a certain extent, it is possible to apply only some of these `overrides` and, in such a way, customise the model scope.

A typical list of `overrides` to include all of the model features and sectors, shall be like the following: 
```python
"industry_fuel,transport,heat,config_overrides,gas_storage,link_cap_dynamic,freeze-hydro-capacities,add-biofuel,synfuel_transmission,res_3h"
``` 
The meaning of these overrides is reported in the `README.md` file. We repeat it here for convenience:

* `industry_fuel`: Includes all non-electrical industry demands distributed by region, and the necessary technologies to generate those fuels synthetically. This includes, e.g., annual methanol requirements for the chemical industry.

* `transport`: This ensures ICE and EV light and heavy vehicle technologies and annual demands are in the model. It also includes reference to constraints required to make smart-charging of EVs work (e.g. weekly demand requirements).

* `heat`: This ensures that all heat provision technologies and hourly demands are in the model. Carriers added are `heat` (space heating and hot water) and `cooking`. Technologies added can be found in `heat-techs.yaml`.

* `config_overrides`: This includes high-level simplifications, such as removal of technologies that are considered redundant (e.g., less interesting combined heat and power technologies).

* `res_3h`: Sets the model with a 3h resolution. Can be omitted or can be one of `res_2h`, `res_3h`, `res_6h`, `res_12h`. If you want to create more options (e.g., `res_24h`), open the `model.yaml` corresponding to the weather year of your interest. At the bottom of the file, you'll see the above resolution overrides. Simply copy and paste one of those modifying the resolution as needed. You will be then able to call the override when building the model as it occurs for the dafault ones listed here.

* `gas_storage`: Includes underground methane storage facilities, based on latest data on a national level. Can be omitted to remove the option of this technology.

* `link_cap_dynamic`: Sets a limit on transmission line capacities. The limits are chosen subjectively based on current capacity, such that lines with smaller current capacities can proportionally increase much more (e.g., 100x) than larger lines (e.g. 2x). See `national/links.yaml` for other override options to apply here.

* `freeze-hydro-capacities`: Sets hydro capacities to equal "today's" capacities. This seems more reasonable than setting current capacities as upper limits, as this causes the model to install no hydro.

* `add-biofuel`: Enables a biofuel supply stream with a distinct `biofuel` carrier, with annual limits on biofuel that can be provided (based on JRC residuals). This differs from the default inherited from the original, non-sector-coupled version of Euro-Calliope, which is a black box technology converting biofuel to electricity directly.

* `synfuel_transmission`: Enables methane and diesel to be distributed around all model regions, without losses. This can be useful to ensure that if diesel/methane is produced in region A for transport/heating, it can be consumed in region B.

## Running and analysing the model
In principle, the model can be interpreted by Calliope and run in the same fashion as the tutorials provided in the Calliope documentation. However, the Sector-Coupled Euro-Calliope model includes a few custom constraints that are not part of the basic Calliope code and that, therefore, cannot be seamlessly interpreted. The reason for these constraints not being part of the Calliope framework by default is that they mostly make sense within the design of the Sector-Coupled Euro-Calliope model only, and are not necessarily applicable to other Calliope models.

What this means in practice, is that the `model creation` step will fail to recognise some of the constraints defined in the .yaml files, and that is fine. In fact, the `model run` step will include ad-hoc code to recover all these "custom constraints" and add them on top of the previously created model.This two-step approach is reflected in the two Python scripts found in the top-level directory: `create_input.py` and `run.py`, which need to be run in sequence.

There are different ways to run such scripts and analyse the model results. The two main approaches are the following:
1. Running from the command line. See the `README.md` file for an example of how to do so.
2. Running from an additional, customised master Python script.

Below is an example of how to implement the second option:

- create a new, empty Python script in the top-level model folder. For instance, `my_model_run.py`.
- import the other two Python scripts and calliope: 
```python
import create_input
import run
import calliope
```
- point to the `model.yaml` of your interest (depending on the chosen weather year), define which Calliope overrides/scenarios you want to use and build the model:

```python
path_to_model_yaml = '2050/model/eurospores/model-2016.yaml' 
scenarios_string = "industry_fuel,transport,heat,config_overrides,gas_storage,link_cap_dynamic,freeze-hydro-capacities,add-biofuel,synfuel_transmission,res_3h"
path_to_netcdf_of_model_inputs = '2050/eurospores/inputs.nc' # indicates where to store and how to name the Calliope model object to be then picked up by the "run.py" script

model_input = create_input.build_model(path_to_model_yaml, scenarios_string_onlypt, path_to_netcdf_of_model_inputs)
```
- Specify a path to store the results. In the example below, we create beforehand a `results` path in the top-level directory. Then, run:

```python
path_to_netcdf_of_results = 'results/test_results.nc'
run.run_model(path_to_netcdf_of_model_inputs, path_to_netcdf_of_results)
```
- Inspect and analyse the model results. You can do so asynchronously simply reading the results file generated above. Below is a simple exaple where we analyse the installed technology capacities:

```python
model = calliope.read_netcdf('results/test_results.nc')
caps = model.get_formatted_array('energy_cap').sum('locs').to_pandas()
```
