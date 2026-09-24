---
title: ORCA Tips and Troubleshooting Guide
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, computational-chemistry, orca, troubleshooting
---

# ORCA Tips and Troubleshooting Guide

This guide compiles common ORCA issues encountered in routine electronic-structure work and offers practical fixes. It follows the same spirit as the Gaussian guide, but the advice below is rooted in the official ORCA FAQ and manuals rather than in forum folklore or anecdotal advice.

> **Tip**: Use your browser's search function (Ctrl+F / Cmd+F) to jump to the relevant error text. The ORCA FAQ and the SCF/geometry sections of the manual are the best and most reliable reference points for troubleshooting.

---

## Memory and Resource Issues

### "Please increase MaxCore"

**Common symptom:**
- ORCA aborts with a message similar to `Please increase MaxCore`

**What the official guide says:**
The ORCA FAQ states that the SCF may need more memory than the user-specified `MaxCore` and that newer ORCA versions estimate the memory requirement early in the calculation. If the estimate is larger than `MaxCore` but smaller than `2*MaxCore`, ORCA gives a warning and continues. If it is larger than `2*MaxCore`, the job aborts. The FAQ also notes that `MaxCore` is the memory dedicated to each process, not the total job memory.

**Practical fixes:**

1. Increase the per-process memory in the job script or launch command.
2. Reduce the number of MPI processes if memory per process is too low.
3. Re-check whether the job is overcommitted on memory.

Example:

```text
! def2-TZVP TightSCF
%maxcore 2000
```

---

## SCF Convergence Issues

### SCF does not converge

**Common symptoms:**
- the energy oscillates or drifts for many cycles
- the calculation reaches many iterations without stabilizing
- the orbital gradient remains large

**What the official guide says:**
The ORCA SCF convergence chapter explains that convergence is a major practical issue in quantum chemistry. It documents convergence tolerances, damping, level shifting, DIIS, and the TRAH algorithm. It also notes that if the integral accuracy is worse than the SCF tolerance, a direct SCF calculation cannot converge properly.

**Practical fixes:**

1. Start with a more robust SCF level:
   ```text
   ! TightSCF
   ```
   or
   ```text
   ! SlowConv
   ```

2. Use damping or level shifting early in a difficult calculation:
   ```text
   %scf
     CNVDamp true
     DampFac 0.7
     DampErr 0.001
   end
   ```

3. Increase the maximum number of SCF cycles if needed.

4. For hard cases, allow the automatic TRAH fallback:
   ```text
   %scf
     AutoTRAH true
   end
   ```

The SCF manual specifically states that TRAH is useful when the conventional SCF procedures struggle.

### Open-shell or near-degenerate systems

**Common symptoms:**
- convergence stalls for radicals or transition-metal complexes
- the SCF oscillates between qualitatively different solutions
- the result is numerically stable but chemically questionable

**What the official guide says:**
The ORCA FAQ and SCF section emphasize that open-shell systems, especially transition-metal complexes, can be difficult. The guidance recommends checking `S^2`, UCO overlaps, and spin populations to confirm that the solution is meaningful.

**Practical fixes:**

1. Confirm the charge and multiplicity are chemically reasonable.
2. Try a better initial guess.
3. Use a more restrictive convergence setting for difficult electronic structures.
4. Inspect the resulting electronic structure rather than trusting a “converged” number blindly.

---

## Geometry Optimization Issues

### Optimization stalls or fails to converge

**Common symptoms:**
- the optimizer cycles without reaching a stable structure
- the job keeps stepping but never satisfies geometry thresholds
- a structure appears optimized but is still badly distorted

**What the official guide says:**
The ORCA geometry optimization chapter explains that optimization can be difficult even when the underlying method is valid. It also notes that floppy structures with many rotations around single bonds or soft dihedral modes are especially challenging. The manual describes the role of the initial Hessian, step type, and coordinate system in convergence.

**Practical fixes:**

```text
%geom
  MaxIter 50
  Step qn
  Trust 0.3
  coordsys redundant
  cartfallback true
end
```

1. Check the starting geometry.
2. Use a better initial Hessian or a more suitable coordinate system.
3. If the optimization is problematic, revisit the starting structure instead of assuming the failure is purely numerical.

### Transition state optimization fails or follows the wrong mode

**Common symptoms:**
- the calculation converges to a minimum instead of a transition state
- the optimizer identifies the wrong reaction coordinate
- multiple negative modes remain

