---
title: SCF in Practice
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, computational-chemistry, scf
---

# SCF in Practice

This post is a short, package-agnostic primer on the self-consistent field (SCF) procedure used in routine Hartree–Fock (HF) and Kohn–Sham DFT calculations. The goal is basic intuition: what the loop does, what drives the cost (integrals vs diagonalization, disk vs RAM), how basis sets, XC grids, and implicit solvent enter, how jobs are started and converged, and what gradients and Hessians cost relative to the energy. For program-specific keywords and failure recipes, see the [ORCA](/2026/01/01/ORCA-tips-and-troubleshooting.html) and [Gaussian](/2026/01/01/Gaussian-tips-and-troubleshooting.html) guides.

## What the SCF loop is

In HF and DFT, the electronic energy depends on the occupied molecular orbitals (or equivalently the density matrix). Those orbitals come from diagonalizing an effective one-electron Hamiltonian—the Fock matrix in HF, the Kohn–Sham (KS) matrix in DFT—which itself depends on the density. The SCF procedure closes that loop:

1. Start from an initial density (or orbital guess).
2. Build the Fock/KS matrix from that density.
3. Diagonalize (or otherwise update orbitals) to get a new density.
4. Repeat until the density, energy, and orbital gradient stop changing within a chosen threshold.

Almost every downstream result—geometry steps, frequencies, and most molecular properties—assumes a converged SCF. A “finished” optimization with a poorly converged electronic structure is not trustworthy.

## Basis functions: primitives, contractions, and how $N$ grows

Let $N$ be the number of **contracted** atomic basis functions in the calculation. That $N$ is the dimension of the Fock/KS and overlap matrices, and it is what appears in the familiar SCF scaling arguments below.

A **primitive Gaussian-type orbital (GTO)** is a single Gaussian radial factor times an angular factor. A **contracted GTO (cGTO)** is a fixed linear combination of several primitives that share the same center and angular momentum:

$$
\chi_\mu(\mathbf{r}) = \sum_{k} d_{k\mu}\, g_k(\mathbf{r}).
$$

Production HF/DFT bases (def2, Pople, correlation-consistent, …) are almost always **contracted**: the contraction coefficients $d_{k\mu}$ are frozen, so the SCF works in a smaller space of $N$ cGTOs while each two-electron integral over cGTOs still expands into many primitive integrals. That is the usual efficiency trade-off for SCF: fewer variational degrees of freedom, more work per matrix element.

“Uncontracted” bases (closer to raw primitive GTOs) enlarge $N$ and are more common when radial flexibility matters for **correlated wavefunction methods** (MP2, CC, …), which need to describe dynamical correlation, not only the mean-field density. For everyday DFT/HF geometry work, contracted sets such as def2-SVP are the default.

### How many functions per atom? A def2-SVP sketch

def2-SVP is a polarized valence double-$\zeta$ set. With spherical harmonics (5 $d$ and 7 $f$ functions), typical contracted sizes are:

| Atom | Electrons treated in the basis | Contraction (primitives $\rightarrow$ cGTOs) | Spherical $N$ per atom |
| --- | --- | --- | ---: |
| H | all (1) | $(4s,1p)\rightarrow[2s,1p]$ | 5 |
| C | all (6) | $(7s,4p,1d)\rightarrow[3s,2p,1d]$ | 14 |
| Fe | all (26) | $(14s,9p,5d,1f)\rightarrow[5s,3p,2d,1f]$ | 31 |
| Au | valence only (19); 60 in ECP | $(7s,6p,5d,1f)\rightarrow[6s,3p,2d,1f]$ | 32 |

Two takeaways:

- **$N$ tracks the electrons you actually expand in the basis, not the nuclear charge alone.** Going from H to C to Fe grows the per-atom basis because more shells are occupied and polarized.
- **Effective core potentials (ECPs / pseudopotentials) cut that growth for heavy elements.** From Rb onward, def2 sets pair with def2-ECP: gold’s 60 core electrons are removed from the SCF, so Au/def2-SVP has roughly the same number of basis functions as all-electron Fe/def2-SVP (~32 vs ~31), not “79/26 times more.” Without an ECP, an all-electron basis for Au would be far larger and usually needs a relativistic Hamiltonian as well.

