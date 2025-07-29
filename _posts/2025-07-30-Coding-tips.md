---
title: Writing tips and tricks
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines
---

# Coding Practices Guidelines

## Foundation & Learning Resources
* **Software Carpentry Fundamentals**: Master basic coding practices through [Software Carpentry workshops](https://software-carpentry.org/lessons/) covering Unix shell commands, Git version control, and essential Python programming skills
* **Scientific Computing Scripting**: Develop proficiency in scripting for scientific computing using resources from the [NSF MolSSI group's comprehensive Python tutorials](https://education.molssi.org/python_scripting_cms/aio/index.html)
* **Command Line Editors**: Gain familiarity with standard editors like Vim, Emacs, or Nano to facilitate efficient editing and remote cluster work; [MIT's IAP "Missing Semester" class](https://missing.csail.mit.edu) provides excellent guidance on these tools and other essential development utilities

## Python Development Standards
* **Code Style & Quality**: We do most of our coding in python. Adhere to [PEP 8 guidelines](https://realpython.com/python-pep8/) for Python code formatting and use automated tools like Pylint, Google's yapf, black, or Sublime Text's AutoPEP8 to maintain consistent style standards
* **Type Safety**: Ideally, implement [optional type hints](https://docs.python.org/3/library/typing.html) to improve code readability and help others understand function signatures and variable types more easily
* **Style Guide Compliance**: Follow [Google's comprehensive Python style guide](http://google.github.io/styleguide/pyguide.html) for professional-quality code structure and documentation practices

## Project Structure & Organization
* **Beyond Notebooks**: While Jupyter notebooks are excellent for experimental analysis and tutorials, prioritize well-organized, documented codebases for rigorous testing and reproducible research using frameworks like [cookiecutter-data-science templates](https://github.com/drivendata/cookiecutter-data-science) or [dl-chem-101](https://github.com/rociomer/dl-chem-101)
* **Repository Management**: Utilize private repositories for continuous work backup
* **Code Publication**: Before open-sourcing code, thoroughly audit repositories to remove proprietary information, passwords, or private data; consider creating clean repositories for finalized code publication in the [group's GitHub organization](https://github.com/rlaplaza-lab)

## Reproducibility & Environment Management
* **Dependency Management**: Always use dependency managers like [Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html) or [uv](https://docs.astral.sh/uv/) for Python package management, or [Docker containers](https://docs.docker.com/get-started/overview/) for complex multi-language dependencies to ensure reproducible environments
* **Environment Documentation**: Maintain clear documentation of all software dependencies, versions, and system requirements necessary to reproduce your computational environment

## Deep Learning & Domain-Specific Resources
* **Chemistry/ML Integration**: Leverage the group's PyTorch implementations repository showcasing common Chemistry/ML deep learning tasks as templates for project structure and best practices
* **Deep Learning Education**: Utilize [d2l.ai resources](https://d2l.ai) for comprehensive deep learning fundamentals, including their preliminaries section and appendix of essential tools

## Collaboration & Continuous Learning
* **Knowledge Sharing**: Actively contribute to and consult the [group's code-tips repository](https://github.com/coleygroup/code-tips), which serves as a living collection of useful resources, helpful packages, and development tools
* **Ask for Help**: Don't hesitate to seek guidance from group members on development workflows, GitHub usage, or any coding challenges—everyone learns these skills over time
* **Workflow Optimization**: Trust your instincts when thinking "there must be a better way to do this"—there usually is, and the group can help you find it

## Modern Development Tools
* **AI-Assisted Coding**: Embrace large language model assistants like GitHub Copilot for code generation, while maintaining responsibility for understanding, testing, and validating all generated code
* **Code Accountability**: Ensure you can explain and verify any AI-generated code through appropriate testing and remain fully accountable for the final software product


