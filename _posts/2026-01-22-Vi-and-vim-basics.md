---
title: Vi and Vim Basics
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines, linux, bash, vim, vi
---

# Vi and Vim Basics

If you work on a Linux machine, a remote server, or an HPC system, you will eventually need to edit a configuration file, a shell script, or a short note in the terminal. `vi` and `vim` are classic tools for exactly that. They are usually installed on Unix-like systems, they work well over SSH, and they do not require a graphical interface.

This short guide is meant to be practical. It complements the [Linux Sysadmin Basics](/2026/01/22/linux-sysadmin-basics.html) article and the [Coding Practices Guidelines](/2025/07/30/coding-tips.html), especially the parts about shell work, remote systems, and command-line tooling.

## What are `vi` and `vim`?

`vi` is the traditional Unix text editor. `vim` ("Vi Improved") is the modern, more feature-rich version that is installed on many systems today. In practice, the commands are nearly the same, so learning one means you are already well on your way to using both.

Linux servers and cluster login nodes often provide only terminal editors, which is why `vim` remains useful even if you prefer VS Code or another GUI editor for local work.

## Opening, saving, and quitting

Open a file:

```bash
vim ~/.bashrc
```

You are usually in Normal mode when the editor starts. Normal mode is where you move around and run commands. Press `i` to enter Insert mode and type text. Press `Esc` to go back to Normal mode.

The most important commands are:

```vim
i        Enter Insert mode
Esc      Leave Insert mode
:w       Save the current file
:q       Quit
:wq      Save and quit
:q!      Quit without saving
```

A beginner habit worth keeping is: whenever you want to execute a normal-mode command, make sure you are not in Insert mode.

## Basic motion and editing

Use the arrow keys or the classic movement keys:

```vim
h j k l   Move left, down, up, and right
w        Move to the next word
b        Move to the previous word
0        Move to the beginning of the line
$        Move to the end of the line
G        Go to the end of the file
gg       Go to the beginning of the file
/word    Search forward for a word
n        Repeat the search forward
N        Repeat the search backward
```

Common editing commands:

```vim
x        Delete the character under the cursor
dd       Delete the current line
dw       Delete the current word
yy       Copy the current line
p        Paste after the cursor
P        Paste before the cursor
u        Undo
Ctrl-r   Redo
```

The editing workflow is simple:

```bash
vim notes.txt
```

Then:

1. Move to the right place with `h`, `j`, `k`, `l`, or the arrow keys.
2. Press `i` to insert text.
3. Type what you need.
4. Press `Esc`.
5. Use `:w` to save or `:wq` to save and quit.

## Visual mode and block editing

`vim` is especially handy for editing blocks of text. The main visual modes are:

```vim
v        Character-wise visual selection
V        Line-wise visual selection
Ctrl-v   Block-wise visual selection
```

Once you select a block, you can do things such as:

```vim
d        Delete the selection
y        Copy the selection
>        Shift the block right (indent)
<        Shift the block left (dedent)
```

This is useful when aligning configuration blocks, editing repeated patterns, or changing a series of similar lines. For example, you might indent several lines in a shell script or add a comment prefix to a set of configuration entries.

The block-selection mode is one of the features that makes `vim` feel surprisingly powerful once you get used to it.

## Search and replace

Search is often the fastest way to move through a file:

```vim
/PATH
n
```

This searches for `PATH` and jumps to the next match. To replace text throughout a file:

```vim
:%s/old/new/g
```

This replaces every occurrence of `old` with `new` in the whole file. If you want confirmation for each change, use:

```vim
:%s/old/new/gc
```

This is especially useful when editing config files or scripts where names or paths need to be updated consistently.

## `vimdiff` for comparing files

`vimdiff` is a very useful tool for reviewing changes between two versions of a file. For example:

```bash
vimdiff file1.txt file2.txt
```

You will see both files side by side, and `vimdiff` highlights the differences. Some useful commands are:

```vim
]c       Jump to the next difference
[c       Jump to the previous difference
do       Diff obtain: copy from the other file
dp       Diff put: copy to the other file
```

This is a great tool for checking changes in config files, scripts, or results from different runs. If you are debugging a parameter change or reconciling versions, it is much faster than manually diffing text in a shell.

## Editing configuration files

The configuration files you will edit most often are `~/.bashrc`, `~/.config/...`, or other user-level settings. A typical workflow is:

```bash
vim ~/.bashrc
```

Then add or adjust an alias, a `PATH` export, or a shell option. After saving the file, reload it in the current shell:

```bash
source ~/.bashrc
```

This is exactly the kind of task that `vim` is made for: quick edits on remote systems without needing a graphical editor.

## Good first habits

Start small. Learn only the commands you need most often:

- open and close a file
- move around
- insert text
- save
- search
- quit without saving when needed

Do not try to memorize every `vim` command at once. A few workflows become second nature quickly, and then the editor starts to feel much less intimidating.

## Useful resources

The best places to continue learning are:

* [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) has a great introduction to shell tools and terminal editors.
* [Vim's built-in help](https://vimhelp.org/) is the definitive reference.
* `vimtutor` is the best built-in primer for beginners. Run it with:

```bash
vimtutor
```

* The [Linux Sysadmin Basics](/2026/01/22/linux-sysadmin-basics.html) article covers shell setup, environment variables, and remote work habits that go well with `vim`.
* The [Using Agustina](/2025/08/01/using-agustina.html) guide is a practical example of editing shell configuration files in an HPC workflow.

Once you know the basics, `vim` becomes one of the most useful tools in a Linux workflow: it is fast, flexible, and available almost everywhere.

---