So for a molecule, $N \approx \sum_{\text{atoms}} n_{\text{basis}}(Z)$, with $n_{\text{basis}}$ jumping at shells and **dropping again when an ECP replaces the core**.

### Diffuse functions: anions vs cations

Standard polarized valence sets (def2-SVP, 6-31G(d), …) are tuned for **neutral** densities. Charge changes how spatially extended the electron cloud is, and the basis has to follow.

- **Anions** (and loosely bound electrons, Rydberg states, some excited states) need **diffuse** functions—primitives with small exponents that reach farther from the nucleus. Without them, the extra electron is artificially confined, electron affinities and anion energetics are often badly wrong, and the SCF may still “converge” to a basis-set artifact. Augmented sets (aug-cc-pVXZ, ma-def2, 6-31+G(d), …) add those diffuse shells and increase $N$, which also makes integral screening harder (more diffuse = denser overlap = costlier Fock builds and sometimes tougher SCF).
- **Cations** are the opposite trend: removing electrons **contracts** the density. Diffuse functions are usually unnecessary for the cation itself; a solid polarized valence set is often enough. (You may still want diffuse functions on a large neutral or anionic partner in the same calculation.)
- **Practical pattern:** add diffuse functions when the species of interest is anionic or when absolute electron-binding matters; do not scatter “+” or “aug” onto every atom by default—each diffuse shell grows $N$ and can worsen linear dependence in large bases.

## Cost: two-electron integrals vs Fock diagonalization

Each SCF cycle has two different kinds of work:

1. **Build the Fock/KS matrix** — dominated by two-electron contributions (Coulomb, and exact exchange for HF/hybrids), plus the XC grid for DFT.
2. **Update orbitals** — classically a generalized diagonalization of an $N \times N$ matrix,
   $$\mathbf{F}\,\mathbf{C} = \mathbf{S}\,\mathbf{C}\,\boldsymbol{\varepsilon},$$
   which scales as $O(N^3)$.

Formally, AO two-electron integral work scales as $O(N^4)$ before screening. With density-based screening and locality, the effective Fock-build cost is often closer to $O(N^2)$–$O(N^3)$ for large molecules, but **raising the basis quality still hurts**: more functions per atom means denser, less local integrals and a heavier XC grid.

**Which piece wins?** For the system sizes typical in molecular DFT/HF (up to a few thousand basis functions), **Fock/KS construction almost always dominates** wall time. Diagonalization’s $O(N^3)$ only becomes competitive for very large $N$ or when integral engines and screening are extremely efficient. Hybrid functionals keep a large exact-exchange term, so they stay integral-heavy longer than pure GGAs.

Rules of thumb:

- Double-$\zeta$ → triple-$\zeta$ increases $N$ per atom substantially; each Fock build often slows by more than the naive $N$ ratio.
- Hybrid DFT pays for Coulomb-like terms plus a fraction of exact exchange; that exchange piece is usually the expensive one compared with a pure GGA.
- A larger basis also makes a bad initial guess more painful: each cycle is heavier, and you may need more cycles before DIIS settles.

## Numerical XC grids

In KS DFT, the exchange–correlation (XC) energy and potential are not computed as analytic AO integrals the way Coulomb and exact exchange are. Instead, the density (and its derivatives for GGAs and beyond) is sampled on a real-space grid—typically a union of atom-centered grids with radial and angular points—and the XC contribution is integrated numerically into the KS matrix **every SCF cycle**.

Practical consequences:

- Grid work scales roughly with (number of grid points) $\times$ (basis functions that have appreciable weight at those points). Denser grids raise both cost and the numerical noise floor of the energy.
- Grids are a DFT-specific cost channel, separate from whether AO integrals are stored on disk or recomputed.
- Coarser grids are often acceptable early in the SCF; the final energy—and especially gradients and Hessians—usually need a denser grid so that numerical noise does not masquerade as forces or force constants.
- If two “identical” DFT jobs differ only in grid settings, small energy differences are expected; do not over-interpret microhartree changes across grid changes.

## Implicit solvent in the SCF

Continuum solvation models (PCM, CPCM, SMD, COSMO, …) do **not** add explicit solvent molecules. They treat the solvent as a polarizable dielectric outside a molecular cavity. The solvent's reaction field depends on the solute’s charge density, so it must be updated **inside the SCF**, not bolted on after a gas-phase calculation (unless you deliberately choose a non-self-consistent approximation).