**What the official guide says:**
The ORCA geometry optimization section explains that TS optimization is a separate problem: it maximizes the energy along one Hessian eigenmode while minimizing along the others. It also discusses `TS_Mode`, Hessian updates, and relaxed scans for TS optimization.

**Practical fixes:**

```text
! OptTS
%geom
  TS_Mode {M 0} end
  Calc_Hess true
  Recalc_Hess 5
end
```

1. Use an explicit TS setup.
2. Use a relaxed surface scan to locate the relevant reaction coordinate before TS optimization.
3. Confirm that the final structure has exactly one imaginary mode.

### Atoms merge into each other during optimization

**Common symptom:**
- neighboring atoms collapse into one another during optimization

**What the official guide says:**
The ORCA FAQ explicitly states that this usually occurs due to a poor or wrong initial molecular-orbital guess or a bad basis-set definition on the relevant atoms. The advice is to check the basis set on the problematic atoms and then the corresponding MOs.

This is a good example of an ORCA-specific issue: the problem is not a generic optimizer bug, but a quality-of-initial-guess and basis-definition issue.

---

## Frequency Calculation Issues

### Frequencies fail when the wavefunction is not converged

**Common symptoms:**
- the job stops before printing a frequency analysis
- a frequency calculation fails just after a geometry optimization
- the Hessian is not trusted because the SCF never reached a stable state

**What the official guide says:**
The SCF chapter is explicit: properties and numerical calculations, including `NumGrad` and `NumFreq`, are not performed on non-converged wavefunctions.

**Practical fixes:**

1. Ensure the SCF is converged before asking for frequencies.
2. Use a tight SCF and a well-converged geometry before `Freq`.
3. If necessary, optimize first and then compute frequencies in the same workflow:
   ```text
   ! B3LYP def2-TZVP Opt Freq
   ```

---

## File and I/O Problems

### MOREAD says that no orbitals were found in the .gbw file

**Common symptom:**
- ORCA reports that no orbitals are found when using `MOREAD`

**What the official guide says:**
The ORCA FAQ explains that ORCA writes the `.gbw` file immediately after reading the geometry and basis set information. If a stale `.gbw` file with the same basename remains in the working directory, it can be overwritten or interfere with the current calculation. The recommendation is to rename the old file before rerunning.

**Practical fix:**

```bash
mv oldjob.gbw oldjob.gbw.old
```

### Old inputs stop working after a version upgrade

**Common symptom:**
- a working input from an older ORCA version fails after installing a new version

**What the official guide says:**
The FAQ explicitly states that keywords and defaults may change between ORCA versions, and the same input may give slightly different results or fail entirely. The recommendation is to consult the current manual and release notes instead of assuming the old input remains valid.

---

## Installation and Launch Issues

### ORCA is installed but the program does not start

**Common symptoms:**
- the shell cannot find the ORCA executable
- the binary appears installed but no calculation starts
- the job exits immediately without writing output

**What the official guide says:**
The FAQ explains that ORCA is invoked from the command line on all platforms and gives the standard launch pattern:

```bash
<full orca binary folder path>/orca example.inp > example.out
```

It also explains the installation process for Linux/macOS: the `.run` installer is made executable with `chmod a+x`, followed by execution of the installation script.

**Practical fixes:**

1. Verify that the ORCA executable is on `PATH`.
2. Check that the working directory is the one expected by the script.
3. Run a minimal test calculation before moving to a large job.

---

## A good ORCA triage checklist

- Check the SCF convergence before trusting gradients, Hessians, or frequencies.
- If ORCA says “Please increase MaxCore,” confirm that the limit is per process.
- For difficult systems, first try `TightSCF`, damping, or `AutoTRAH`.
- For TS optimizations, verify the Hessian and reaction mode.
- For `MOREAD`, ensure stale `.gbw` files are not colliding with the new job.
- If the input was written for an older version, re-check the current manual before debugging deeper.

---

## Sources

- ORCA FAQ: https://www.faccts.de/docs/orca/6.0/manual/contents/faq.html
- ORCA SCF convergence: https://www.faccts.de/docs/orca/6.0/manual/contents/detailed/scfconv.html
- ORCA geometry optimization: https://www.faccts.de/docs/orca/6.0/manual/contents/detailed/geomopt.html
- ORCA installation and startup: https://www.faccts.de/docs/orca/6.0/manual/contents/faq.html

These are the official ORCA documentation pages used as the verified basis for the advice above.
