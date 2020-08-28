# In-place Parallel Super Scalar Radix Sort (IPS²Ra)

This is the implementation of the algorithm IPS²Ra presented in the publication [In-place Parallel Super Scalar Samplesort (IPS⁴o)](https://arxiv.org/abs/1705.02257) (todo update),
which contains an in-depth description of its inner workings, as well as an extensive experimental performance evaluation.
Here's the abstract:

> We present a sorting algorithm that works in-place, executes in parallel, is
> cache-efficient, avoids branch-mispredictions, and performs work O(n log n) for
> arbitrary inputs with high probability. The main algorithmic contributions are
> new ways to make distribution-based algorithms in-place: On the practical side,
> by using coarse-grained block-based permutations, and on the theoretical side,
> we show how to eliminate the recursion stack. Extensive experiments show that
> our algorithm IPS⁴o scales well on a variety of multi-core machines. We
> outperform our closest in-place competitor by a factor of up to 3. Even as
> a sequential algorithm, we are up to 1.5 times faster than the closest
> sequential competitor, BlockQuicksort.

The radix sort algorithm IPS²Ra is an adaption of the samplesort algorithm IPS⁴o. 
The implementation of IPS⁴o is also available on [GitHub](https://github.com/ips4o/ips4o).
In the sequential case, IPS²Ra outperforms IPS⁴o for many input distributions and data types. However, IPS⁴o may be faster for very skewed key distributions. In the parallel case, IPS⁴o outperforms IPS²Ra most of the time.

An initial version of IPS⁴o has been described in our [publication](https://drops.dagstuhl.de/opus/volltexte/2017/7854/pdf/LIPIcs-ESA-2017-9.pdf) on the 25th Annual European Symposium on Algorithms.

## Usage

```C++
#include "ips2ra.hpp"

// sort sequentially
ips2ra::sort(begin, end[, comparator]);

// sort in parallel (uses OpenMP if available, std::thread otherwise)
ips2ra::parallel::sort(begin, end);
```

IPS²Ra provides a CMake library for simple usage:

```CMake
add_subdirectory(<path-to-this-folder>)
target_link_libraries(<your-target> PRIVATE ips2ra)
```

The parallel version of IPS²Ra requires 16-byte atomic compare-and-exchange instructions to run the fastest.
Most CPUs and compilers support 16-byte compare-and-exchange instructions nowadays.
If the CPU in question does so, IPS²Ra uses 16-byte compare-and-exchange instructions when you set your CPU correctly (e.g., `-march=native`) or when you enable the instructions explicitly (`-mcx16`).
In this case, you also have to link against GCC's libatomic (`-latomic`).
Otherwise, we emulate some 16-byte compare-and-exchange instructions with locks which may slightly mitigate the performance of IPS²Ra.

If you use the CMake example shown above, we automatically optimize IPS²Ra for the native CPU (e.g., `-march=native`).
You can disable the CMake property `IPS2RA_OPTIMIZE_FOR_NATIVE` to avoid native optimization and you can enable the CMake property `IPS2RA_USE_MCX16` if you compile with GCC or Clang to enable 16-byte compare-and-exchange instructions explicitly.

IPS²Ra uses C++ threads if not specified otherwise.
If you prefer OpenMP threads, you need to enable OpenMP threads, e.g., enable the CMake property `IPS2RA_USE_OPENMP` or add OpenMP to your target.
If you enable the CMake property `DISABLE_IPS2RA_PARALLEL`, most of the parallel code will not be compiled and no parallel libraries will be linked.
Otherwise, CMake automatically enables C++ threads (e.g., `-pthread`) and links against TBB and GCC's libatomic. (Only when you compile your code for 16-byte compare-and-exchange instructions you need libatomic.)
Thus, you need the Thread Building Blocks (TBB) library to compile and execute the parallel version of IPS²Ra.
We search for TBB with `find_package(TBB REQUIRED)`.
If you want to execute IPS²Ra in parallel but your TBB library is not accessible via `find_package(TBB REQUIRED)`, you can still compile IPS²Ra with parallel support. 
Just enable the CMake property `DISABLE_IPS2RA_PARALLEL`, enable C++ threads for your own target and link your own target against your TBB library (and also link your target against libatomic if you want 16-byte atomic compare-and-exchange instruction support).

If you do not set a CMake build type, we use the build type `Release` which disables debugging (e.g., `-DNDEBUG`) and enables optimizations (e.g., `-O3`).

Currently, the code does not compile on Windows.

## Licensing

IPS²Ra is free software provided under the BSD 2-Clause License described in the [LICENSE file](LICENSE). If you use IPS²Ra in an academic setting please cite the publication [In-place Parallel Super Scalar Samplesort (IPS⁴o)](https://arxiv.org/abs/1705.02257) (todo update) using the BibTeX entry

(todo update)
```bibtex 
@InProceedings{axtmann2017,
  author =	{Michael Axtmann and
                Sascha Witt and
                Daniel Ferizovic and
                Peter Sanders},
  title =	{{In-Place Parallel Super Scalar Samplesort (IPSSSSo)}},
  booktitle =	{25th Annual European Symposium on Algorithms (ESA 2017)},
  pages =	{9:1--9:14},
  series =	{Leibniz International Proceedings in Informatics (LIPIcs)},
  year =	{2017},
  volume =	{87},
  publisher =	{Schloss Dagstuhl--Leibniz-Zentrum fuer Informatik},
  doi =		{10.4230/LIPIcs.ESA.2017.9},
}
```