In each cycle the workflow is conceptually:

1. Build the usual vacuum Fock/KS pieces from the current density.
2. From that density, compute the solute–solvent interaction (apparent surface charges on the cavity, or an equivalent reaction potential).
3. Add the corresponding one-electron operator to $\mathbf{F}$ / the KS matrix.
4. Obtain a new density and repeat until both the electronic structure **and** the reaction field are consistent.

Consequences for practice:

- Solvated SCF is a bit more expensive per cycle than the gas-phase analogue (cavity setup and reaction-field build), but the scaling is usually mild compared with exact exchange or a large basis.
- The optimized density in solvent can differ from the gas phase—especially for ions and polar molecules—so geometries, dipole moments, and orbital energies shift. That is the point of putting solvent in the SCF.
- Charged solutes are where continuum models are most used and most delicate: the cavity, radii, and electrostatic asymptotics matter; combining anions with **both** diffuse functions and a continuum solvent is common, but check for SCF instability or over-polarization if something looks chemically odd.
- Non-electrostatic terms (cavitation, dispersion, SMD-style corrections) may be added to the free energy after or alongside the SCF depending on the model; the part that changes the orbitals is the electrostatic reaction field.

## Integrals in practice: conventional vs direct

Two classical strategies exist for the AO two-electron integrals $(\mu\nu\mid\lambda\sigma)$.

### Conventional (in-disk) SCF

Compute the unique integrals once, write them to a scratch file, and **read them back every SCF cycle** to assemble $\mathbf{F}$.

Consequences on real HPC:

- **Disk capacity:** the integral file grows as $O(N^4)$ before symmetry/screening; even with screening it can reach tens or hundreds of GB. Shared scratch that fills up kills the job for everyone on the node.
- **I/O bandwidth and latency:** each cycle re-reads that file. On a busy cluster filesystem (NFS, shared parallel FS), many cores doing large sequential or random reads contend with other users. Wall time then tracks **I/O wait**, not CPU flops.
- **Local scratch helps** (node-local SSD/NVMe): conventional SCF can still be fine for small-to-medium $N$ with many iterations, provided the file fits and the disk is fast.
- **Network scratch hurts:** if the scratch filesystem is remote (shared NFS or similar), conventional SCF is often slower than recomputing integrals.

### Direct SCF

Recompute screened integral batches every iteration instead of storing the full set. The bottleneck shifts to **CPU and RAM**:

- **RAM** holds the density matrix, Fock/KS matrix, MO coefficients, integral buffers, DIIS history, and (for DFT) grid batches. Parallel runs often need memory **per process**; too little memory causes thrashing or aborts.
- Semidirect schemes keep only the most expensive or most reused integral classes on disk and recompute the rest—a compromise when RAM is tight but some I/O is acceptable.

Modern defaults lean toward direct (or semidirect) SCF once $N$ is large enough that the integral file would be huge or the filesystem would dominate. Whatever the mode, integral and grid accuracy must stay tighter than the SCF convergence threshold: you cannot converge the density below the noise in $\mathbf{F}$.

## Initial guesses

The first density decides how hard the early SCF cycles will be. Common options:

- **Core Hamiltonian / minimal Hückel-like guesses** — cheap and often crude; fine for simple closed-shell organics, fragile for metals or open shells.
- **Superposition of atomic densities (SAD) and related atomic guesses** — usually a much better starting density for molecules.
- **Reading orbitals from a previous calculation** — often the best choice when the geometry or method changed only slightly (optimization steps, basis upgrades, or restarting a failed job).
- **Open-shell tricks** — mixing HOMO/LUMO character or breaking $\alpha$/$\beta$ symmetry can help land on a broken-symmetry or diradicaloid solution instead of collapsing to a closed-shell state.

A good guess often saves more wall time than raising the maximum number of cycles. A bad guess can send the SCF into oscillation between qualitatively different electronic structures.

## Getting to convergence

Plain Roothaan–Hall iteration (build Fock/KS → diagonalize → repeat) frequently oscillates: each new density overcorrects the previous one. Production codes therefore accelerate and stabilize the update:

