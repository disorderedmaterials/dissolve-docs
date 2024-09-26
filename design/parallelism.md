# Dissolve Parallel Tech Spec

## Introduction

### Purpose
This document lays out the current and future technical requirements for the parallel architecture of the Dissolve project.

### Background

Dissolve is an atomistic simulation code for the interrogation and analysis of neutron scattering data. Since the simulation boxes (collections of atoms) that it produces must be representative of the physical systems measured experimentally, particle numbers of 1 million and beyond must be treatable. This is not practical on a single-core machine, or even a modestly powerful workstation, and so suitable parallel technology must be implemented.

## Reason for Parallel Operation

At present Dissolve's core operation is to generate (ensembles of) atomic coordinates for a given system that are consistent with measured experimental data. It achieves this through definition of a reference description of the basic (approximate) interactions occurring between atoms, and then iteratively modifies this potential while evolving the system using standard techniques such as Monte Carlo or Molecular Dynamics, or a combination of both. Once a suitable 'fit' to the experimental data has been achieved, further (long) simulations are performed in order to calculate structural properties of interest to the end user. The basic workflow is thus:

REF -> ( SIM -> REFINE ) -> SIM

The speed at which the program converges on the best solution is heavily dependent on the types of and number molecules present in the system, the strength of the interactions between atoms, the complexity of the system, and the completeness of the experimental data driving the refinement process. As such, the number of refinement iterations required in the main loop cannot easily be predicted, although thousands of iterations for small molecular liquids up to tens of thousands for self-assembled systems such as micelles are common. For multi-million atom systems with higher complexity the number of required refinement iterations will likely increase further.

In the simulation itself, performed at least once on every refinement iteration, simple evolution mechanisms such as Molecular Dynamics scale as O(N(N+1)/2) (per timestep), which demands considerable computational effort for millions of atoms and beyond, and which might be run for 50 - 100 steps or more on each pass. However, these may be parlalelised with relatively little effort as the mutation to every particle in the system is calculated in one single calculation that itself does not alter the state. Monte Carlo offers a greater challenge to efficient parallelisation, where each particle or molecule is moved individually, and the energy change is used to guide whether the move is accepted or not. Since moving a particle or molecule affects the energy of the surrounding particles and molecules (within the cutoff distance) care must be taken to ensure that any parallel scheme trials simultaneous moves that are independent of each other.

### Current Implementation

At present (v0.7.0) Dissolve utilises the widely-available OpenMPI standard to achieve parallelism across multiple CPU cores over multiple physical processing enclosures. Each slave process maintains its own copy of all simulation data (replicated data strategy) leading to high memory requirements.

Dissolve's original design was to allow partitioning of processors into sub-groups in order to permit:

1. Fine-grained control over partitioning of workload in order to achieve balance between processing time and communication overhead.
2. Forking of the main loop to allow simultaneous calculations targetting different atomic configurations or modules

As a consequence of 2), each main loop iteration requires a broadcast of all simulation data between processes to ensure that each process has up-to-date information, forcing the requirement for T::broadcast() and T::equality() (enforced by inheriting from GenericItemBase) for each class object that needs to be distributed.

Energy and force calculations are relatively straightforward to parallelise, simply involving striding through elements of arrays according to the number of available processors (or groups of processors) and followed by a sum of the final value(s) over the relevant MPI communicator. Monte Carlo techniques are parallelised through the use of a MoleculeDistributor class, which partitions the system contents in such a way as to allow parallel trialling of particle / molecule moves independently, based on the simulation cutoff distance for energy calculation. Point 1) above is important here, as the number of regions that the system can be partiioned in to is not directly predictable, and so requires adaptive partitioning of the available processes.

## Requirements

### Code

- A single codebase must be maintained for serial, GUI and parallel versions, with parallelism built-in when requested through the build generator (CMake).
- The GUI version will remain as a serial code, as its scope covers only the set-up and analysis of simulations, and the running of small tutorial systems that do not require to be run in parallel.

### General Approach

- The ability to partition available processes into task-groups for certain algorithms is beneficial.
- Scaling over CPU- and GPU-based multi-core systems should be considered.
- Replicated data strategies over CPU cores should not be employed - such approaches are not feasible for systems of 1 million particles and beyond.

### System Architecture

- Parallelisation of the Linux non-GUI version is essential. 
- Parallelisation of the OSX non-GUI version is desirable.
- Parallelisation of the Windows non-GUI version is beneficial, but not essential. 
- Parallelisation of the GUI version is not essential.
- Any parallel scheme so chosen should be flexible enough to permit the adoption and paralleilsation of future algorithms into the code.
