---
title: Gaussian Basics and Troubleshooting
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, computational-chemistry, gaussian, troubleshooting
---

# Gaussian Basics and Troubleshooting

This guide starts with a short tutorial on how to write and run a basic Gaussian job—input layout, parallelism and memory, and geometry optimization for minima and transition states. The second half is a troubleshooting catalog for common errors. The companion [ORCA Basics and Troubleshooting](/2026/01/01/ORCA-tips-and-troubleshooting.html) guide covers the same topics for ORCA. For background on how the SCF works in practice (scaling, integrals, guesses, convergence accelerators, and analytical gradients), see [SCF in Practice](/2026/10/05/SCF-in-practice.html). The error catalog builds on community resources, particularly [Zhe Wang's comprehensive error guide](https://wongzit.github.io/gaussian-common-errors-and-solutions/).

> **Tip**: Use your browser's search function (Ctrl+F / Cmd+F) to quickly find specific error messages.

---

## Basic input file

A Gaussian input has Link 0 lines (`%…`), a route section (`# …`), a title, charge and multiplicity, then coordinates. Blank lines separate the title from the molecule specification and usually end the file.

```text
%mem=8GB
%nprocshared=4
%chk=water_opt.chk
# opt freq b3lyp/def2SVP EmpiricalDispersion=GD3BJ

Water optimization and frequencies

0 1
O    0.000000    0.000000    0.117300
H    0.000000    0.757200   -0.469200
H    0.000000   -0.757200   -0.469200
```

- `%mem`, `%nprocshared`, and `%chk` set memory, shared-memory cores, and the checkpoint file.
- The route line starts with `#` and lists method, basis, and job keywords.
- The title line is free text; the next non-blank line is `charge multiplicity`, then atoms.
- Prefer Cartesian coordinates unless you have a reason to use a Z-matrix. Visualization tools (GaussView, Avogadro) help avoid formatting mistakes.

London dispersion is critical for organics, noncovalent contacts, and many reaction energetics, yet it is missing from most hybrid functionals (including bare B3LYP and PBE0). Add `EmpiricalDispersion=GD3BJ` (or GD3), or use a functional that already includes a dispersion model such as `wB97XD`.

---

## Recommended functionals

Practical starting points in Gaussian:

- **B3LYP-D3(BJ)** — everyday hybrid with Grimme D3(BJ): `b3lyp … EmpiricalDispersion=GD3BJ`.
- **PBE0-D3(BJ)** — solid global hybrid (Gaussian keyword `PBE1PBE`) with the same dispersion keyword: `PBE1PBE … EmpiricalDispersion=GD3BJ`.
- **ωB97X-D** — range-separated hybrid with a built-in dispersion correction: `wB97XD` (do not stack an extra `EmpiricalDispersion` term on top).

Useful DFT benchmark and method papers:

- [GMTKN55](https://doi.org/10.1039/C7CP04913C) (Goerigk et al., 2017) — large main-group thermochemistry, kinetics, and noncovalent-interaction benchmark of many dispersion-corrected DFAs.
- [Grimme, WIREs Comput. Mol. Sci. (2011)](https://doi.org/10.1002/wcms.30) — review of London dispersion corrections to DFT and why they matter beyond weakly bound dimers.
- [Grimme et al., J. Chem. Phys. (2010)](https://doi.org/10.1063/1.3382344) — DFT-D3 parametrization (the usual `EmpiricalDispersion=GD3` / `GD3BJ` family).
- [Chai and Head-Gordon, PCCP (2008)](https://doi.org/10.1039/B810189B) — ωB97X-D functional (Gaussian `wB97XD`).
- [Mardirossian and Head-Gordon, PCCP (2014)](https://doi.org/10.1039/C3CP54374A) — ωB97X-V (strong hybrid with VV10; useful context even when you use `wB97XD` in Gaussian).
- [Goerigk et al., ChemPhysChem (2011)](https://doi.org/10.1002/cphc.201100826) — dispersion-corrected DFT on the S66/S66x8 noncovalent interaction sets.

---

## Recommended basis sets

Prefer the Karlsruhe **def2** family for DFT work (Gaussian keywords `def2SVP` and `def2TZVP`):

- **def2-SVP** — economical split-valence polarized set for geometry optimizations, frequencies, and exploratory energetics.
- **def2-TZVP** — triple-zeta polarized set for more converged single points or final energetics once the structure is settled.

These sets were designed with DFT (and HF/MP2) in mind, are balanced across much of the periodic table, and pair heavier elements with Stuttgart effective core potentials so you do not need a separate all-electron treatment for most mid-to-late-row atoms. The examples below use `def2SVP` for routine jobs; step up to `def2TZVP` when you need tighter energetics.

The original design and accuracy assessment across a large molecular test set is in [Weigend and Ahlrichs (2005)](https://doi.org/10.1039/B508541A).

---

## Parallelism and memory

`%nprocshared` sets how many cores Gaussian uses with shared-memory parallelism. `%mem` is the memory Gaussian is allowed to allocate. On many systems Gaussian uses roughly 1 GB more than the value you set, so leave headroom relative to the job script:

```text
%mem=8GB
%nprocshared=4
```

```bash
#SBATCH --mem=16G   # comfortably above %mem
#SBATCH --cpus-per-task=4
```

Match `%nprocshared` to the CPU count you requested. If memory is limited, reduce cores first: fewer processors lower the peak memory demand. Use `%nprocshared` rather than Linda-style `nprocl` unless your site explicitly supports Linda.

Checkpoint files (`%chk=…`) are worth keeping for restarts and chained jobs (`geom=allcheck`, `guess=read`). Point `GAUSS_SCRDIR` at a large scratch filesystem when jobs write heavy intermediate files.

---

## Optimizing minima

For a ground-state minimum, put `opt` on the route line. Computing frequencies in the same job confirms a true minimum (no imaginary modes):

```text
# opt freq b3lyp/def2SVP EmpiricalDispersion=GD3BJ
```

Useful options when an optimization is slow or stubborn:

```text
# opt=(calcfc,maxcycle=200) b3lyp/def2SVP EmpiricalDispersion=GD3BJ
```

- `calcfc` computes force constants at the start (often more stable than a crude guess Hessian).
- `maxcycle` raises the step limit when the structure is close but not quite there.
- Alternatives such as `opt=RFO`, `opt=GDIIS`, or `opt=cartesian` are covered in the troubleshooting half if the default optimizer stalls.

What “done” looks like: the log reports that the optimization completed (or “Normal termination”), and a frequency job shows **zero** imaginary frequencies for a minimum. Inspect the geometry in a viewer before trusting the energy.

---

## Optimizing transition states

A transition-state search maximizes energy along one mode and minimizes along the others. A typical Gaussian setup is:

```text
# opt=(ts,calcfc,noeigentest) freq b3lyp/def2SVP EmpiricalDispersion=GD3BJ
```

Practical checklist:

1. Start from a good TS guess (often from a scan or a previous lower-level TS).
2. Use `opt=ts` with an initial Hessian (`calcfc` or `calcall` when affordable).
3. After convergence, run frequencies and confirm **exactly one** imaginary mode that matches the reaction coordinate.

`noeigentest` (or `noeigen`) skips the intermediate eigenvalue check when the optimizer reports the wrong number of negative eigenvalues mid-run. That can let the search continue, but it does **not** replace a final frequency verification—see the TS troubleshooting section below.

When a calculation fails or behaves oddly, use the catalog below. Search for the error text or the symptom that matches your output.

---

## Troubleshooting common issues

---

## Memory and Resource Issues

### Insufficient Memory Errors

**Common Error Messages:**
- `CISAX needs XXXXX more words of memory`
- `XXXXX words are not enough for AlAXAO`
- `galloc: could not allocate memory`
- `malloc failed: Resource temporarily unavailable`

**Solutions:**

1. **Increase memory allocation**: Use the `%mem` directive at the top of your input file:
   ```
   %mem=8GB
   ```
   Note: Gaussian actually uses about 1 GB more than specified, so set `%mem` to at least 1 GB less than your job script's memory allocation.

2. **Reduce CPU cores**: If memory is limited, reduce the number of processors:
   ```
   %nprocshared=4
   ```
   This reduces memory requirements per core.

3. **Check job script settings**: Ensure your job submission script (SLURM, PBS, etc.) allocates sufficient memory:
   ```bash
   #SBATCH --mem=16G  # Should be ~1GB more than %mem
   ```

---

## Geometry Optimization Issues

### Optimization Not Converging

**Error Messages:**
- `Number of steps exceeded, NStep= 100`
- `Delta-x Convergence NOT Met`
- `Maximum of *** iterations exceeded in RedStp`

**Solutions:**

1. **Increase maximum cycles**: Add `opt=maxcycle=n` where n is 2-3 times the current number of steps:
   ```
   # opt=calcfc opt=maxcycle=200
   ```

2. **Try different optimization methods**:
   - `opt=RFO` - Rational Function Optimization
   - `opt=GDIIS` - Geometry Direct Inversion in Iterative Subspace
   - `opt=GEDIIS` - Geometry Energy-Direct Inversion in Iterative Subspace

3. **Switch coordinate system**: If Z-matrix fails, try Cartesian coordinates:
   ```
   opt=cartesian
   ```

4. **Check initial geometry**: Verify your starting structure is reasonable. Use molecular visualization software (GaussView, Avogadro) to inspect the geometry.

5. **Modify symmetry**: Sometimes reducing symmetry helps:
   ```
   # symm=loose
   ```

### Transition State Optimization Issues

**Error Message:**
- `Wrong number of Negative eigenvalues: Desired= 1 Actual= 4`

**Solution:**
If you're optimizing a transition state but get multiple negative frequencies, use:
```
opt=(ts,noeigen)
```
This skips the eigenvalue check. However, always verify your final structure has exactly one imaginary frequency!

---

## Frequency Calculation Issues

### Frequency Calculation Errors

**Error Messages:**
- `Error in INITNF`
- `Linear search skipped for unknown reason`
- `Inconsistency: ModMin= N Eigenvalue= MM`

**Solutions:**

1. **Ensure optimization converged**: Frequency calculations require a fully optimized geometry. Check that your optimization completed successfully.

2. **Use `freq=readfc`**: If you have a checkpoint file with force constants:
   ```
   # freq=readfc
   ```

3. **Recalculate from scratch**: Sometimes it's best to re-optimize and then compute frequencies in a single job:
   ```
   # opt freq
   ```

---

## SCF Convergence Issues

For the conceptual background (guesses, DIIS, SOSCF, direct vs conventional integrals), see [SCF in Practice](/2026/10/05/SCF-in-practice.html). The recipes below are Gaussian-specific.

### SCF Not Converging

**Error Messages:**
- `Convergence failure -- run terminated`
- `SCF Done:  E(RB3LYP) =  ...  A.U. after   50 cycles`

**Solutions:**

1. **Use convergence aids**:
   ```
   scf=(conver=8,xqc)
   ```
   - `conver=8` sets tighter convergence criteria
   - `xqc` uses quadratic convergence

2. **Try different initial guess**:
   ```
   guess=mix
   guess=huckel
   guess=read  # from checkpoint file
   ```

3. **Use damping**:
   ```
   scf=(conver=8,damp)
   ```

4. **Check for problematic systems**: 
   - Open-shell systems may need `stable=opt`
   - Systems with near-degeneracies may need different methods

---

## Post-HF Method Convergence

### CCSD/CCSD(T) Not Converging

**Error Messages:**
- `Error termination via Lnk1e in l913.exe` (after CCSD iterations)
- Large amplitudes in the output

**Solutions:**

1. **Increase maximum cycles**:
   ```
   ccsd(t,maxcyc=100)
   ```
   Default is 50 cycles.

2. **Check convergence trend**: Look at the `DE(Corr)` values in the output. If they're converging (approaching a stable value), increasing cycles should help.

3. **Verify reference state**: Ensure your HF reference is reasonable. Try:
   ```
   stable=opt
   ```
   before the CCSD calculation.

4. **Try different basis set**: Sometimes a smaller or different basis set helps establish convergence.

---

## File and I/O Errors

### Disk Space Issues

**Error Messages:**
- `Erroneous write. write 122880 instead of 4239360`
- `writwa: No space left on device`
- `Erroneous write during file extend`

**Solutions:**

1. **Check disk space**: 
   ```bash
   df -h
   du -sh ~/scratch/
   ```

2. **Clean up scratch directory**: Remove old checkpoint files and temporary files.

3. **Use smaller basis set**: For very large calculations, consider using a smaller basis set or reducing system size.

4. **Set scratch directory**: Ensure `GAUSS_SCRDIR` points to a directory with sufficient space:
   ```bash
   export GAUSS_SCRDIR=/path/to/large/disk/scratch
   ```

### Checkpoint File Issues

**Error Messages:**
- `Error termination in NtrErr: Operation on file out of range`
- `Error imposing constraints`

**Solutions:**

1. **Regenerate checkpoint file**: The checkpoint file may be corrupted or incomplete. Re-run the calculation that generates the needed data.

2. **Don't rely on incomplete checkpoints**: If a previous job failed or was killed, don't try to read from its checkpoint file.

---

## Input File Errors

### Z-Matrix and Coordinate Errors

**Error Messages:**
- `End of file in Zsymb`
- `Found a string as input`
- `There are no atoms in this input structure`
- `Symbol not found in Z-matrix`
- `Variable index is out of range`
- `Determination of dummy atom variables in z-matrix conversion failed`

**Solutions:**

1. **Check input format**: Ensure proper spacing and formatting in your Z-matrix or Cartesian coordinates.

2. **Use Cartesian coordinates**: If Z-matrix conversion fails, switch to Cartesian:
   ```
   # opt=cartesian
   ```

3. **Verify atom definitions**: Check that all atoms are properly defined and variables are correctly referenced.

4. **Use GaussView or similar**: Generate input files using molecular visualization software to avoid formatting errors.

---

## Method-Specific Issues

### DFT Functional Limitations

**Error Message:**
- `No func 3rd derivs with HSE` (or similar for other functionals)

**Explanation:**
Some functionals don't support third-order derivatives needed for hyperpolarizability calculations.

**Solutions:**

1. **For polarizability only**: Use `polar=Numerical`:
   ```
   # polar=Numerical
   ```
   This calculates polarizability α but may fail for hyperpolarizability β.

2. **Use different functional**: Switch to a functional that supports third-order derivatives (most standard functionals do).

3. **Check output**: Sometimes the desired property (e.g., α) is calculated before the error occurs, so check earlier in the output file.

---

## System and Permission Errors

### Gaussian Installation Issues

**Error Messages:**
- `Files in the Gaussian directory are world accessible. This must be fixed.`
- `failed to open execfile`

**Solutions:**

1. **Fix permissions**:
   ```bash
   chmod -R 750 /path/to/Gaussian
   ```

2. **Check Linda vs. OpenMP**: If using `nprocl`, ensure your system supports Linda. Otherwise, use `nprocshared`:
   ```
   %nprocshared=8  # instead of nprocl
   ```

3. **Verify environment variables**: Check that `g09root` or `g16root` and `GAUSS_EXEDIR` are set correctly.

---

## Monitoring and performance tips

1. **Watch output in real time**: `tail -f jobname.log` while the job runs.
2. **Check for convergence**: Look for “Optimization completed” or “Normal termination”.
3. **Save intermediate results**: Keep checkpoint files for restarts and `geom=allcheck` chains.
4. **Use an appropriate basis**: Larger is not always better for exploratory work.
5. **Parallelize wisely**: More cores trade against memory; match `%nprocshared` to the allocation.
6. **Prefer `opt=calcfc`** when the default Hessian guess is unreliable.

---

## Quick Reference: Common Fixes

| Problem | Quick Fix |
|---------|-----------|
| Out of memory | Increase `%mem` or decrease `%nprocshared` |
| Optimization not converging | Add `opt=maxcycle=200` or try `opt=RFO` |
| SCF not converging | Add `scf=(conver=8,xqc)` or `scf=damp` |
| Multiple negative frequencies | Use `opt=(ts,noeigen)` but verify result |
| Disk space error | Clean scratch directory or use smaller basis |
| Checkpoint file error | Regenerate checkpoint from scratch |
| Z-matrix error | Switch to `opt=cartesian` |

---

## Additional Resources

- [Gaussian Official Documentation](https://gaussian.com/)
- [Gaussian User's Reference](https://gaussian.com/man/)
- [Zhe Wang's Gaussian Error Guide](https://wongzit.github.io/gaussian-common-errors-and-solutions/)
- [Crawford Group Computational Chemistry Resources](https://github.com/CrawfordGroup/ProgrammingProjects)

---

## Getting Help

If you encounter errors not covered here:

1. **Check the full output file**: Errors often have context earlier in the file
2. **Search error messages**: Many errors are documented online
3. **Consult colleagues**: Often someone has seen the same issue
4. **Gaussian support**: For licensed users, contact Gaussian Inc. support

Remember: Computational chemistry calculations can be finicky. When in doubt, start simpler (smaller basis set, fewer atoms) and work your way up!
