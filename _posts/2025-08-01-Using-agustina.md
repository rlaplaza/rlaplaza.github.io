---
title: Using Agustina
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines
---

# Using Agustina

Agustina is a High Performance Computing (HPC) center. There are many such centers in the world (a list of the most powerful supercomputers is available in the [Top500](https://top500.org/), in case you are curious). 

The main documentation of Agustina is [here](https://doc--publica-bifi-es.translate.goog/agustina/manual_agustina.html?_x_tr_sl=es&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp), which includes account creation and basic usage. Remember that any changes you do to your ```.bashrc``` will be applied once you create a new terminal or run ```source .bashrc```.

* Like most HPC centers, Agustina requires users to connect from a local network for security reasons. In other centers you must use a VPN. In Agustina, you can ssh to the login node (``agustina.bifi.unizar.es```) through a bridge node (```bridge.bifi.unizar.es```). Setting up these aliases in your ```.bashrc``` file will be useful:

```
alias tob='ssh -X username@bridge.bifi.unizar.es'
alias toa='ssh -J username@bridge.bifi.unizar.es username@agustina.bifi.unizar.es'
```

That way you can jump over bridge to access Agustina using simply ```toa```. i

* Once in Agustina, you will notice that the command line interface is very plain. To improve it, you can set up a few extras in your ```.bashrc``` file (in your Agustina user home!):

```
export PS1="\[\e[31m\][\[\e[m\]\[\e[33m\]\u\[\e[m\]@\[\e[36m\]\h\[\e[m\]\[\e[31m\]]\[\e[m\]:\[\e[33;40m\]\w\[\e[m\]\[\e[40m\] \[\e[m\]\[\e[32;40m\]\\$\[\e[m\] "
alias wqm="watch squeue --format=\'%.18i %.9P %.30j %.8u %.8T %.10M %.9l %.6D %R\' -u username"
```

* Next step is setting up github and git. To start, go to your [github keys](https://github.com/settings/keys) setting and then generate a new ssh key in Agustina and add it to your account following [the instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh). 

Once that is done, you can get the lab's utility scripts from the [group organization page](https://github.com/rlaplaza-lab/utility_scripts), assuming you have asked for permissions. Clone this directory to a directory (I put it in ```/home/username/Software/```) and add the directory to your path in ```.bashrc``` :

```
export PATH="/home/username/Software/utility_scripts:$PATH"
```

Now you can use the scripts from this directory in your command line. You can check usage by reading the scripts or, in general, using the help flag, e.g.:

```subconda.sh -h```

* The next step is setting up ```conda``` or some other package manager.

You can follow the instructions in the documentation [here](https://doc--publica-bifi-es.translate.goog/agustina/anaconda.html?_x_tr_sl=es&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp). 

As you can see, the space you have in your ```/home``` in Agustina is small, so your conda envs should be stored in ```/fs/agustina/username/conda-env/```. Since typing the full path to activate an env is a pain in the derriere, you can add that location to let conda figure out aliases.  

```
conda config --append envs_dirs /fs/agustina/username/conda-env/
```

You can now activate envs using their names directly.

---


