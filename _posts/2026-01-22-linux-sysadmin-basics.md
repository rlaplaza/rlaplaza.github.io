---

title: Linux Sysadmin Basics
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines, linux, bash, sysadmin
---

# Linux Sysadmin Basics

If you work in research or software, you will probably spend time on a Linux machine sooner or later. You may use a local terminal, connect to a department server over SSH, or submit work to an HPC system. The commands are not difficult, but it helps to understand what your shell is doing before you start changing files or launching jobs.

This is a short introduction to the pieces you will use every day. For a broader learning path, see the [Coding Practices Guidelines](/2025/07/30/coding-tips.html), especially its sections on Unix shell commands, remote cluster work, and reproducible projects.

## The shell and your home directory

When you open a terminal, you are usually running a shell such as Bash. The shell reads your commands, expands variables, starts programs, and keeps track of your current directory.

`$HOME` is an environment variable containing the path to your home directory. It is normally the place where your personal files and configuration live. The `$` means "expand this variable"; the directory itself is usually written as `HOME` without the dollar sign.

```bash
echo "$HOME"
pwd
cd "$HOME"
pwd
```

`pwd` prints the working directory. `cd` changes it, and `cd` without an argument returns you to `$HOME`. The tilde is a shorter spelling of your home directory:

```bash
cd ~
cd ~/projects
```

Do not confuse `$HOME` with the directory where a program happens to be installed. Your home directory belongs to your user account, while system software may live under locations such as `/usr/bin` or `/opt`.

## What is `.bashrc`?

`.bashrc` is a hidden configuration file in your home directory. Bash reads it when it starts an interactive non-login shell. It is commonly used for aliases, the command prompt, shell options, and additions to your `PATH`.

You can inspect the file without editing it:

```bash
less ~/.bashrc
```

An alias gives a short name to a command you use often. For example:

```bash
alias ll='ls -lah'
```

To add this permanently, put the line in `~/.bashrc`. After saving the file, load it into the current shell:

```bash
source ~/.bashrc
```

The [Using Agustina](/2025/08/01/using-agustina.html) guide shows practical examples of `.bashrc` aliases, SSH jump hosts, prompt customization, and `PATH` settings for an HPC system. It is a useful next step once the basic idea is familiar. For editing files like this directly from the terminal, a classic option is `vim`; see the [Vi and Vim Basics](/2026/01/22/vi-and-vim-basics.html) guide for the essentials: modes, editing, visual blocks, and `vimdiff`.

Be careful when modifying `.bashrc`. A syntax error can make every new shell print an error, and a badly constructed `PATH` can make commands difficult to find. Before making a substantial change, keep a backup:

```bash
cp ~/.bashrc ~/.bashrc.backup
```

### The `PATH` variable

`PATH` is a colon-separated list of directories where the shell looks for executable commands. This command shows its current value:

```bash
printf '%s\n' "$PATH"
```

If you keep your own scripts in `~/bin`, you can add that directory while preserving the existing path:

```bash
mkdir -p "$HOME/bin"
export PATH="$HOME/bin:$PATH"
```

Putting the `export` line in `.bashrc` makes the change apply to future interactive Bash shells. Avoid replacing `PATH` with a single directory: the existing value is what lets the shell find standard commands.

## Executing Bash scripts

A Bash script is a text file containing commands that the shell can run in sequence. The first line is called a **shebang**. It tells the operating system which interpreter should run the file:

```bash
#!/usr/bin/env bash

echo "Hello from Bash"
printf 'Working directory: %s\n' "$PWD"
```

Save this as `hello.sh`. You can run it by passing the file to Bash directly:

```bash
bash hello.sh
```

You can also make the file executable and run it by its path. The `u+x` mode adds execute permission for the file's owner:

```bash
chmod u+x hello.sh
./hello.sh
```

The `./` matters: it means "run the file in the current directory." Linux usually does not search the current directory automatically when looking through `PATH`. This avoids accidentally running a file in the current directory when you intended to run a system command with the same name.

Use `ls -l hello.sh` to inspect its permissions. If you receive a "Permission denied" error, check that the file is executable and that you are running the intended file. `bash hello.sh` does not require the executable bit because Bash is being asked to read the file explicitly.

## Basic commands