- **Damping and level shifting** — mix old and new densities, or shift virtual levels, to damp oscillations early on.
- **DIIS (Direct Inversion in the Iterative Subspace)** — store recent Fock/KS matrices and error vectors (e.g. the orbital gradient) and extrapolate a linear combination that drives the error toward zero. This is the workhorse accelerator for routine SCF.
- **SOSCF / second-order orbital optimization** — treat orbital rotations more like a Newton step using an approximate **orbital** Hessian (second derivatives with respect to orbital rotations, not nuclear coordinates). Useful when DIIS stalls, especially for difficult open-shell or near-degenerate cases.
- **Quadratic or trust-region fallbacks** — last-resort algorithms in some packages when standard DIIS/SOSCF paths fail.

A practical order of operations: start from a sensible guess, let DIIS (the usual default) work, add damping or a level shift if the energy oscillates, and only then escalate to second-order or package-specific robust SCF modes. Tightening thresholds without fixing the underlying electronic-structure problem mostly burns cycles.

## Analytical SCF gradients

Once the HF or DFT density is converged, nuclear gradients

$$
G_{A,\alpha} = \frac{\partial E}{\partial R_{A,\alpha}}
$$

can be evaluated **analytically**. That is much cheaper than numerical differentiation, which would require on the order of $3N_{\mathrm{atoms}}$ extra energy calculations (or twice that for central differences).

An analytical SCF gradient is typically on the order of **one extra SCF-like step** after convergence. The main pieces are:

- derivatives of the AO integrals with respect to nuclear positions,
- XC contributions on the grid, including grid-weight / grid-center derivative terms in DFT,
- Pulay terms that appear because the basis functions move with the nuclei (basis-set incompleteness / atom-centered basis effects).

Cost still tracks basis-set size in the same broad way as the energy: larger $N$ means heavier derivative-integral and XC work. Gradients are also more sensitive to a loose SCF or a coarse XC grid than a rough single-point energy is—noisy forces slow or mislead geometry optimizations even when the energy looks “almost” converged.

## Nuclear Hessians: how they are computed and why they matter

The nuclear Hessian is the matrix of second derivatives of the energy with respect to nuclear coordinates,

$$
H_{A\alpha,B\beta} = \frac{\partial^2 E}{\partial R_{A\alpha}\,\partial R_{B\beta}}.
$$

It is what you diagonalize (after mass-weighting) to get vibrational frequencies and normal modes.

**Why it matters in practice:**

- **Characterize stationary points:** a true minimum has all real frequencies (no imaginary modes); a transition state has exactly one imaginary mode along the reaction coordinate.
- **Thermochemistry:** zero-point energy, enthalpy, and free-energy corrections come from the frequency spectrum.
- **Better geometry optimization:** many optimizers start from a cheap approximate Hessian and update it; computing an exact Hessian (or a good partial one) can stabilize difficult optimizations, at a price.

**How they are computed:**

- **Analytical Hessian:** differentiate the gradient expressions again. For HF/DFT this requires higher-derivative AO integrals, XC second-derivative / grid contributions, and solution of **coupled-perturbed SCF (CPSCF)** response equations for the density’s response to nuclear displacements. Cost is much higher than one gradient—often loosely “several to many times” an energy-plus-gradient—and memory pressure rises with the response intermediates.
- **Numerical Hessian:** finite differences of **analytical gradients** (displace atoms, compute $G$, form difference quotients). Roughly $O(N_{\mathrm{atoms}})$ gradient calculations (about $2\times 3N_{\mathrm{atoms}}$ displacements for central differences, reduced by symmetry). Easier to implement and common for DFT when analytical second derivatives are unavailable or fragile; still far cheaper than finite differences of energies alone.
- **Semi-numerical / hybrid schemes:** some packages differentiate some contributions analytically and others by finite difference.

Practical notes: Hessians inherit all SCF and grid sensitivity of the gradient, amplified—loose SCF thresholds or coarse XC grids produce noisy frequencies and spurious small imaginary modes. Always confirm that the electronic structure is tightly converged before trusting a frequency job.

## Where to go next

For keyword-level SCF troubleshooting in the packages used in the group:

- [ORCA Basics and Troubleshooting — SCF Convergence Issues](/2026/01/01/ORCA-tips-and-troubleshooting.html#scf-convergence-issues)
- [Gaussian Basics and Troubleshooting — SCF Convergence Issues](/2026/01/01/Gaussian-tips-and-troubleshooting.html#scf-convergence-issues)
