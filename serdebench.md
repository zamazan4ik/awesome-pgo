PGO benchmarks for different serialization/deserialization libraries with https://github.com/llogiq/serdebench benchsuite

## Test environment

* Macbook M1 Pro (6+2 CPU, 16 Gib RAM, 512 SSD)
* macOS 13.6 Ventura
* Rustc version: `rustc 1.77.0-nightly (d78329b92 2024-01-13)`
* serdebench version: `4cd6b3404758dff633615029b790f32859bf4e0c` commit from the `master` branch

## Benchmark setup

Release results are collected with `cargo +nightly bench`. PGO optimization is done with [cargo-pgo](https://github.com/Kobzol/cargo-pgo). PGO instrumentation is done with `cargo +nightly pgo bench`. PGO optimization is done with `cargo +nightly pgo optimize bench`.

## Results

I got the following results:

* Release: https://gist.github.com/zamazan4ik/b030a926ce164aedc46fdf4d5ba0739a
* PGO optimized (compared to Release): https://gist.github.com/zamazan4ik/a9d4123ed9d1b6820d0591431be9dbfc
* (just for reference) PGO instrumented: https://gist.github.com/zamazan4ik/1258945a185d0d72299cd1913332d0fa

According to the tests, PGO helps with optimizing performance in many cases. However, in the benchmarks, there are regressions with PGO in some rare scenarios.
