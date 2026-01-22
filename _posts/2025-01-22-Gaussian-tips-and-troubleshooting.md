---
title: Gaussian Tips and Troubleshooting Guide
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, computational-chemistry, gaussian, troubleshooting
---

# Gaussian Tips and Troubleshooting Guide

This guide compiles common errors encountered when running Gaussian calculations and provides practical solutions. Whether you're optimizing geometries, computing frequencies, or running excited-state calculations, these tips should help you navigate the most frequent issues.

> **Tip**: Use your browser's search function (Ctrl+F / Cmd+F) to quickly find specific error messages. This guide is inspired by and builds upon resources from the computational chemistry community, particularly [Zhe Wang's comprehensive error guide](https://wongzit.github.io/gaussian-common-errors-and-solutions/).

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

## General Best Practices

### Input File Organization

1. **Always include route section**: Start with `#` followed by method and keywords
2. **Use title line**: Provide a descriptive title
3. **Specify charge and multiplicity**: Essential for proper calculation
4. **Use checkpoint files**: Include `%chk=filename.chk` for recovery

### Example Well-Structured Input:

```
%mem=8GB
%nprocshared=4
%chk=water_opt.chk
# opt freq b3lyp/6-31g(d)

Water optimization

0 1
O    0.000000    0.000000    0.117300
H    0.000000    0.757200   -0.469200
H    0.000000   -0.757200   -0.469200
```

### Monitoring Calculations

1. **Watch output in real-time**: Use `tail -f` to monitor progress:
   ```bash
   tail -f jobname.log
   ```

2. **Check for convergence**: Look for "Optimization completed" or "Normal termination"

3. **Save intermediate results**: Use checkpoint files to restart or extract data

### Performance Tips

1. **Use appropriate basis sets**: Don't use larger basis sets than necessary
2. **Parallelize wisely**: More cores don't always mean faster (memory trade-off)
3. **Use efficient methods**: Consider `opt=calcfc` to calculate force constants once
4. **Chain calculations**: Use `geom=allcheck` to continue from previous calculations

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
