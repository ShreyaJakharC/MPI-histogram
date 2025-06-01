# MPI Histogram

## Overview

This repository contains  implementation of a parallel histogram computation using MPI. The goal of the project was to:

1. Generate a large array of random floating-point values (in the range [0, 20.0)).  
2. Partition the data evenly across MPI processes.  
3. Have each process compute a local histogram (number of items falling into each bin).  
4. Optionally verify correctness by comparing against a sequential (reference) histogram.  
5. Measure parallel runtime for varying numbers of processes, data-set sizes, and bin counts, then compute speedup and efficiency statistics.



## Tech Stack

- **Language:** C  
- **Parallel Framework:**  
  - MPI (tested with Open MPI and MPICH)  
- **Compiler & Build Tools:**  
  - `mpicc` (Open MPI v4.1.1 or MPICH v4.x)  
  - `Makefile` (optional but recommended)  
- **Execution:**  
  - `mpirun` or `mpiexec` (to launch multiple ranks)  
- **Timing Routine:**  
  - `MPI_Wtime()` for wall-clock measurement  

## Features

- **Random Data Generation (Rank 0):**  
  - Rank 0 allocates an array of `num_items` floats, fills it with uniform random values in [0, 20.0), then scatters equally to all ranks.

- **Parallel Binning:**  
  - Each process receives `local_num_items = num_items / num_ranks` floats.  
  - Computes its local histogram by dividing `[0, 20.0)` into `num_bins` equal intervals and incrementing counts accordingly.  
  - Ensures that if a value lands exactly on an upper bound, it is placed in the last bin.

- **MPI Communication:**  
  - `MPI_Scatter` distributes chunks of the data array from rank 0 to all ranks.  
  - `MPI_Reduce` (with `MPI_MAX`) collects the maximum elapsed time across all processes, reported by rank 0.

- **Correctness Check (Rank 0):**  
  - After the parallel timing, rank 0 reconstructs the histogram sequentially (using the same data array) and compares each bin to the final result.  
  - Any mismatch in bin counts is printed for debugging.

- **Configurable Parameters:**  
  1. **Number of Data Items (`num_items`):**  
     - Acceptable values: 1 … 10,000,000,000 (practically limited by memory).  
  2. **Number of Bins (`num_bins`):**  
     - Any positive integer (e.g., 4, 10, 100).  
  3. **Number of MPI Ranks:**  
     - Specified at runtime via `mpirun -np <ranks>`.

- **Error Handling:**  
  - If `num_items < num_ranks`, the program aborts to avoid a divide-by-zero or zero items per rank.  
  - If memory allocation fails on any process, the program calls `MPI_Abort`.


## Quick Start

Below are step-by-step instructions to compile and run the lab on a Unix/Linux cluster.

### Prerequisites

- A working MPI installation (Open MPI or MPICH).  
- A C compiler with MPI support (`mpicc`).  
- Sufficient memory to hold the data array on rank 0 (e.g., if you choose `num_items = 10 million`, that’s ~40 MB for floats).

### 1. Clone or Download the Repository

```bash
git clone https://github.com/<your-username>/parallel-computing-lab1.git
cd parallel-computing-lab1
