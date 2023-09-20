Here I we have some PGO benchmarks for Nginx.

## Test environment

* Fedora 38
* Linux kernel 6.4.15
* AMD Ryzen 9 5900x
* 48 Gib RAM
* SSD Samsung 980 Pro 2 Tib
* Compiler - Clang 16 (`16.0.6` from Fedora repositories )
* Nginx version: 1.25.2
* Other details: Turbo boost is disabled

## Benchmark setup

An idea of how to do the benchmark I got from the Rathole benchmark [guide](https://github.com/rapiz1/rathole/blob/main/docs/benchmark.md#http) for HTTP load. So I implemented the same benchmark for Nginx: `Benchmark tool -> Nginx (own binary) -> Nginx (system binary)`.

As a benchmark tool, I use [Nighthawk](https://github.com/envoyproxy/nighthawk) with this command line: `taskset -c 4 ./nighthawk_client --rps 15000 --duration 300 --connections 4 --concurrency auto --prefetch-connections -v info http://127.0.0.1:8080`.

Nginx was tested with this command line: `taskset -c 0 ./nginx`. `nginx.conf` content is here: https://pastebin.com/0yv6NJFx . I use `worker_processes 1` in the config file since I want to load Nginx to 100% on 1 core so I can easily measure the maximum throughput and get the difference in max RPS between Release and PGO builds.

`taskset` is used everywhere just to reduce OS scheduling noise during the measurements. All measurements are done multiple times (they are almost the same), on the same hardware/software with the same background load (as much as I can guarantee).

## Optimization steps

Release Nginx is built with `CC=clang CXX=clang++ ./configure --with-cc=/usr/bin/clang --with-cpp=/usr/bin/clang++ --with-cc-opt="-O3" && make` command.

Nginx PGO is built in the following steps:
* Build Instrumented Nginx with `CC=clang CXX=clang++ ./configure --with-cc=/usr/bin/clang --with-cpp=/usr/bin/clang++ --with-cc-opt="-O3 -fprofile-instr-generate=/home/zamazan4ik/open_source/bench_nginx/profiles/nginx_%m_%p.profraw" --with-ld-opt="-fprofile-instr-generate=/home/zamazan4ik/open_source/bench_nginx/profiles/nginx_%m_%p.profraw" && make`.
* Run the instrumented Nginx to collect the profile. As a training workload, I used completely the same workload as used for the benchmark purposes.
* Prepare the profile for Clang with `llvm-profdata merge -output=nginx.profdata nginx_*.profraw`
* Compile Nginx once again with the collected profiles with `CC=clang CXX=clang++ ./configure --with-cc=/usr/bin/clang --with-cpp=/usr/bin/clang++ --with-cc-opt="-O3 -fprofile-instr-use=/home/zamazan4ik/open_source/bench_nginx/profiles/nginx.profdata" --with-ld-opt="-fprofile-instr-use=/home/zamazan4ik/open_source/bench_nginx/profiles/nginx.profdata"`.

## Results

In short, I get the following RPS results from Nighthawk:
* Release: ~8800 RPS
* Release + PGO: ~9300 RPS
* Instrumented (just for the reference): ~8400 RPS

More detailed reports from Nighthawk are available here:
* Release: https://pastebin.com/fBbvveM3
* Release + PGO: https://pastebin.com/QTWk9WDp
* Instrumented (just for the reference): https://pastebin.com/23PPdM4z

According to the tests, PGO helps with optimizing Nginx's performance from the throughput perspective at least in the tested scenario. However, from the latency perspective the results are worse with PGO compared to the Release build - it should be investigated further.

I think more robust benchmarks are required here to perform since these benchmarks on one local machine can be quite noisy.
