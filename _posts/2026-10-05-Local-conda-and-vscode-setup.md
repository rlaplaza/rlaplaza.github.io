---
title: Local Conda and VS Code Setup
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines, python, conda, vscode
---

# Local Conda and VS Code Setup

This guide walks you through installing Miniconda and Visual Studio Code on your own computer, creating a dedicated Conda environment, installing packages inside it, and running Python from that environment in VS Code.

By the end you should be able to:

1. Open a terminal and confirm that `conda` works.
2. Create and activate a named Conda environment.
3. Install packages into that environment only (not into the whole computer).
4. Select that environment as the Python interpreter in VS Code and run a small script.

If terminal commands feel unfamiliar, skim the [Linux Sysadmin Basics](/2026/01/22/linux-sysadmin-basics.html) post first. For broader coding habits and why environments matter, see the [Coding Practices Guidelines](/2025/07/30/Coding-tips.html). Once you need Conda on the cluster rather than on your laptop, continue with [Using Agustina](/2025/08/01/Using-agustina.html).

## 1. Open a terminal

Use the native terminal for your operating system. Run the commands one line at a time, and wait for each one to finish before starting the next.

### Windows

**Very important:** these Windows steps are for native Windows PowerShell. Do **not** use a WSL (Ubuntu) terminal for this guide.

1. Open the Windows Search bar.
2. Search for `Windows PowerShell`.
3. Open it.

Continue with the Windows instructions in section 2.

### macOS

1. Open Spotlight search (or look under Applications → Utilities).
2. Type `Terminal` and open it.

Continue with the macOS instructions in section 2.

### Linux

**Very important:** these Linux steps are for a normal Linux machine. Do **not** use a WSL (Ubuntu) terminal inside Windows for this guide.

1. Open Activities (or press `Ctrl + Alt + T` on many distributions).
2. Search for `Terminal` and open it.

Continue with the Linux instructions in section 2.

## 2. Install Miniconda

Miniconda is a small installer for Conda. Conda manages Python versions and packages in separate environments so projects do not interfere with each other.

