# Conda

1. Conda is XXXXXX

2. Install it from: https://conda.io/miniconda.html

3. Use mamba instead of conda: `conda install mamba -n base -c conda-forge`

The rest of this page is some mostly useful `conda` gotchas.

## Configuration management

Useful configuration variables:

- `channel_priority`: whether to stick to a channel when satisfying package dependencies. My (@Suvayu Ali) recommendation is to not mix channels, use `strict`. if there is an issue, it's one of:
    1. a misconfiguration on your end,
    2. a packaging bug, or
    3. a conda bug.

    If it is anything else, I owe you a drink.

- `pkgs_dirs`: this variable is useful to manage where the untared packages are stored.  This might be useful for shared installations, or simply when you are trying to separate your home/work directory from the conda directory hierarchy (possibly because of disk quotas, backup management, etc).  Both this variable and `envs_dirs` are lists.
- `envs_dirs`: this variable is useful to specify where your actual conda environment directory hierarchy tree is stored. If `conda` is doing its job right, then it's using hard links.  For that to work, both `pkgs_dirs` and `envs_dirs` should point to the same disk partition.

`conda` configuration for a user is stored in `~/.condarc`, however if you have any environment specific config, it will be stored inside `/path/to/env/.condarc`.  A `condarc` file is essentially a `yaml` file, so you could always hand edit it, but the preferable method is to use `conda config`.

- To set a value

    `$ conda config --set channel_priority strict`

- To add a value to the front of a list

    `$ conda config --add channels conda-forge`

- To add a value to the end of a list

    `$ conda config --append pkgs_dirs /opt/conda/pkgs`

## Managing environments

**Exporting/restoring to/from a file**

You may export an environment to a file, which can later be used to recreate it.  To export an environment listing all packages with their versions, you may use:

`$ conda env export --file /path/to/file.txt`

However, this will include many low level dependencies, and pins every dependency to a version, which might not always be the intention. So to export a simpler environment file, you may use:

`$ conda env export --file /path/to/file.txt --from-history`

Note, exporting your environment is not entirely portable.  It includes the `conda` environment path as an absolute path.  Which is specific for an install.  In case the file is used to "recreate" an environment under a different `conda` install, the path information should be edited out manually.

To recreate an environment from an exported file, you can do the following:

`$ conda create -n <name> --file /path/to/file [pkg_spec]`

The `pkg_spec` at the end of `conda create` maybe used to override packages in the file.

**Cloning and overriding**

Another useful variation is to use existing environments to create "derived" environments.  You may create a new environment by cloning an existing environment as its base.

`$ conda create --clone <existing> -n <new>`

This allows one to build on top of existing environments without changing the original (like a fork).

**Managing disk space**

With time, `conda` environments can get bloated in size as older unused packages are not cleaned.  You can use [`moult`]( https://github.com/suvayu/moult/) to clean up periodically.  Please read the README carefully before using.

## A typical `condarc`

```yaml
channels:
- conda-forge
  - defaults
channel_priority: strict
envs_dirs:
  - /opt/conda/envs
pkgs_dirs:
  - /opt/conda/pkgs
```

---

### Caveats

Although `conda` is supposed to use hard links to create directory hierarchies, I (@Suvayu Ali) have found that isn't always true.  I'm not sure exactly how to ensure that hard links are used optimally.
