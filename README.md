# Using plumed benchmark

## Options

Let's go trough the options one by one,
By inputting  `plumed benchmark --help` in the terminal we get
```
The following arguments are compulsory: 

               --plumed - ( default=plumed.dat ) colon separated path(s) to the input 
                          file(s) 
               --kernel - ( default=this ) colon separated path(s) to kernel(s) 
               --natoms - ( default=100000 ) the number of atoms to use for the 
                          simulation 
               --nsteps - ( default=2000 ) number of steps of MD to perform (-1 means 
                          forever) 
              --maxtime - ( default=-1 ) maximum number of seconds (-1 means forever) 
                --sleep - ( default=0 ) number of seconds of sleep, mimicking MD 
                          calculation 
    --atom-distribution - ( default=line ) the kind of possible atomic displacement 
                          at each step 

In addition you may use the following options: 

              --help/-h - ( default=off ) print this help 
 --domain-decomposition - ( default=off ) simulate domain decomposition, implies 
                          --shuffle 
```

 - `--plumed` is used to pass one or more input files to the benchmark
 - `--kernel` is used to pass one or more plumed kernels
 - `--natoms` decides the number of atoms
 - `--sleep`  add a sleep timer during the loop, simulating an MD
 - `--atom-distribution` chose between a few different kinds of atom distributions: "line", "cube", "sphere", "globs" and "sc"

Now let's talk more about the options that are more obscure

### Input files

With `--plumed` we can specify a series of input files, column separated, like `--plumed "plumed.dat:coord.dat:third.dat"`,
without any other options this will run the "this" kernel, see [below](#kernels) as three separated instances against the 3 input files

#### Examples


>`plumed benchmark --kernel "this:./other.so:./new.so"`

```mermaid
flowchart LR

k1[[this]]       == uses ==> f1[/plumed.dat/]
k2[[./other.so]] == uses ==> f1
k3[[./new.so]]   == uses ==> f1

```

>`plumed benchmark --kernel "this:./other.so:./new.so" --plumed "coord.dat"`

```mermaid
flowchart LR

k1[[this]]       == uses ==> f1[/coord.dat/]
k2[[./other.so]] == uses ==> f1
k3[[./new.so]]   == uses ==> f1
```


### Kernels
With `--kernel` we can specify a series of **path** to various libplumedKernel.so or "this", "this" is the kernel "bundled" with the used plumed installation.
The benchmark will run the specified kernels (like `--kernel "this:./kenel1.so:../../src/lib/install/libplumedKernel.so`) against the specified plumed file.

If you have moved the kernels in the execution path use `./kernelName.so`, since `kernelName.so` will search in the LD_LIBRARY_PATH --------verify this

#### Examples

>`plumed benchmark --plumed "plumed.dat:coord.dat:third.dat"`

```mermaid
flowchart LR

k1[[this]] == uses ==> f1[/plumed.dat/]
k1         == uses ==> f2[/coord.dat/]
k1         == uses ==> f3[/third.dat/]

```

>`plumed benchmark --kernel "./new.so" --plumed "plumed.dat:coord.dat:third.dat"`

```mermaid
flowchart LR

k1[[./new.so]] == uses ==> f1[/plumed.dat/]
k1             == uses ==> f2[/coord.dat/]
k1             == uses ==> f3[/third.dat/]

```

### Combining --kernel and --plumed

You can specify multiple kernels **and** multiple input files together.
In this case the number of two will be equal: the kernel in first position will be used with the file in first position, the kernel in second position will be used with the file in second position and so on.

#### Examples

>`plumed benchmark --kernel "this:./other.so:./new.so" --plumed "plumed.dat:coord.dat:third.dat"`

```mermaid
flowchart LR

k1[[this]]       == uses ==> f1[/plumed.dat/]
k2[[./other.so]] == uses ==> f2[/coord.dat/]
k3[[./new.so]]   == uses ==> f3[/third.dat/]

```

>`plumed benchmark --kernel "this:this:./new.so" --plumed "plumed.dat:coord.dat:coord.dat"`

```mermaid
flowchart LR

k1[[this]]       == uses ==> f1[/plumed.dat/]
k1               == uses ==> f2[/coord.dat/]
k3[[./new.so]]   == uses ==> f2

```


### Atom Distribution
When specifying `--atom-distribution` you can choose one among "line", "cube", "sphere", "globs" and "sc"

The atom distribution can be divided into two categories:
 - Random distributions: the atoms are distributed uniformly with a density of 1 atom per nm^3 (the total volume is equal to the number of atoms). In these distributions, the atoms are randomly displaced in the available volume at each step, so they are **not** compatible with a neighbor list that updates every n>1 steps :
    - "sphere" the atoms are uniformly distributed in a sphere
    - "cube" the atoms are uniformly distributed in a cube
    - "globs" the atoms are uniformly distributed in two spheres
 - standard: the base is that each atom is in a grid with each atom at a distance of 1 nm  (so that the atom radius is 0.5nm).
    - "line" the atoms are displaced in a line at 1nm from each other, then at each step they are displaced in a sphere or radius 0.5nm around the initial position
    - "sc" the atoms are displaced in a simple cubic crystal, as the time of writing this there are no extra movements, the cube is always a perfect cube (1,8,27,64...) but it is filled bottom-up with the number of atoms requested 

#### Examples
I all the examples here the atoms are visualize with radius 0.5nm.

In the cube, sphere and globs example the atom 0 is higligted in red to show that is randomly displaced at each step
#### cube
![](cube.gif)
#### sphere
![](sphere.gif)
#### globs
![](globs.gif)
##### line
The fisrt 50 frames of a line with 10 atoms, the atoms are coloreb by their index to underline that they move around their initial position
![](line.gif)
##### sc
How the simple cubic system is made with increasig number of atoms
![](sc.gif)