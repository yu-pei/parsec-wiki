== Install required softwares ==
To compile PaRSEC/DPLASMA in Intel Xeon Phi,you will need the following software of Intel Xeon Phi version:
* a BLAS library excitable on Intel Xeon Phi, usually is MKL.
* [[http://icl.cs.utk.edu/plasma/software/index.html |PLASMA]]. To get a Intel Xeon Phi version, please use x86_64-k1om-linux-gcc or icc with flags -mmic as C compiler, and x86_64-k1om-linux-gfortran or ifort with flags -mmic as Fortran compiler . Since PLASMA requires LAPACK and LAPACKE, make sure you LAPACK and LAPACKE are also compiled with the above compilers.