---
title: Using Agustina
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines
---

# Using Agustina

Agustina is a High Performance Computing (HPC) center. There are many such centers in the world (a list of the most powerful supercomputers is available in the [Top500](https://top500.org/), in case you are curious).

The main documentation of Agustina is [here](https://doc--publica-bifi-es.translate.goog/agustina/manual_agustina.html?_x_tr_sl=es&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp), which includes account creation and basic usage. Remember that any changes you do to your `.bashrc` will be applied once you create a new terminal or run `source .bashrc`.

* Like most HPC centers, Agustina requires users to connect from a local network for security reasons. In other centers you must use a VPN. In Agustina, you can ssh to the login node (`agustina.bifi.unizar.es`) through a bridge node (`bridge.bifi.unizar.es`). Setting up these aliases in your `.bashrc` file will be useful:

```
alias tob='ssh -X username@bridge.bifi.unizar.es'
alias toa='ssh -J username@bridge.bifi.unizar.es username@agustina.bifi.unizar.es'
```

That way you can jump over bridge to access Agustina using simply `toa`.

* For Windows users, the usual choice is PuTTY. The Agustina documentation describes the same login pattern: users can connect directly with `ssh username@agustina.bifi.unizar.es` if the firewall allows it, or use the bridge node with `ssh username@bridge.bifi.unizar.es` and then `ssh username@agustina.bifi.unizar.es` once you are inside the bridge. In PuTTY, this means:

  1. Open PuTTY and set the hostname to `username@bridge.bifi.unizar.es` if you are going through the bridge.
  2. Save the session with a memorable name and click `Open`.
  3. Log in with your Agustina username and password.
  4. Once you are on the bridge, run `ssh username@agustina.bifi.unizar.es` to reach the login node.

  If you are connecting directly and your IP is already authorized by the firewall, you can simply set the hostname to `username@agustina.bifi.unizar.es` instead. For file transfers, PuTTY's companion tools (`pscp` / `psftp`) or a graphical client like WinSCP are often the easiest alternatives to `scp` and `rsync`.

* Once in Agustina, you will notice that the command line interface is very plain. To improve it, you can set up a few extras in your `.bashrc` file (in your Agustina user home!):

```
export PS1="\[\e[31m\][\[\e[m\]\[\e[33m\]\u\[\e[m\]@\[\e[36m\]\h\[\e[m\]\[\e[31m\]]\[\e[m\]:\[\e[33;40m\]\w\[\e[m\]\[\e[40m\] \[\e[m\]\[\e[32;40m\]\\$\[\e[m\] "
alias wqm="watch squeue --format=\'%.18i %.9P %.30j %.8u %.8T %.10M %.9l %.6D %R\' -u username"
```

* Next step is setting up github and git. To start, go to your [github keys](https://github.com/settings/keys) setting and then generate a new ssh key in Agustina and add it to your account following [the instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh).

Once that is done, you can get the lab's utility scripts from the [group organization page](https://github.com/rlaplaza-lab/utility_scripts), assuming you have asked for permissions. Clone this directory to a directory (I put it in `/home/username/Software/`) and add the directory to your path in `.bashrc`:

```
export PATH="/home/username/Software/utility_scripts:$PATH"
```

Now you can use the scripts from this directory in your command line. You can check usage by reading the scripts or, in general, using the help flag, e.g.:

```subconda.sh -h```

* The group also provides a small set of job-submission helpers for Agustina in the `utility_scripts` repository. The relevant files live under `utility_scripts/agustina/`, and the two most useful ones are `suborca.sh` and `subconda.sh`. In practice, I clone the repository to a stable location, add the `agustina/` folder to `PATH`, and then use the wrappers from anywhere:

```
git clone https://github.com/rlaplaza-lab/utility_scripts.git ~/utility_scripts
printf '\nexport PATH="$HOME/utility_scripts/agustina:$PATH"\n' >> ~/.bashrc
source ~/.bashrc
```

After that, you can inspect the built-in usage text:

```
suborca.sh -h
subconda.sh -h
```

The idea is simple: the wrapper creates a `sbatch` job script for you, sets the job resources, activates the chosen environment, runs the job, and writes the logs under a `slurm_logs/` folder next to your input files. This saves a lot of boilerplate and reduces the chance of forgetting the `--account` project or a required environment module.

`suborca.sh` is meant for ORCA jobs. A typical example is:

```
suborca.sh -n 4 -m 16000 my_calculation.inp
suborca.sh -n 8 -m 32000 -p agustina_thin my_calculation.inp
```

This will submit an ORCA input file, defaulting to the resources described in the input (`%PAL nprocs`, `%MaxCore`) whenever possible. If your input file does not include those directives, you should set them explicitly in the `.inp` file or use the `-n` and `-m` flags carefully. The script also has a `-d` dry-run option, which is useful to inspect the generated SLURM script before submitting it. One thing to keep in mind: the script uses a default project account value (currently `molcat` in the local copy), so if your project is different, edit that variable near the top of the script before submitting.

`subconda.sh` is the equivalent helper for Python jobs that should run inside a specific Conda environment. Typical usage is:

```
subconda.sh -e my_env -n 4 -m 16000 my_script.py
subconda.sh -e my_env -n 8 -m 32000 -p agustina_thin my_script.py arg1 arg2
```

The script loads `anaconda/2025`, activates the requested environment, sets a good default for `OMP_NUM_THREADS` and related variables, and runs the Python job in a temporary scratch directory by default. This is especially handy when your project writes many files, because it keeps the job isolated and then copies the results back to the submission directory. If you want to stay in place instead of using scratch, add `-s`.

The job submission scripts are a convenience layer around `sbatch`, not a replacement for understanding SLURM. The key ideas are still the same: choose the correct project account, request enough memory and CPUs, and keep a close eye on your `slurm_logs` outputs when debugging a failed run.

* The next step is setting up `conda` or some other package manager.

You can follow the instructions in the documentation [here](https://doc--publica-bifi-es.translate.goog/agustina/anaconda.html?_x_tr_sl=es&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp).

Personally, I added these to my `.bashrc` as well:

```
module load anaconda/2025 > /dev/null # This avoids errors with scp and rsync with host jumping
export CONDA_PKGS_DIRS=/fs/agustina/$(whoami)/.conda-pkgs
mkdir -p /fs/agustina/$(whoami)/conda-env
# To activate or create envs, we now need --prefix=/fs/agustina/$(whoami)/conda-env/my-conda-env-name
# ... other lines you may need
export TERMINFO=/usr/share/terminfo # This makes sure that clear command works
```

The space you have in your `/home` in Agustina is small, so your conda envs should be stored in `/fs/agustina/username/conda-env/` (some packages are very big!). Since typing the full path to activate an env is a pain in the derriere, you can add that location to let conda figure out aliases.

```
conda config --append envs_dirs /fs/agustina/username/conda-env/
```

You can now activate envs using their names directly.

---


