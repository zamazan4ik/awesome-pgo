Here I we have some PGO benchmarks for httpd.

## Test environment

* Fedora 38
* Linux kernel 6.4.15
* AMD Ryzen 9 5900x
* 48 Gib RAM
* SSD Samsung 980 Pro 2 Tib
* Compiler - Clang 16 (`16.0.6` from Fedora repositories )
* httpd version: 2.4.57
* Other details: Turbo boost is disabled

## Benchmark setup

An idea of how to do the benchmark I got from the Rathole benchmark [guide](https://github.com/rapiz1/rathole/blob/main/docs/benchmark.md#http) for HTTP load. So I implemented the same benchmark for Httpd: `Benchmark tool -> httpd (own binary) -> Nginx (system binary)`.

As a benchmark tool, I use [Nighthawk](https://github.com/envoyproxy/nighthawk) with this command line: `taskset -c 15-23 ./nighthawk_client --rps 2000 --duration 120 --connections 2 --concurrency auto --prefetch-connections -v info http://127.0.0.1:8080`.

Httpd was tested with this command line: `taskset -c 0 httpd -X`. `httpd.conf` content is here: https://gist.github.com/zamazan4ik/cdae9e04e29651e55790ed2b7fab51a0 , all other configuration files are in a default state. I use single-core benchmark setup since I want to load Httpd to 100% on 1 core so I can easily measure the maximum throughput and get the difference in max RPS between Release and PGO builds.

`taskset` is used everywhere just to reduce OS scheduling noise during the measurements. All measurements are done multiple times (they are almost the same), on the same hardware/software with the same background load (as much as I can guarantee) and the results are compared between runs - they are reproducible with almost no variation between runs.

## Optimization steps

Release Httpd is built with `CC=clang ./configure --with-included-apr && make` command.

Httpd with PGO is built in the following steps:
* Build Instrumented Httpd with `CC=clang CFLAGS="-g -O2 -fprofile-instr-generate=httpd_%p_%m.profraw" LDFLAGS="-fprofile-instr-generate=httpd_%p_%m.profraw" ./configure --with-included-apr && make`.
* Run the instrumented Httpd to collect the profile. As a training workload, I used completely the same workload as used for the benchmark purposes.
* Prepare the profile for Clang with `llvm-profdata merge -output=httpd.profdata httpd_*.profraw`
* Compile Httpd once again with the collected profiles with `CC=clang CFLAGS="-g -O2 -fprofile-instr-use=httpd.profdata -Wno-error -Wno-profile-instr-out-of-date" ./configure --with-included-apr && make`.

## Results

In short, I get the following RPS results from Nighthawk:
* Release: ~10800 RPS
* Release + PGO: ~11500 RPS
* Instrumented (just for reference): ~10400 RPS

More detailed reports from Nighthawk are available here:
* Release: https://gist.github.com/zamazan4ik/0703010060ed566fae8073f3f49b88f9
* Release + PGO: https://gist.github.com/zamazan4ik/332130500c8452546684b682c6c1cf42
* Instrumented (just for reference): https://gist.github.com/zamazan4ik/024937c6ff4fb12cbb4847a41140cac4

According to the tests, PGO helps with optimizing httpd's performance from the throughput perspective at least in the tested scenario.
