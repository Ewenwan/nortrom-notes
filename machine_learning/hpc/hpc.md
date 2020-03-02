[toc]

[main page](../../entry.md)

# matrix computation

## blas

* 优化方案
    * optimizing by blocking and computing kernels
        * openblas
    * optimizing by matrix multiplication algorithms
        * winograd 
    * comparison
        * There exist asymptotically faster matrix multiplication algorithms, eg the Strassen algorithm or the Coppersmith-Winograd algorithm which **have a slightly faster rate than O(n^3)**. However, they are **generally not cache aware and ignore locality** - meaning that data continually needs to be shunted around in memory, so for most **modern architectures the overall algorithm is actually slower** than an optimized block matrix multiplication algorithm.
        * Wikipedia notes that the Strassen algorithm may provide speedups on a single core CPU for matrix sizes greater than several thousand, however the **speedup is likely to be around 10% or so**, and the developers of BLAS probably don't consider it worthwhile for this rare case (saying that, this paper from 1996 claims a speed increase of around 10% over DGEMM for n above about 200 -- though I don't know how out of date that is). The Coppersmith-Winograd algorithm, on the other hand, "only provides an advantage for matrices so large that they cannot be processed by modern hardware".
        * refer
            * [Matrix multiplication time complexity](http://stackoverflow.com/questions/17716565/matrix-multiplication-time-complexity-in-matlab)
            * [math-matrix-computation-complexity](../../math/matrix.md#matrix-computation-complexity)
* referenced papers
    * [High-Performance Implementation of the Level-3 BLAS](http://www.cs.utexas.edu/users/flame/pubs/flawn20.pdf)