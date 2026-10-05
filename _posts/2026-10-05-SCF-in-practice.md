---
title: SCF in Practice
image: images/post-tutorial.jpg
author: rlaplaza
tags: tutorial, computational-chemistry, scf
---

# SCF in Practice

This post is a short, package-agnostic primer on the self-consistent field (SCF) procedure as used in routine Hartree–Fock (HF) and Kohn–Sham DFT calculations. The goal is basic intuition: what the loop does, what drives the cost, how integrals and XC grids enter, how jobs are started and converged, and what an analytical gradient costs relative to the energy. For program-specific keywords and failure recipes, see the [ORCA](/2026/01/01/ORCA-tips-and-troubleshooting.html) and [Gaussian](/2026/01/01/Gaussian-tips-and-troubleshooting.html) guides.

## What the SCF loop is

In HF and DFT, the electronic energy depends on the occupied molecular orbitals (or equivalently the density matrix). Those orbitals are eigenfunctions of an effective one-electron Hamiltonian—the Fock matrix in HF, the Kohn–Sham (KS) matrix in DFT—which itself depends on the density. The SCF procedure closes that loop:

1. Start from an initial density (or orbital guess).
2. Build the Fock/KS matrix from that density.
3. Diagonalize (or otherwise update orbitals) to get a new density.
4. Repeat until the density, energy, and orbital gradient stop changing within a chosen threshold.

Almost every downstream result—geometry steps, frequencies, and most molecular properties—assumes a converged SCF. A “finished” optimization with a poorly converged electronic structure is not trustworthy.

## Cost and basis-set scaling

Let \(N\) be the number of basis functions. The formal bottleneck for conventional HF is the two-electron repulsion integrals, which scale as \(O(N^4)\). In practice, integral screening and locality cut this down for large molecules, but the cost still rises steeply when you enlarge the basis (more functions per atom) or the system size.

A few useful rules of thumb:

- Doubling the basis-set quality (e.g. double-ζ → triple-ζ) increases \(N\) and therefore each Fock build, often by more than a factor of two.
- Hybrid DFT pays for Coulomb-like terms plus a fraction of exact (HF-like) exchange; exact exchange is usually the expensive piece compared with a pure GGA.
- A larger basis also makes a bad initial guess more painful: each cycle is heavier, and you may need more cycles before DIIS or related accelerators settle.

Memory and disk pressure grow with \(N\) as well, which is why how integrals are handled matters in practice.

## Numerical XC grids

In KS DFT, the exchange–correlation (XC) energy and potential are not evaluated as analytic AO integrals the way Coulomb and exact exchange are. Instead, the density (and its derivatives for GGAs and beyond) is sampled on a real-space quadrature grid—typically a union of atom-centered grids with radial and angular points—and the XC contribution is integrated numerically into the KS matrix **every SCF cycle**.

Practical consequences:

- Grid work scales roughly with (number of grid points) × (basis functions that have appreciable weight at those points). Denser grids raise both cost and the numerical noise floor of the energy.
- Grids are a DFT-specific cost channel, separate from whether AO integrals are stored on disk or recomputed.
- Coarser grids are often acceptable early in the SCF; the final energy—and especially gradients—usually need a denser grid so that numerical noise does not masquerade as forces or stall a geometry optimization.
- If two “identical” DFT jobs differ only in grid settings, small energy differences are expected; do not over-interpret microhartree changes across grid changes.

## Integrals in practice: conventional vs direct

Two classical strategies exist for the AO two-electron integrals:

- **Conventional (in-disk):** compute integrals once, store them on disk, and read them back each SCF cycle. Throughput is then limited by disk space and I/O bandwidth. This can be attractive for small jobs with many SCF cycles when scratch storage is fast and large enough.
- **Direct:** recompute integrals (or screened batches of them) every iteration instead of storing the full set. The trade-off shifts toward CPU (and memory for buffers), which is usually preferable once \(N\) is large enough that the integral file would be huge or the I/O would dominate.

Modern defaults lean toward direct (or semi-direct) SCF for anything beyond tiny calculations. Whatever the mode, the integral (and grid) accuracy must stay tighter than the SCF convergence threshold: you cannot converge the density below the noise in the Fock/KS matrix.

## Initial guesses

The first density decides how hard the early SCF cycles will be. Common options:

- **Core Hamiltonian / minimal Hückel-like guesses** — cheap and often crude; fine for simple closed-shell organics, fragile for metals or open shells.
- **Superposition of atomic densities (SAD) and related atomic guesses** — usually a much better starting density for molecules.
- **Reading orbitals from a previous calculation** — often the best choice when the geometry or method changed only slightly (optimization steps, basis upgrades, or restarting a failed job).
- **Open-shell tricks** — mixing HOMO/LUMO character or breaking α/β symmetry can help land on a broken-symmetry or diradicaloid solution instead of collapsing to a closed-shell state.

A good guess often saves more wall time than raising the maximum number of cycles. A bad guess can send the SCF into oscillation between qualitatively different electronic structures.

## Getting to convergence

Plain Roothaan–Hall iteration (build Fock/KS → diagonalize → repeat) frequently oscillates: each new density overcorrects the previous one. Production codes therefore accelerate and stabilize the update:

- **Damping and level shifting** — mix old and new densities, or shift virtual levels, to damp oscillations early on.
- **DIIS (Direct Inversion in the Iterative Subspace)** — store recent Fock/KS matrices and error vectors (e.g. the orbital gradient) and extrapolate a linear combination that drives the error toward zero. This is the workhorse accelerator for routine SCF.
- **SOSCF / second-order orbital optimization** — treat orbital rotations more like a Newton step using an approximate orbital Hessian. Useful when DIIS stalls, especially for difficult open-shell or near-degenerate cases.
- **Quadratic or trust-region fallbacks** — last-resort algorithms in some packages when standard DIIS/SOSCF paths fail.

A practical mental order of operations: start from a sensible guess, let DIIS (the usual default) work, add damping or level shift if the energy oscillates, and only then escalate to second-order or package-specific robust SCF modes. Tightening thresholds without fixing the electronic-structure problem mostly burns cycles.

## Analytical SCF gradients

Once the HF or DFT density is converged, nuclear gradients with respect to atomic coordinates can be evaluated **analytically**. That is much cheaper than numerical differentiation, which would require on the order of \(3N_\mathrm{atoms}\) extra energy calculations (or twice that for central differences).

An analytical SCF gradient is typically on the order of **one extra SCF-like step** after convergence. The main pieces are:

- derivatives of the AO integrals with respect to nuclear positions,
- XC contributions on the grid, including grid-weight / grid-center derivative terms in DFT,
- Pulay terms that appear because the basis functions move with the nuclei (basis-set incompleteness / atom-centered basis effects).

Cost still tracks basis-set size in the same broad way as the energy: larger \(N\) means heavier derivative-integral and XC work. Gradients are also more sensitive to a loose SCF or a coarse XC grid than a rough single-point energy is—noisy forces slow or mislead geometry optimizations even when the energy looks “almost” converged.

## Where to go next

For keyword-level SCF troubleshooting in the packages used in the group:

- [ORCA Basics and Troubleshooting — SCF Convergence Issues](/2026/01/01/ORCA-tips-and-troubleshooting.html#scf-convergence-issues)
- [Gaussian Basics and Troubleshooting — SCF Convergence Issues](/2026/01/01/Gaussian-tips-and-troubleshooting.html#scf-convergence-issues)