1. Open the official install overview: [Installing Miniconda](https://www.anaconda.com/docs/getting-started/miniconda/install).
2. Choose the guide that matches your operating system **and** preferred method:
   - **Windows beginners:** prefer the **Windows graphical installer** (point-and-click).
   - **Windows command line:** use the Windows shell installer only if you are comfortable in PowerShell or Command Prompt.
   - **macOS beginners:** prefer the **macOS graphical (`.pkg`) installer**.
   - **macOS / Linux terminal:** use the terminal installer guide for your OS.
3. Follow **all** of the steps on that page for your choice. Do **not** stop after the first download command. Run each suggested command carefully, line by line.
4. When the installer asks whether to initialize Conda for your shell, choose **yes** (or leave the equivalent box checked).

When the install finishes, close the terminal and open a **new** one. Then check that Conda is available:

```bash
conda --version
```

You should see a version number. If the command is not found, reopen the terminal once more. On Windows, prefer the **Anaconda Prompt** application after install if PowerShell still cannot find `conda`.

### Windows tip: download path access denied

If you use the Windows shell installer and PowerShell fails with an error like:

```text
Invoke-WebRequest : Access to the path 'C:\windows\system32\Miniconda3-latest-Windows-x86_64.exe' is denied.
```

that usually means the download tried to write into a protected folder. Fix it by moving to your home directory first:

```powershell
cd $HOME
```

Then rerun the download and install commands from the official Windows shell installer page, starting again from the download step. If this feels stressful, switch to the Windows graphical installer instead.

## 3. Install Visual Studio Code

### 3.1. Download VS Code

Download and install Visual Studio Code from the official site: [https://code.visualstudio.com/Download](https://code.visualstudio.com/Download).

Choose the installer that matches your operating system and accept the default options unless you have a reason not to.

### 3.2. Install useful extensions

1. Open Visual Studio Code.
2. Click the Extensions icon in the left sidebar (four squares).
3. Search for and install each of these extensions:

   - `Python` (published by Microsoft)
   - `Jupyter`
   - `Python Indent`
   - `Rainbow CSV`
   - `Prettier - Code formatter`

The Python extension is the essential one for selecting a Conda environment and running scripts. The others are helpful defaults for notebooks, indentation, CSV files, and formatting.

## 4. Create a simple project folder

Before creating environments, make a short, simple folder for your work.

1. Create a folder called `Project` (or another short English name) on your computer.
2. Keep the full path short and simple.

**Very important path rules** (these avoid many beginner failures later):

1. Do **not** put the folder inside OneDrive or another cloud-synced folder.
2. Do **not** bury it under a very long chain of directories.
3. Prefer unaccented English letters (`A–Z`, `a–z`), numbers, and underscores `_` in folder names. Avoid accents, spaces, and punctuation when you can.

Good examples:

- Windows: `C:\Users\jv\Desktop\Project`
- macOS / Linux: `~/Project` or `~/Desktop/Project`

Open that folder in VS Code with **File → Open Folder…**.

## 5. Create and use a Conda environment

A Conda environment is an isolated workspace with its own Python and packages. Create one environment per project (or per course) instead of installing everything into `base`.

### 5.1. Open the right terminal

- **Windows:** after installing Miniconda, open **Anaconda Prompt** (recommended) or a new PowerShell window where `conda --version` already works.
- **macOS / Linux:** open a normal terminal as in section 1.

### 5.2. Go to your project folder

Use `cd` to enter the folder you created. Examples:

```powershell
# Windows (Anaconda Prompt or PowerShell)
cd C:\Users\jv\Desktop\Project
```

```bash
# macOS / Linux
cd ~/Project
```

Your prompt should show that you are inside that folder. If you are unsure, check the current directory:

```powershell
# Windows PowerShell
pwd

# Windows Anaconda Prompt (cmd): prints the current folder
cd
```

```bash
# macOS / Linux
pwd
```

You can also simply look at the path shown in the prompt.

### 5.3. Create the environment

Create a new environment named `myenv` with Python 3.11 (any supported recent Python 3.x is fine; keep the name short):

```bash
conda create -n myenv python=3.11 -y
```

Activate it:

```bash
conda activate myenv
```

Your prompt should now start with `(myenv)`. That prefix means later `python` and `pip` commands use this environment, not the system Python.

Check which Python you are using:

```bash
python --version
which python
```

On Windows Anaconda Prompt / PowerShell, use:

```powershell
python --version
where python
```

### 5.4. Install packages inside the environment

With `(myenv)` still active, install packages into **this** environment only.

With Conda (preferred when a package is available on conda channels):

```bash
conda install numpy pandas -y
```

With pip (useful when a package is mainly distributed on PyPI):

```bash
python -m pip install requests
```

Using `python -m pip` is safer than bare `pip`, because it installs into the same Python that `python` currently points to.

List what is installed:

```bash
conda list
```

Deactivate when you are done:

```bash
conda deactivate
```

To return to the project later:

```bash
conda activate myenv
```

## 6. Run Python from this environment in VS Code

1. Open your `Project` folder in VS Code if it is not already open.
2. Open the Command Palette:
   - Windows / Linux: `Ctrl + Shift + P`
   - macOS: `Cmd + Shift + P`
3. Type `Python: Select Interpreter` and choose that command.
4. Pick the interpreter that shows `myenv` (or the environment name you chose).

If you do not see it yet:

1. Make sure you created and activated the environment successfully in a terminal.
2. Click the refresh icon in the interpreter list, or reload the VS Code window.
3. On Windows, restart VS Code after installing Miniconda if the list is empty.

Create a small test file named `hello.py` in the project folder:

```python
import sys

print("Hello from", sys.executable)
```

Run it in VS Code with the play button, or from a VS Code terminal after activating the environment:

```bash
conda activate myenv
python hello.py
```

The printed path should point inside your Conda environment, not to a system-wide Python. That is the check that you are running inside the isolated environment.

You can also open a notebook (`.ipynb`) and choose the same `myenv` kernel in the kernel picker.

## 7. What to do next

You now have a local workflow: terminal → Conda environment → packages → VS Code interpreter → run Python.

Suggested next steps:

1. Follow a short Conda tutorial to practice creating environments, exporting them, and understanding channels: [Introduction to Conda for Data Scientists](https://edcarp.github.io/introduction-to-conda-for-data-scientists/aio/index.html).
2. Read the reproducibility section in the [Coding Practices Guidelines](/2025/07/30/Coding-tips.html), then continue with the linked Python learning resources there.
3. When you move work to the cluster, use [Using Agustina](/2025/08/01/Using-agustina.html) for module-based Conda, storage paths, and job helpers such as `subconda.sh`.
4. If shell navigation still feels shaky, revisit [Linux Sysadmin Basics](/2026/01/22/linux-sysadmin-basics.html).

Keep one habit from day one: activate the correct environment before installing packages or running project code. If the prompt does not show `(myenv)`, pause and activate first.

---
