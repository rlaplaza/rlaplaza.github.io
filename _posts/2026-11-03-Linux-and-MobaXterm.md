---
title: Linux and MobaXterm for bachelor students
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines, beginner
---

# Linux and MobaXterm for bachelor students

This page is the first stop if you have only ever used Windows. By the end you will be able to install MobaXterm or Ubuntu inside Windows, type basic Linux commands on your own computer, and open a remote session on the lab cluster, Agustina.

You do not need to memorize the commands. Type them once, read what comes back, and keep this page open.

## Why the lab uses Linux

The lab computers and Agustina run Linux. Linux is an operating system, like Windows, with a different way of talking to the machine.

Think of the computer as a building. Folders are rooms. Files are the things inside a room. On Windows you walk around by clicking. In Linux you mostly send short text messages. The window where you type those messages is the **terminal**. The program inside it that reads each line is the **shell**.

A **command** is one of those messages. You type it after the prompt (the text that already sits on the line, often ending in `$`) and press Enter. Do not type the `$` itself.

{% include alert.html type="info" content="On a Mac, skip the install below and open the app named Terminal. The commands on this page are the same. On Linux, open the app named Terminal and do the same." %}

## Install MobaXterm

MobaXterm is a free Windows program that gives you a Linux-style terminal and a way to connect to other computers.

