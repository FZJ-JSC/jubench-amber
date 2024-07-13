# Amber

## About

Assisted Model Building with Energy Refinement (Amber) is a molecular dynamics code for biomolecular simulations.
This repository compiles Amber for [GPUs](https://doi.org/10.1021/ct400314y) and runs the benchmark. 
The base code is written in Fortran and uses OpenMP, MPI to parallelize. CUDA C++ is used for the GPU acceleration.   

## Quick start:

```
module load JUBE
jube run ./benchmark/jube/amber.yaml
jube result -a ./benchmark/jube/bench_run
```

You might need to adapt the `.yaml` according to your system configuration.

The JUBE script takes care of execution of the benchmark. It also configures, builds, and installs the
program in the process.

## Source

Amber is a proprietary application, the license and the source code need to be acquired by benchmark users. 
The Amber Benchmark Suite can be downloaded from the [Amber website](https://ambermd.org/Amber20_Benchmark_Suite.tar.gz).

The required Amber version is 22 and needs to be patched up to `update.2`. AmberTools22 should be updated to `update.4`

```
cd  $AMBERSRCPATH/ 
./update_amber --update-to=AmberTools.4,Amber.2 
```

### JUBE

The benchmark is deeply integrated with JUBE. JUBE assumes the Amber source file tarball to be place at `$BENCHMARKHOME/src/amber-amber22-with-patches.tar.gz`. You can change the source tar.gz filename and its path using `AMBERSRCFILE` and `AMBERSRCPATH` in the .yaml file. The tar file of the Amber Benchmark Suite is automatically downloaded and extracted by JUBE in the `unpack` step. The `unpack` step also takes care of the update. See below.

## Building

The benchmark for Amber is for the GPU version only. Amber needs the following software to build:

1. GCC compiler
2. CMake
3. MPI or NCCL (or similar for better performance)
4. Flex
5. CUDA

Once the dependencies are in place, the code can be built and installed with the following commands:

```
cd Amber
mkdir build && cd build
cmake ../ -DCMAKE_INSTALL_PREFIX=/your/install/path  -DBUILD_PYTHON=FALSE -DCOMPILER=GNU -DMPI=TRUE -DCUDA=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=FALSE -DNCCL=TRUE 

make -j16 install
```

This will install the Amber executables to `/your/install/path`.

Building takes quite a little while.

### JUBE

Using JUBE, the steps `config` and `compile` configure and build Amber, respectively. See below.

## Execution

The executable of the benchmark is `pmemd.cuda.MPI`, the input file is `mdinOPT.GPU`.

### Rules

- Changing the input file `mdinOPT.GPU` is not within scope.
- Floating point optimization must not be set to `--ffast-math` or similar.
- Disabling ECC is not allowed.

### Command Line

On the command line the benchmark can be called with:

```
[mpiexec] ${AMBERHOME}/bin/pmemd.cuda.MPI -O -i mdinOPT.GPU -p STMV.prmtop -c STMV.inpcrd -o md.opt.out -x md.x -r md.r -e md.e
```

### JUBE

Using JUBE, the step `submit` takes care of submitting the benchmark – including the proper options – to the batch queue.

To execute the benchmark with JUBE -- including all aforementioned steps -- use:

```
jube run ./benchmark/jube/amber.yaml
```

## Verification

The benchmark should run successfully without any errors being reported. The observables averaged over 10 steps in the `md.opt.out` should look like following: 

```
      A V E R A G E S   O V E R      10 S T E P S


 NSTEP =    10000   TIME(PS) =    5161.003  TEMP(K) =   311.21  PRESS =     0.0
 Etot   =  -2596988.5718  EKtot   =    687463.6188  EPtot      =  -3284452.1905
 BOND   =     41847.4843  ANGLE   =     96261.3868  DIHED      =    122381.3675
 1-4 NB =     40307.5356  1-4 EEL =    215443.8500  VDWAALS    =    330808.6552
 EELEC  =  -4131502.4698  EHBOND  =         0.0000  RESTRAINT  =         0.0000
 ------------------------------------------------------------------------------
```

In particular, the total energy `Etot` should be the same. 

### JUBE

Using JUBE, the step `submit` takes care verification. `submit` can finish without errors only if it passes the verification.

## Results

The benchmark prints a runtime (`elapsed_time(s)`) and number of GPUs used for calculations

### Command Line

To get results (`elapsed_time(s)`), execute this command:
```
user@jwlogin24 work]$ grep "Elapsed(s)" md.opt.out | tail -1| awk {'print $2,$3,$4'}
Elapsed(s) = 28.29
```

### JUBE

To see the benchmark results with JUBE, execute this command:

```
jube analyse ./benchmark/jube/bench_run
jube result  ./benchmark/jube/bench_run

```

The output should look like this:

```
[user@jwlogin24 amber-gpu]$ jube result -a benchmark/jube/bench_run --id 30
nodes,ngpu,elapsed_time_last,verified
1,4,28.58,true
```


## Baseline

The baseline configuration must be chosen such that the elapsed time is less than or equal to 30 s.

Nnode | NGPU  | elapsed_time(s)
----- | ----- | ---          
1     | 4     | 28.29
