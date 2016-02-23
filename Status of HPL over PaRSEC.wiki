 * V0: First version numerically correct with dynamic creation of workspaces, (9f2b7ee1366d+ 5304+ zenati)
 ** In zenati branch, zgetrf_fusion.jdf
 ** No check have been done with GPUs
 ** Panel:
 *** Uses only one preallocated workspace for the U during panel factorization
 *** Local reduction is done with a binary tree with three elements reduced at once
 *** Distributed reduction is done through Bruck's algorithm
 *** NO immediates are used in the panel
 ** Update:
 *** Swap exchanges are done with a full tile
 *** GEMM and SWAP are done in column major
 *** SWAP_COLLECT are done in parallel
 *** SWAP_RECV and GEMM are not merged together

 * V1: (283d459cd4be 6904 zenati)
 ** In zenati branch, zgetrf_fusion.jdf
 ** V0 +:
 *** Update: Extraction done in Row Major, but TRSM/GEMM are still in column major

 * TODO:
 ** Panel:
 *** Cleaner Bruck algorithm (take example on getrf_qrf from qr_lu branch)
 *** Add immediate task to improve reactivity on the panel
 *** UMAT could be replaced by WRITE data
 ** Update:
 *** Perfom all TRSM/GEMM in row major to improve performance off swap in case we want to use GPUs
 *** Merge SWAP_RECV and GEMM tasks to reduce the task number
 *** Reduce the exchanged volume during SWAP to its minimum

