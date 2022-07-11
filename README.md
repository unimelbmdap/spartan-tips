# Spartan tips and tricks

> This is a living document with tips and tricks for Spartan. Many of the tips will be applicable to other HPC systems with Slurm schedulers (with minor tweaking).

## The queue

### Get a nice list of your queued and running jobs

``` sh
squeue -u $USERNAME -Stime -o "%.18i %.9P %.20j %.8u %.2t %.10M %.6D %R"
```

### Get a rough idea when when a job would start

``` sh
sbatch --test-only -n 16 -N 1 -p physical -t 01:00:00 --wrap "hostname"
```

Here `-p` stands for "partition". Swap it for "bigmem", "gpgpu" as necessary.

Note that the result is just an **estimate**. Job sumbissions from other users between now and the start time may mean your job gets pushed down the queue! For average wait times you can also check out the [Spartan weather report](https://dashboard.hpc.unimelb.edu.au/status_specs/#wait-time).


### Highlight your jobs in the queue (so you can find them!)

``` sh
showq | grep --color=always -e "$USERNAME\|$"
```

### Check if a job crashed because it ran out of memory

``` sh
sacct -j <JOBID> --format MaxRSS,ReqMem --units G
```

If MaxRSS >~ ReqMem then there is a good chance you ran out of memory.

### Get a list of past jobs

Find all jobs that you ran that started after 2022-06-10 and ended before 2022-06-17:

```sh
sacct -S 2022-06-10 -E 2022-06-17
```

See `man --pager='less -p end_time' sacct` for the possible time formats that can be used.

## Modules

### Collections

You can save collections of commonly used modules and set the default modules which get loaded when you log in.

From the `module --help` output:

``` txt
Handling a collection of modules:
--------------------------------
  save | s                          Save the current list of modules to a user defined "default" collection.
  save | s            name          Save the current list of modules to "name" collection.
  reset                             The same as "restore system"
  restore | r                       Restore modules from the user's "default" or system default.
  restore | r         name          Restore modules from "name" collection.
  restore             system        Restore module state to system defaults.
  savelist                          List of saved collections.
  describe | mcc      name          Describe the contents of a module collection.
  disable             name          Disable (i.e. remove) a collection.
```

So to set your default modules you can do something like this:

```
module purge
module load gcc/10.2.0 git/2.28.0-nodocs
module save
```

> **⚠️ Warning**  
> This will also affect your job submission scripts too!


## Python

### Poetry

The `python/3.8.6` module has poetry pre-installed and is the easiest and safest way to mix and match poetry with other python packaging options.

### Anaconda / Miniconda / Mambaforge / Miniforge / Snakes on a plane

There are **a lot** of different variations of Conda. Spartan has both [miniconda](https://docs.conda.io/en/latest/miniconda.html) and [anaconda](https://www.anaconda.com/products/distribution) available as modules. However, I highly recommend [mambaforge](https://github.com/conda-forge/miniforge#mambaforge). It is relatively lightweight and uses the [mamba](https://github.com/mamba-org/mamba) package manager, which is much faster than `conda` and almost 100% compatible.

> **✏️ Note**  
> As of 7 Jul 2022, there is a pending ticket requesting that mambaforge be installed as a module on Spartan.
> The below info remains the recommended installation method until this ticket is resolved.

To use on Spartan it's easiest to just install it yourself by downloading the appropraite shell script from [here](https://github.com/conda-forge/miniforge#mambaforge) (Linux x86_64 (amd64)) and following the installation instructions.

#### A few tips for running `mambaforge` or `miniconda` on Spartan...

Create a directory in one of your project spaces to hold your conda environments. There is only limited space available in your home directory on Spartan (50 GB) and it will fill up fast if you store your environments there. Once you've done that, I recommend using the following `.condarc` file in your home directory to set up `mamba` / `conda` with minimum headaches:

```yaml
envs_dirs:
    - <YOUR CHOSEN ENVIRONMENT LOCATION HERE; eg. /data/gpfs/projects/punim0000/smutch/conda_envs>
auto_activate_base: false  # This will prevent the `default` environment from being activated upon login and make sure you are only using conda/mamba when you actually want to
channels:  # This list isn't necessary if you are using `mambaforge`, but it doesn't hurt...
  - conda-forge
  - defaults
channel_priority: strict  # Always worth using for reproducability. Libraries like Snakemake will complain if you don't have this set.
```

Note that the location of this rc file is `$HOME/.condarc` regardless of if you are using `conda` or `mamba`!
