### diskmat

diskmat is a simple, header-only tool for providing a generic front-end to matrices that may not fit into memory.

It consists of two structures: `diskmat::DiskMat`, which uses mmap to store the matrix, and `diskmat::PolymorphicMat`,
which defaults to in-memory if the space consumed is below a threshold and falls back to disk-based management afterward.

### Example

```c++
#include <diskmat/diskmat.h>

int main() {
    diskmat::PolymorphicMat<float> pmat(100000, 1000000);
    // pmat is now a 4 terabyte matrix on disk
    // Initializing
    std::uniform_real_distribution<float> urd;
    for(size_t i = 0; i < (~pmat).rows(); ++i) {
        auto r = row(~pmat, i);
        r = blaze::generate(1000000, [&urd](auto x){std::mt19937_64 mt(x); return urd(mt);});
    }
    // Next, we can perform our math over the mmap'd data transparently
    // This includes reductions:
    blaze::DynamicVector<float> meanvec = trans(blaze::mean<blaze::columnwise>(~pmat));
    // Matrix multiplications, inc. via BLAS bindings
    // blaze::DynamicVector<float> mvp = (~pmat) * meanvec;
    // and most other techniques
    diskmat::PolymorphicMat<float> smallermat(1000, 40000); // This is allocated on the heap
}

```

This can be simplified, as desired, by simply getting a reference to the base matrix (`auto &matr = ~pmat`).