These commands cover much of the daily work of navigating and inspecting a Linux system:

| Command | Purpose |
| --- | --- |
| `ls` | List files and directories |
| `cd` | Change directory |
| `pwd` | Print the current directory |
| `mkdir` | Create a directory |
| `cp` | Copy a file or directory |
| `mv` | Move or rename a file |
| `rm` | Remove a file or directory |
| `less` | Read a text file one screen at a time |
| `man` | Read a command's manual page |

A small example:

```bash
mkdir -p ~/practice/notes
cd ~/practice
printf '%s\n' 'first note' > notes/example.txt
less notes/example.txt
cp notes/example.txt notes/example-copy.txt
mv notes/example-copy.txt notes/renamed-note.txt
ls -l notes
```

The `>` operator writes output to a file and replaces that file if it already exists. Use `>>` to append instead. Quoting paths is a good habit because it also works when a path contains spaces:

```bash
less "$HOME/path with spaces/notes.txt"
```

Read commands before running them, especially commands involving `rm`. Removing a file is not the same as moving it to a recycle bin, and `rm -r` can remove entire directory trees. When you are unsure what a command does, start with its manual page:

```bash
man cp
man rm
```

You can also ask many commands for a short usage summary:

```bash
cp --help
```

When a file needs to be edited directly from the shell instead of a GUI editor, `vim` is the standard choice on Linux systems. The [Vi and Vim Basics](/2026/01/22/vi-and-vim-basics.html) guide is a quick reference for the most common commands and workflows.

The [Basic Git and GitHub Guide](/2025/07/30/git-tips.html) builds on these same terminal habits with commands for cloning repositories, checking changes, creating commits, and collaborating through GitHub.

## Looking at running processes with `top`

`top` is a terminal program that gives you a changing view of running processes and system resources. Start it with:

```bash
top
```

Useful keys while `top` is running include:

| Key | Action |
| --- | --- |
| `q` | Quit `top` |
| `P` | Sort by CPU usage |
| `M` | Sort by memory usage |
| `1` | Show individual CPU statistics |

At first, use `top` for observation. Look for processes consuming an unusual amount of CPU or memory, and notice the load and memory summaries at the top of the display. Do not terminate a process just because it appears in the list: first identify which user started it, what command it represents, and whether it belongs to an important job.

On a shared server or HPC system, your institution may provide additional monitoring commands or a scheduler such as SLURM. The [Using Agustina](/2025/08/01/using-agustina.html) post describes job helpers and log locations for that environment. For another example of useful monitoring habits, the [Gaussian Tips and Troubleshooting Guide](/2025/01/22/gaussian-tips-and-troubleshooting.html) discusses reading complete output, following logs with `tail -f`, and checking available disk space.

## Good first habits

When working on a machine you do not fully control:

1. Know where you are with `pwd` before creating or moving files.
2. Use `$HOME` and other variables instead of copying long paths by hand.
3. Read help or manual pages before using unfamiliar options.
4. Keep code, input files, and results organized in predictable directories.
5. Check logs and error messages before repeating a failed command.
6. Keep important work backed up and under version control.
7. Record the commands, software versions, and environment needed to reproduce a result.

The [General Guidelines for Master's and PhD Students](/2025/07/29/general-guidelines.html) expands on documentation, backups, version control, and remote collaboration. These habits are as important as memorizing individual commands.

The [Vi and Vim Basics](/2026/01/22/vi-and-vim-basics.html) guide is the practical companion for the quick editing commands you will use every time you change a shell config file or small script on a remote system.

## Further reading

Once these basics feel comfortable, the following sources provide more detail:

* [Bash startup files](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html) explains when Bash reads files such as `.bashrc`.
* The [GNU Coreutils manual](https://www.gnu.org/software/coreutils/manual/coreutils.html) documents many everyday file and text commands.
* [The Linux man-pages project](https://www.kernel.org/doc/man-pages/) provides detailed manual-page documentation.
* The [procps-ng project](https://gitlab.com/procps-ng/procps) provides tools including `top` and `ps`.
* [Vi and Vim Basics](/2026/01/22/vi-and-vim-basics.html) explains the core editor commands, visual block editing, and `vimdiff` for quick terminal-based comparisons.
* [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) has excellent lessons on the shell, command-line tools, editors, and remote work.

---