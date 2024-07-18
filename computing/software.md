---
title: Software and Toolboxes
parent: Computing
---


# Neurodesk

`Neurodesk` is a flexible package for reproducible neuroimaging. It ships with all commonly used toolboxes, often in different versions, and allows for
an easy and rapid installation. The package is built upon the use of singularity containers which allows for a fixed software environment for each tool
facilitating reproducibility. It can be used on local computers, **with and without** graphical interface, as well as on Scinet where it integrates
well with the `lmod` system (i.e., the `module` command).

* [website](https://www.neurodesk.org/)
* [integration as shell commands using `lmod`](https://www.neurodesk.org/docs/getting-started/neurocommand/hpc/)

#### Installation

1. Google [`lmod documentation`](https://lmod.readthedocs.io/en/latest/#installing-lmod) and go to `Installing Lmod` > `Installing Lua and Lmod` > `Bash under Ubuntu`.
2. Add the code under the Lmod installation guide's `Bash under Ubuntu` section at the top of your `~/.bashrc` file.
3. Uncomment all `deb-src` lines in `/etc/apt/sources.list`. Run the code under `Using Your Package Manager for Ubuntu or Debian Systems`.
4. [Clone](#cloning-a-repository-on-github) the repository of [neurocommand.git](https://github.com/NeuroDesk/neurocommand) without the `export` commands. Add `module use /localdata/neurocommand/local/containers/modules/` to your `~/.bashrc` file, replacing `/localdata/` with wherever your cloned `Neurocommand`. Restart your terminal (`source ~/.bashrc`) and run `module avail` to see if the installation was successful.
5. Navigate to your `neurocommand` folder and run `bash containers.sh` to see how to use the command. `bash containers.sh <module_name>` will display module versions that can be downloaded and used. As of the time of this writing, we want `bidscoin/4.3.0`. Run `bash containers.sh bidscoin` to view available BIDScoin modules and run `./local/fetch_containers.sh bidscoin 4.3.0 20240221`.
6. Check that you successfully downloaded the `bidscoin/4.3.0` module using `module avail` and `module load bidscoin/4.3.0`. Run `module avail` once more to check that it has been loaded; loaded modules will display `(L)` beside them.

#### Notes on `neurocommands`

1. When using the integration as shell commands via the `neurocommands` package it might be necessary to pass environment variables to a launched command
  (e.g. Freesurfer's `SUBJECTS_DIR` variable when running `mri_sclimbic_seg`). Since the shell integration of `neurocommands` launches singularity commands
  in the background, the regular way of exporting bash variables via `export SUBJECTS_DIR=...` does not work. Instead, the variables that should be used by
  any of these `Neurodesk` commands need to be defined at runtime when via `SINGULARITYENV_SUBJECTS_DIR=... mri_sclimbic_seg ...`. In other words, any variable
  name needs to be prepended with the term `SINGULARITYENV_`.

2. In order to access data through `neurocommands`, the corresponding folders have to be "*made available*" to the underlying singularity command. This works
  by defining all necessary paths for reading and writing data in the environment variable `SINGULARITY_BINDPATH`. For example, if the command `fslmaths` reads
  data from `/localdata/input` and writes data to `/localdata/output` the variable should be set as `SINGULARITY_BINDPATH=/localdata/input,/localdata/output`.
  It would work equally well to set the variable as `SINGULARITY_BINDPATH=/localdata`. For common data locations (e.g. on a local computer) it is useful to
  define the bindpaths in the file `.bashrc` as `export SINGULARITY_BINDPATH=/localdata` so that future commands using data only in `/localdata` do not need
  another specification of the environment variable.

3. In case specific singularity flags need to be passed when launching a command from `neurocommands`, the environment variable `neurodesk_singularity_opts` can
  be used. For example, in order to launch a graphical tool like `mrview`, first define `export neurodesk_singularity_opts="--nv"` to activate the singularity
  GPU support.


#### Example workflow

After successful installation of the shell integration, a typical script could look like this:
```
# 1. activate freesurfer commands
module load freesurfer/7.4.1
module load fsl/6.0.5

# 2. define available data locations (everything in $SCRATCH and $HOME will be accessible,
#    e.g. $SUBJECTS_DIR if it is located in either of those locations)
export SINGULARITY_BINDPATH=$SCRATCH,$HOME

# 3. use Freesurfer and FSL commands
...
SINGULARITYENV_SUBJECTS_DIR=/path/to/subjects/dir mri_sclimbic -s sub-01 ...
...
fslmaths ...
...
```
