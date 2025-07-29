---
title: Writing tips and tricks
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, guidelines
---

# Scientific Paper Writing Guidelines

## About

This document provides a structured guide to help you write a clear, rigorous, and publishable **scientific manuscript**, especially in the **chemical sciences**. It is tailored to those using **[Overleaf](https://www.overleaf.com/)** with **[ACS](https://www.overleaf.com/latex/templates/acs-publishing-template/jngpwwcvzjtv)** or **[RSC](https://www.overleaf.com/latex/templates/rsc-article-template/fhxhhrxcqzvm)** LaTeX templates, but the core principles apply universally.

---

## Getting Started

### Choose the Right Journal

Before writing, define your **target journal**. This influences structure, style, and reference format. Consider:

- Scope of the journal
- Impact factor and audience
- Recent articles similar to your work

Links:
- [ACS Journals List](https://pubs.acs.org/page/journals)
- [RSC Journals List](https://www.rsc.org/journals-books-databases/about-journals/)
- [Journal Finder (Elsevier)](https://journalfinder.elsevier.com/)

### Use the Official Template

Overleaf templates ensure correct formatting and metadata setup.

- **ACS Template:** [Link](https://www.overleaf.com/latex/templates/acs-publishing-template/jngpwwcvzjtv)
- **RSC Template:** [Link](https://www.overleaf.com/latex/templates/rsc-article-template/fhxhhrxcqzvm)

Follow the structure provided and don’t override style commands unless necessary.

---

## General Structure of a Scientific Paper

### 1. Title

- Concise, descriptive, and specific
- Avoid jargon or overly broad titles
- Include keywords if relevant

### 2. Abstract

- 150–250 words
- Summarize the problem, key methods, major results, and conclusions
- Write it **last**, even though it appears first

### 3. Introduction

- Context and motivation
- Review of relevant work (with citations)
- Clearly state the **research question or hypothesis**
- End with a summary of **your contribution**

### 4. Results and Discussion

- Present results in **logical order**, not necessarily chronological
- Use **figures and tables** to highlight key findings
- Discuss implications, limitations, and comparisons with prior work
- Include **control experiments** or computational benchmarks if relevant

### 5. Methods / Experimental Section

- Describe methods with enough detail to allow replication
- Separate subsections for synthesis, measurements, and simulations
- Include software versions, parameters, and tools used
- Use SI units and standard nomenclature

### 6. Conclusions

- Recap major findings, make sure message is clear even if its repetitive
- Emphasize scientific impact
- Optionally outline future directions

### 7. References

- Use a **consistent and journal-appropriate** style (ACS, RSC, etc.)
- Use BibTeX with a reference manager like:
  - [Zotero](https://www.zotero.org/)
  - [Mendeley](https://www.mendeley.com/)
  - [JabRef](https://www.jabref.org/)

To generate BibTeX entries from DOIs, use:
- [DOI2Bib](https://www.doi2bib.org/)

To clean up `.bib` files, use:
- [betterbib (GitHub)](https://github.com/texworld/betterbib)

---

## Writing Tips

### Be Clear and Concise

- Prefer short, direct sentences
- Avoid unnecessary jargon
- Eliminate filler phrases (e.g., "it is worth mentioning that")

### Be Scientific

- Support all claims with **data or references**
- Use error bars, confidence intervals, and statistical significance where applicable
- Avoid overclaiming or hyping results

### Be Structured

- Use informative section headers
- Use logical transitions between paragraphs
- Ensure each paragraph has one core idea

### Be Visual

- Use well-labeled figures and tables
- Avoid clutter in graphs (no rainbow plots, overly small fonts)
- Refer to every figure/table in the main text

Use tools like:
- [ChemDraw](https://www.perkinelmer.com/category/chemdraw)
- [Matplotlib](https://matplotlib.org/) / [Plotly](https://plotly.com/)
- [Affinity Designer](https://affinity.serif.com/en-us/designer/) (for layout work)

---

## Overleaf and LaTeX Tips

- Use `\cite{}` with a `.bib` file to manage references
- Use `\autoref{}` or `\ref{}` for cross-referencing
- Define custom commands for repeated symbols or equations
- Add `% TODO:` comments to mark unfinished areas
- Use `[draft]` option in `\documentclass` for fast compilation during writing

---

## Final Checklist

Before submission or feedback, ensure:

### ✅ Scientific Rigor

- All claims supported with data or references
- All figures readable and reproducible
- All abbreviations defined at first use

### ✅ Formatting

- Correct journal style (template, references, headings)
- Figures and tables placed appropriately
- Units and nomenclature are consistent and SI-compliant when possible (for energies, we like kcal/mol or eV)

### ✅ Language and Clarity

- Spell-checked and grammar-checked
- No overly long or vague sentences
- Clear and engaging abstract

### ✅ Metadata

- Author names and affiliations are correct
- ORCID and email info included
- Proper keywords provided (if required)

---

## Bonus Tools

- [Grammarly](https://www.grammarly.com/) – grammar and style check
- [Writefull](https://writefull.com/) – language correction for scientific writing
- [LaTeX table converter](https://tableconvert.com/) – for tables
- [LaTeX equation writer](https://latexeditor.lagrida.com/) – for equations
- [Overleaf LaTeX Tutorials](https://www.overleaf.com/learn)

---