1. Open the [MobaXterm download page](https://mobaxterm.mobatek.net/download.html).
2. Choose the **Home** edition. It is free.
3. Choose the **Installer** edition, not the portable zip.
4. Run the downloaded installer and accept the default options.
5. Open **MobaXterm** from the Start menu.

The first screen has a big button that starts a **local terminal**. "Local" means this terminal is running on your laptop, not on Agustina. Click **Start local terminal**.

You should see a line that ends with `$` and a blinking cursor. That cursor is where your typing goes.

Three parts of the window matter:

- The black (or dark) area is the terminal. You type commands there.
- **Session**, at the top, is how you phone another computer later.
- The left panel is a file browser. After you connect to Agustina it shows the remote files, and you can drag a file between your laptop and the cluster. You do not need it for the practice below.

Your Windows Desktop is not the first place this terminal opens. MobaXterm keeps its own home folder, often `/home/mobaxterm`. Stay there while you practice, so you do not change real Windows files.

## Install Ubuntu with WSL

MobaXterm is enough for the first commands and for logging into Agustina. **WSL** (Windows Subsystem for Linux) is a second option: a real Ubuntu Linux that lives inside Windows. Use it when a later guide asks you to install Linux programs on your laptop. The [molecular visualization](/2026/10/08/Molecular-visualization-with-v-vmol-xyzrender.html) guide is one of those.

{% include alert.html type="warning" content="The [Local Conda and VS Code Setup](/2026/10/05/Local-conda-and-vscode-setup.html) guide wants Windows PowerShell, not this Ubuntu window." %}

You need Windows 11, or a recent Windows 10. The computer will restart once.

1. Open the Start menu and type `PowerShell`.
2. Right-click **Windows PowerShell** and choose **Run as administrator**. Say yes if Windows asks for permission.
3. Type this and press Enter:

```powershell
wsl --install
```

4. Wait until it finishes. If it asks you to restart, restart the computer.
5. After the restart, the **Ubuntu** window should open by itself. If it does not, open **Ubuntu** from the Start menu.
6. Ubuntu asks for a new username and a new password. This account belongs to Ubuntu. It is not your Windows password.

{% include alert.html type="tip" content="The Ubuntu password stays invisible while you type it. Type it, press Enter, then type it again when Ubuntu asks you to confirm." %}

You should land at a line that ends with `$`. Check that this really is Ubuntu:

```bash
cat /etc/os-release
```

You should see `NAME="Ubuntu"` near the top.

Next time, open **Ubuntu** from the Start menu. The same commands in the rest of this page work here. Two addresses are different from MobaXterm:

- Your Ubuntu home is `/home/` plus the username you just chose, not `/home/mobaxterm`.
- Your Windows files live under `/mnt/c/Users/YourName/`. In MobaXterm that same place is `/drives/c/Users/YourName/`.

Bring Ubuntu's list of installable programs up to date before you install anything else. `sudo` means "run this as Ubuntu's administrator". It asks for the Ubuntu password.

```bash
sudo apt update
sudo apt upgrade
```

Press `Y` and Enter if `upgrade` asks whether to continue. This can take a few minutes. When the `$` prompt comes back, Ubuntu is ready.

If `wsl --install` says the command is not recognized, install the **Ubuntu** app from the Microsoft Store, open it, and create the username and password there. If Windows complains that virtualization is turned off, ask in the lab before changing BIOS settings.

## Your first commands

Type each block one line at a time. Press Enter after each line. Wait until the `$` prompt comes back before you type the next command.

```bash
pwd
```

`pwd` means "print working directory". A **directory** is a folder. This command tells you which folder you are standing in. In MobaXterm you should see a path such as `/home/mobaxterm`. In Ubuntu you should see `/home/` plus the username you created.

A **path** is the address of a file or folder. Slashes separate the rooms. `/home/mobaxterm` means: start at the root of the disk (`/`), go into `home`, then into `mobaxterm`.

```bash
ls
```

`ls` lists the names in the current folder. An empty answer is fine. It means the folder has nothing to show yet.

```bash
mkdir linux-practice
cd linux-practice
pwd
```

`mkdir` makes a new folder. `cd` changes into it ("change directory"). After `pwd`, the path should end in `linux-practice`.

Now make a tiny text file. `vi` is the editor you will use in the terminal. It starts in Normal mode, where the keys are commands, not letters on the page.

```bash
vi hello.txt
```

- Press `i` to enter Insert mode. The bottom of the window should say `-- INSERT --`.
- Type one sentence, for example `Hello from the terminal`.
- Press `Esc` to go back to Normal mode.
- Type `:wq` and press Enter. That writes the file and quits.

If you get stuck, press `Esc`, then type `:q!` and press Enter. That leaves without saving. The rest of the editor, including `vim`, is in [Vi and Vim Basics](/2026/01/22/Vi-and-vim-basics.html).

You are back at the `$` prompt. Check that the sentence was saved:

```bash
cat hello.txt
```

`cat` prints a whole file. You should see the sentence you typed.

```bash
cp hello.txt hello-copy.txt
mv hello-copy.txt renamed.txt
ls
```

`cp` copies a file. `mv` moves or renames a file. `ls` should now show `hello.txt` and `renamed.txt`.

{% include alert.html type="warning" content="`rm` deletes files immediately. There is no Recycle Bin." %}

Delete only the copy you just made:

```bash
rm renamed.txt
ls
```

`renamed.txt` is gone. `hello.txt` should still be there. Do not run `rm` on a name you did not create yourself.

Step back out of the practice folder. Two dots means "the folder above this one":

```bash
cd ..
pwd
```

You should be back in the folder you started from, and `linux-practice` is still there for next time.

## Three keys that save time

- **Tab** finishes a name. Type `cd linux-prac` and press Tab. The shell should complete `linux-practice`. If nothing happens, the name matches more than one file, or the spelling is off.
- **Up arrow** brings back the previous command. Press Enter to run it again.
- **Ctrl+C** cancels the line you are typing, or stops a command that is still running. Use this when you are stuck. It does not turn the computer off.

## Paths, in one picture

| You type | It means |
| --- | --- |
| `/` | The root. Every full address starts here. |
| `~` | Your home directory, a shortcut for "my files". |
| `..` | The folder one level up. |
| `.` | The folder you are in right now. |

Try this:

```bash
cd ~
pwd
ls -a
```

`cd ~` jumps home from anywhere. `ls -a` lists hidden names too. A hidden name starts with a dot. You will probably see `.bashrc`.

`.bashrc` is a settings file the shell reads each time it starts. Later guides ask you to add lines to it. Do not edit it today. Seeing the name is enough.

Windows files, if you need them, live under `/drives/c/Users/YourName/Desktop` in MobaXterm and under `/mnt/c/Users/YourName/Desktop` in Ubuntu. You can `cd` there. For this practice, stay in the Linux home.

## Phone another computer with SSH

**SSH** is a phone call to another computer. You type commands in the same kind of window, but they run on the far machine. The far machine for this lab is Agustina, a shared supercomputer. A shared front door called the **bridge** sits in front of it.

You need the username and password the lab gave you. If you do not have them yet, stop after the local practice and ask.

{% include alert.html type="tip" content="When Linux asks for a password, the cursor does not move and no dots appear. That is normal. Type the password and press Enter." %}

From Ubuntu, skip the clicks below and type `ssh username@bridge.bifi.unizar.es`, using your real username. The steps after the password are the same. From MobaXterm, use the mouse:

1. In MobaXterm, click **Session**, then **SSH**.
2. In **Remote host**, type `bridge.bifi.unizar.es`.
3. Tick **Specify username** and type your username.
4. Click **OK**.
5. If it asks whether you trust the host, type `yes` and press Enter. This happens the first time only.
6. Type your password and press Enter.

The prompt should change. You are on the bridge, not on your laptop. MobaXterm remembers this session in the left sidebar, so next time you double-click it instead of filling the form again.

From the bridge, call Agustina:

```bash
ssh username@agustina.bifi.unizar.es
```

Replace `username` with your real username. Type your password again when asked.

Check where you landed:

```bash
pwd
ls
```

`pwd` should show a path under `/home/` with your username. `ls` shows your files on the cluster, which may be empty.

The left panel should now list those remote files. Dragging a file between that panel and your computer is the easy way to move one file. You do not need another program for that.

Leave the same way you would hang up a phone. `exit` drops one connection:

```bash
exit
```

The first `exit` leaves Agustina and returns you to the bridge. A second `exit` leaves the bridge and returns you to the local terminal.

There is a shorter version of this double call. You do not need it today. The [Using Agustina](/2025/08/01/Using-agustina.html) page asks you to save these two nicknames, called **aliases**, inside `.bashrc` on your own computer:

```bash
alias tob='ssh -X username@bridge.bifi.unizar.es'
alias toa='ssh -J username@bridge.bifi.unizar.es username@agustina.bifi.unizar.es'
```

After that, `toa` opens Agustina in one step. Replace `username` before you paste them. That page also explains the cluster itself: storage, conda, and job scripts. This page stops at the door.

If the session waits a long time and then fails, you are probably off the network the cluster expects. Try again from the university network, then ask in the lab. Do not keep guessing passwords.

## What to read next

Stay in this order. Each page assumes you can already do what the previous one taught.

1. [Linux Sysadmin Basics](/2026/01/22/linux-sysadmin-basics.html) repeats these commands and adds `.bashrc`, `PATH`, and small scripts.
2. [Using Agustina](/2025/08/01/Using-agustina.html) is the cluster: the `tob` and `toa` aliases, conda, and how to submit a job.
3. [Git tips](/2025/07/30/Git-tips.html) can be done in this same terminal, whether that is MobaXterm or Ubuntu.
4. [Vi and Vim Basics](/2026/01/22/Vi-and-vim-basics.html) is the rest of the editor you just used: moving around, changing text, and comparing files.
5. [Coding tips](/2025/07/30/Coding-tips.html) points at the [Software Carpentry Unix shell](https://swcarpentry.github.io/shell-novice/) lessons, which are the right next practice once this page feels easy.
6. [Local Conda and VS Code Setup](/2026/10/05/Local-conda-and-vscode-setup.html) is for Python on your laptop, after the terminal feels ordinary.

If a command prints an error, copy the error, say which page you were on, and ask. That is faster than trying random flags.
