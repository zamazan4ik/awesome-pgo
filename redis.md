# Redis

## Test environment

* Fedora 38
* Linux kernel 6.3.7
* AMD Ryzen 9 5900x
* 48 Gib RAM
* SSD Samsung 980 Pro 2 Tib
* Clang 16 (from the Fedora repositories). I use Clang just because I prefer LLVM-based tooling 
* Redis version: the most recent to the date from `unstable` branch (commit `26174123eecb4539406b81aff3a6921c6cc6abba`)

## Tested configurations

I have tested the following Redis configurations (with corresponding `CFLAGS` and `LDFLAGS`):

* Release: `CC=clang make`
* Release with PGO: `CC=clang CFLAGS="-fprofile-instr-use=redis.profdata" LDFLAGS="-fprofile-instr-use=redis.profdata" make`

As a PGO technique, I use `-fprofile-instr-generate`/`-fprofile-instr-use` options from Clang. Build instrumented `redis-server` version, run `redis-benchmark` (I use the usual Release one since we are going to optimize `redis-server` not `redis-benchmark`) with the instrumented `redis-server`, collect instrumentation data, then rebuild `redis-server` again with the collected data.

## Benchmark

I use `redis-benchmark` with `taskset -c 1-3 redis-benchmark -n 1000000 --threads 3`. `redis-server` is started with the command `taskset -c 0 redis-server --logfile redis.log --save ""` .

## Results

Here are the results of running the benchmark on different configurations. All configurations are benchmarked on the same machine, with the same Redis configuration, multiple times, etc. The results are shown in `redis-benchmark` format. I have rechecked - the results are consistent between runs.
 
Also, I shared results for Instrumentation mode so you would be able to estimate how slow Redis is in Instrumentation mode for PGO.

<details>
  <summary>Release</summary>

  ```
taskset -c 1-3 ../redis_release/redis-benchmark -n 1000000 --threads 3
====== PING_INLINE ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 7)
50.000% <= 0.159 milliseconds (cumulative count 885065)
93.750% <= 0.191 milliseconds (cumulative count 954884)
96.875% <= 0.207 milliseconds (cumulative count 969728)
98.438% <= 0.303 milliseconds (cumulative count 987210)
99.219% <= 0.311 milliseconds (cumulative count 996777)
99.805% <= 0.319 milliseconds (cumulative count 998951)
99.902% <= 0.327 milliseconds (cumulative count 999254)
99.951% <= 0.351 milliseconds (cumulative count 999560)
99.976% <= 0.375 milliseconds (cumulative count 999872)
99.988% <= 0.383 milliseconds (cumulative count 999902)
99.994% <= 0.415 milliseconds (cumulative count 999951)
99.997% <= 0.447 milliseconds (cumulative count 999972)
99.998% <= 0.503 milliseconds (cumulative count 999987)
99.999% <= 0.671 milliseconds (cumulative count 999993)
100.000% <= 0.791 milliseconds (cumulative count 999997)
100.000% <= 0.823 milliseconds (cumulative count 999999)
100.000% <= 0.879 milliseconds (cumulative count 1000000)
100.000% <= 0.879 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.460% <= 0.103 milliseconds (cumulative count 4600)
96.973% <= 0.207 milliseconds (cumulative count 969728)
98.721% <= 0.303 milliseconds (cumulative count 987210)
99.994% <= 0.407 milliseconds (cumulative count 999938)
99.999% <= 0.503 milliseconds (cumulative count 999987)
99.999% <= 0.607 milliseconds (cumulative count 999992)
100.000% <= 0.703 milliseconds (cumulative count 999996)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307503.06 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.158     0.024     0.159     0.191     0.311     0.879
====== PING_MBULK ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.023 milliseconds (cumulative count 2)
50.000% <= 0.151 milliseconds (cumulative count 532844)
75.000% <= 0.159 milliseconds (cumulative count 904885)
93.750% <= 0.183 milliseconds (cumulative count 943371)
96.875% <= 0.207 milliseconds (cumulative count 969318)
98.438% <= 0.303 milliseconds (cumulative count 992243)
99.609% <= 0.311 milliseconds (cumulative count 998252)
99.902% <= 0.319 milliseconds (cumulative count 999361)
99.951% <= 0.327 milliseconds (cumulative count 999544)
99.976% <= 0.351 milliseconds (cumulative count 999769)
99.988% <= 0.391 milliseconds (cumulative count 999883)
99.994% <= 0.431 milliseconds (cumulative count 999947)
99.997% <= 0.439 milliseconds (cumulative count 999974)
99.998% <= 0.447 milliseconds (cumulative count 999989)
99.999% <= 0.455 milliseconds (cumulative count 999993)
100.000% <= 0.639 milliseconds (cumulative count 999997)
100.000% <= 0.855 milliseconds (cumulative count 999999)
100.000% <= 0.879 milliseconds (cumulative count 1000000)
100.000% <= 0.879 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.682% <= 0.103 milliseconds (cumulative count 6822)
96.932% <= 0.207 milliseconds (cumulative count 969318)
99.224% <= 0.303 milliseconds (cumulative count 992243)
99.990% <= 0.407 milliseconds (cumulative count 999905)
99.999% <= 0.503 milliseconds (cumulative count 999993)
99.999% <= 0.607 milliseconds (cumulative count 999995)
100.000% <= 0.703 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307597.66 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.156     0.016     0.151     0.191     0.303     0.879
====== SET ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 6)
50.000% <= 0.159 milliseconds (cumulative count 523499)
75.000% <= 0.167 milliseconds (cumulative count 899898)
93.750% <= 0.199 milliseconds (cumulative count 947407)
96.875% <= 0.215 milliseconds (cumulative count 969532)
98.438% <= 0.319 milliseconds (cumulative count 991978)
99.219% <= 0.327 milliseconds (cumulative count 998185)
99.902% <= 0.335 milliseconds (cumulative count 999483)
99.951% <= 0.343 milliseconds (cumulative count 999709)
99.976% <= 0.351 milliseconds (cumulative count 999812)
99.988% <= 0.367 milliseconds (cumulative count 999920)
99.994% <= 0.375 milliseconds (cumulative count 999941)
99.997% <= 0.391 milliseconds (cumulative count 999972)
99.998% <= 0.407 milliseconds (cumulative count 999989)
99.999% <= 0.415 milliseconds (cumulative count 999998)
100.000% <= 0.423 milliseconds (cumulative count 999999)
100.000% <= 0.431 milliseconds (cumulative count 1000000)
100.000% <= 0.431 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.535% <= 0.103 milliseconds (cumulative count 5347)
96.596% <= 0.207 milliseconds (cumulative count 965964)
97.660% <= 0.303 milliseconds (cumulative count 976604)
99.999% <= 0.407 milliseconds (cumulative count 999989)
100.000% <= 0.503 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.165     0.024     0.159     0.207     0.319     0.431
====== GET ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 14)
50.000% <= 0.159 milliseconds (cumulative count 755276)
87.500% <= 0.167 milliseconds (cumulative count 913946)
93.750% <= 0.191 milliseconds (cumulative count 940104)
96.875% <= 0.215 milliseconds (cumulative count 969485)
98.438% <= 0.311 milliseconds (cumulative count 989104)
99.219% <= 0.319 milliseconds (cumulative count 997246)
99.805% <= 0.327 milliseconds (cumulative count 999235)
99.951% <= 0.335 milliseconds (cumulative count 999537)
99.976% <= 0.359 milliseconds (cumulative count 999772)
99.988% <= 0.399 milliseconds (cumulative count 999893)
99.994% <= 0.415 milliseconds (cumulative count 999942)
99.997% <= 0.479 milliseconds (cumulative count 999970)
99.998% <= 0.543 milliseconds (cumulative count 999988)
99.999% <= 0.575 milliseconds (cumulative count 999994)
100.000% <= 0.679 milliseconds (cumulative count 999997)
100.000% <= 0.799 milliseconds (cumulative count 1000000)
100.000% <= 0.799 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.556% <= 0.103 milliseconds (cumulative count 5563)
96.835% <= 0.207 milliseconds (cumulative count 968351)
97.956% <= 0.303 milliseconds (cumulative count 979561)
99.992% <= 0.407 milliseconds (cumulative count 999922)
99.997% <= 0.503 milliseconds (cumulative count 999971)
99.999% <= 0.607 milliseconds (cumulative count 999994)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285714.28 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.162     0.024     0.159     0.199     0.319     0.799
====== INCR ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 10)
50.000% <= 0.159 milliseconds (cumulative count 708852)
75.000% <= 0.167 milliseconds (cumulative count 909098)
93.750% <= 0.199 milliseconds (cumulative count 957247)
96.875% <= 0.223 milliseconds (cumulative count 969108)
98.438% <= 0.311 milliseconds (cumulative count 985853)
99.219% <= 0.319 milliseconds (cumulative count 995076)
99.609% <= 0.327 milliseconds (cumulative count 998267)
99.902% <= 0.343 milliseconds (cumulative count 999113)
99.951% <= 0.359 milliseconds (cumulative count 999552)
99.976% <= 0.391 milliseconds (cumulative count 999813)
99.988% <= 0.431 milliseconds (cumulative count 999884)
99.994% <= 0.487 milliseconds (cumulative count 999940)
99.997% <= 0.551 milliseconds (cumulative count 999971)
99.998% <= 0.663 milliseconds (cumulative count 999985)
99.999% <= 0.711 milliseconds (cumulative count 999993)
100.000% <= 0.799 milliseconds (cumulative count 999997)
100.000% <= 0.855 milliseconds (cumulative count 1000000)
100.000% <= 0.855 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.514% <= 0.103 milliseconds (cumulative count 5137)
96.694% <= 0.207 milliseconds (cumulative count 966940)
97.715% <= 0.303 milliseconds (cumulative count 977152)
99.986% <= 0.407 milliseconds (cumulative count 999856)
99.995% <= 0.503 milliseconds (cumulative count 999945)
99.997% <= 0.607 milliseconds (cumulative count 999972)
99.999% <= 0.703 milliseconds (cumulative count 999989)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285714.28 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.163     0.024     0.159     0.199     0.319     0.855
====== LPUSH ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 8)
50.000% <= 0.167 milliseconds (cumulative count 885624)
93.750% <= 0.199 milliseconds (cumulative count 940956)
96.875% <= 0.223 milliseconds (cumulative count 969588)
98.438% <= 0.319 milliseconds (cumulative count 988715)
99.219% <= 0.327 milliseconds (cumulative count 997045)
99.805% <= 0.335 milliseconds (cumulative count 999159)
99.951% <= 0.351 milliseconds (cumulative count 999637)
99.976% <= 0.367 milliseconds (cumulative count 999790)
99.988% <= 0.423 milliseconds (cumulative count 999879)
99.994% <= 0.463 milliseconds (cumulative count 999941)
99.997% <= 0.543 milliseconds (cumulative count 999970)
99.998% <= 0.567 milliseconds (cumulative count 999988)
99.999% <= 0.615 milliseconds (cumulative count 999993)
100.000% <= 0.863 milliseconds (cumulative count 999997)
100.000% <= 0.887 milliseconds (cumulative count 1000000)
100.000% <= 0.887 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.400% <= 0.103 milliseconds (cumulative count 3998)
96.075% <= 0.207 milliseconds (cumulative count 960753)
97.529% <= 0.303 milliseconds (cumulative count 975291)
99.987% <= 0.407 milliseconds (cumulative count 999874)
99.996% <= 0.503 milliseconds (cumulative count 999965)
99.999% <= 0.607 milliseconds (cumulative count 999992)
99.999% <= 0.703 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.166     0.024     0.167     0.207     0.327     0.887
====== RPUSH ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 5)
50.000% <= 0.167 milliseconds (cumulative count 893705)
93.750% <= 0.199 milliseconds (cumulative count 943968)
96.875% <= 0.223 milliseconds (cumulative count 969623)
98.438% <= 0.319 milliseconds (cumulative count 990229)
99.219% <= 0.327 milliseconds (cumulative count 997829)
99.805% <= 0.335 milliseconds (cumulative count 999391)
99.951% <= 0.343 milliseconds (cumulative count 999627)
99.976% <= 0.351 milliseconds (cumulative count 999760)
99.988% <= 0.375 milliseconds (cumulative count 999910)
99.994% <= 0.399 milliseconds (cumulative count 999948)
99.997% <= 0.487 milliseconds (cumulative count 999971)
99.998% <= 0.551 milliseconds (cumulative count 999988)
99.999% <= 0.559 milliseconds (cumulative count 999997)
100.000% <= 0.567 milliseconds (cumulative count 999999)
100.000% <= 0.591 milliseconds (cumulative count 1000000)
100.000% <= 0.591 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.404% <= 0.103 milliseconds (cumulative count 4035)
96.387% <= 0.207 milliseconds (cumulative count 963869)
97.553% <= 0.303 milliseconds (cumulative count 975534)
99.996% <= 0.407 milliseconds (cumulative count 999958)
99.997% <= 0.503 milliseconds (cumulative count 999974)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285714.28 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.166     0.024     0.167     0.207     0.319     0.591
====== LPOP ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 4)
50.000% <= 0.167 milliseconds (cumulative count 627289)
75.000% <= 0.175 milliseconds (cumulative count 907187)
93.750% <= 0.207 milliseconds (cumulative count 940390)
96.875% <= 0.231 milliseconds (cumulative count 970105)
98.438% <= 0.335 milliseconds (cumulative count 994088)
99.609% <= 0.343 milliseconds (cumulative count 998593)
99.902% <= 0.351 milliseconds (cumulative count 999429)
99.951% <= 0.359 milliseconds (cumulative count 999615)
99.976% <= 0.375 milliseconds (cumulative count 999775)
99.988% <= 0.407 milliseconds (cumulative count 999878)
99.994% <= 0.455 milliseconds (cumulative count 999939)
99.997% <= 0.567 milliseconds (cumulative count 999971)
99.998% <= 0.591 milliseconds (cumulative count 999988)
99.999% <= 0.663 milliseconds (cumulative count 999993)
100.000% <= 0.879 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 1000000)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.243% <= 0.103 milliseconds (cumulative count 2431)
94.039% <= 0.207 milliseconds (cumulative count 940390)
97.400% <= 0.303 milliseconds (cumulative count 973997)
99.988% <= 0.407 milliseconds (cumulative count 999878)
99.997% <= 0.503 milliseconds (cumulative count 999966)
99.999% <= 0.607 milliseconds (cumulative count 999989)
99.999% <= 0.703 milliseconds (cumulative count 999993)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.172     0.024     0.167     0.215     0.335     0.903
====== RPOP ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 5)
50.000% <= 0.167 milliseconds (cumulative count 781863)
87.500% <= 0.175 milliseconds (cumulative count 913467)
93.750% <= 0.207 milliseconds (cumulative count 943988)
96.875% <= 0.231 milliseconds (cumulative count 968808)
98.438% <= 0.327 milliseconds (cumulative count 990581)
99.219% <= 0.335 milliseconds (cumulative count 997640)
99.805% <= 0.343 milliseconds (cumulative count 999268)
99.951% <= 0.351 milliseconds (cumulative count 999515)
99.976% <= 0.375 milliseconds (cumulative count 999776)
99.988% <= 0.415 milliseconds (cumulative count 999878)
99.994% <= 0.503 milliseconds (cumulative count 999944)
99.997% <= 0.567 milliseconds (cumulative count 999973)
99.998% <= 0.583 milliseconds (cumulative count 999988)
99.999% <= 0.639 milliseconds (cumulative count 999993)
100.000% <= 0.783 milliseconds (cumulative count 999997)
100.000% <= 0.847 milliseconds (cumulative count 999999)
100.000% <= 0.855 milliseconds (cumulative count 1000000)
100.000% <= 0.855 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.398% <= 0.103 milliseconds (cumulative count 3985)
94.399% <= 0.207 milliseconds (cumulative count 943988)
97.444% <= 0.303 milliseconds (cumulative count 974439)
99.987% <= 0.407 milliseconds (cumulative count 999871)
99.994% <= 0.503 milliseconds (cumulative count 999944)
99.999% <= 0.607 milliseconds (cumulative count 999991)
99.999% <= 0.703 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285632.69 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.170     0.024     0.167     0.215     0.327     0.855
====== SADD ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 17)
50.000% <= 0.159 milliseconds (cumulative count 643363)
75.000% <= 0.167 milliseconds (cumulative count 913208)
93.750% <= 0.199 milliseconds (cumulative count 954730)
96.875% <= 0.223 milliseconds (cumulative count 969552)
98.438% <= 0.311 milliseconds (cumulative count 984832)
99.219% <= 0.319 milliseconds (cumulative count 995028)
99.609% <= 0.327 milliseconds (cumulative count 999083)
99.951% <= 0.335 milliseconds (cumulative count 999555)
99.976% <= 0.351 milliseconds (cumulative count 999769)
99.988% <= 0.383 milliseconds (cumulative count 999892)
99.994% <= 0.495 milliseconds (cumulative count 999940)
99.997% <= 0.655 milliseconds (cumulative count 999995)
100.000% <= 0.695 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 1000000)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.598% <= 0.103 milliseconds (cumulative count 5975)
96.697% <= 0.207 milliseconds (cumulative count 966974)
97.743% <= 0.303 milliseconds (cumulative count 977427)
99.993% <= 0.407 milliseconds (cumulative count 999928)
99.994% <= 0.503 milliseconds (cumulative count 999940)
99.995% <= 0.607 milliseconds (cumulative count 999950)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.163     0.024     0.159     0.199     0.319     0.807
====== HSET ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.023 milliseconds (cumulative count 1)
50.000% <= 0.167 milliseconds (cumulative count 856174)
87.500% <= 0.175 milliseconds (cumulative count 919513)
93.750% <= 0.207 milliseconds (cumulative count 955269)
96.875% <= 0.223 milliseconds (cumulative count 969757)
98.438% <= 0.327 milliseconds (cumulative count 994416)
99.609% <= 0.335 milliseconds (cumulative count 998889)
99.902% <= 0.343 milliseconds (cumulative count 999508)
99.951% <= 0.351 milliseconds (cumulative count 999654)
99.976% <= 0.367 milliseconds (cumulative count 999783)
99.988% <= 0.407 milliseconds (cumulative count 999882)
99.994% <= 0.559 milliseconds (cumulative count 999949)
99.997% <= 0.615 milliseconds (cumulative count 999970)
99.998% <= 0.655 milliseconds (cumulative count 999985)
99.999% <= 0.695 milliseconds (cumulative count 999993)
100.000% <= 0.847 milliseconds (cumulative count 999997)
100.000% <= 1.087 milliseconds (cumulative count 999999)
100.000% <= 1.103 milliseconds (cumulative count 1000000)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.330% <= 0.103 milliseconds (cumulative count 3296)
95.527% <= 0.207 milliseconds (cumulative count 955269)
97.416% <= 0.303 milliseconds (cumulative count 974164)
99.988% <= 0.407 milliseconds (cumulative count 999882)
99.992% <= 0.503 milliseconds (cumulative count 999923)
99.997% <= 0.607 milliseconds (cumulative count 999966)
99.999% <= 0.703 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 999998)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.168     0.016     0.167     0.207     0.327     1.103
====== SPOP ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 14)
50.000% <= 0.159 milliseconds (cumulative count 828042)
87.500% <= 0.167 milliseconds (cumulative count 918197)
93.750% <= 0.191 milliseconds (cumulative count 947876)
96.875% <= 0.207 milliseconds (cumulative count 969898)
98.438% <= 0.311 milliseconds (cumulative count 992783)
99.609% <= 0.319 milliseconds (cumulative count 998543)
99.902% <= 0.327 milliseconds (cumulative count 999465)
99.951% <= 0.335 milliseconds (cumulative count 999625)
99.976% <= 0.351 milliseconds (cumulative count 999791)
99.988% <= 0.383 milliseconds (cumulative count 999886)
99.994% <= 0.423 milliseconds (cumulative count 999939)
99.997% <= 0.439 milliseconds (cumulative count 999973)
99.998% <= 0.471 milliseconds (cumulative count 999985)
99.999% <= 0.639 milliseconds (cumulative count 999994)
100.000% <= 0.815 milliseconds (cumulative count 999997)
100.000% <= 0.839 milliseconds (cumulative count 999999)
100.000% <= 0.847 milliseconds (cumulative count 1000000)
100.000% <= 0.847 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.507% <= 0.103 milliseconds (cumulative count 5070)
96.990% <= 0.207 milliseconds (cumulative count 969898)
98.191% <= 0.303 milliseconds (cumulative count 981914)
99.992% <= 0.407 milliseconds (cumulative count 999925)
99.999% <= 0.503 milliseconds (cumulative count 999987)
99.999% <= 0.607 milliseconds (cumulative count 999989)
100.000% <= 0.703 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.160     0.024     0.159     0.199     0.311     0.847
====== ZADD ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 4)
50.000% <= 0.167 milliseconds (cumulative count 798180)
87.500% <= 0.175 milliseconds (cumulative count 917443)
93.750% <= 0.207 milliseconds (cumulative count 949216)
96.875% <= 0.223 milliseconds (cumulative count 971196)
98.438% <= 0.327 milliseconds (cumulative count 990807)
99.219% <= 0.335 milliseconds (cumulative count 997998)
99.805% <= 0.343 milliseconds (cumulative count 999434)
99.951% <= 0.351 milliseconds (cumulative count 999635)
99.976% <= 0.367 milliseconds (cumulative count 999802)
99.988% <= 0.399 milliseconds (cumulative count 999891)
99.994% <= 0.487 milliseconds (cumulative count 999941)
99.997% <= 0.599 milliseconds (cumulative count 999971)
99.998% <= 0.903 milliseconds (cumulative count 999985)
99.999% <= 0.951 milliseconds (cumulative count 999993)
100.000% <= 0.975 milliseconds (cumulative count 999997)
100.000% <= 0.999 milliseconds (cumulative count 999999)
100.000% <= 1.047 milliseconds (cumulative count 1000000)
100.000% <= 1.047 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.321% <= 0.103 milliseconds (cumulative count 3213)
94.922% <= 0.207 milliseconds (cumulative count 949216)
97.477% <= 0.303 milliseconds (cumulative count 974767)
99.990% <= 0.407 milliseconds (cumulative count 999905)
99.994% <= 0.503 milliseconds (cumulative count 999944)
99.998% <= 0.607 milliseconds (cumulative count 999980)
99.998% <= 0.703 milliseconds (cumulative count 999981)
99.998% <= 0.807 milliseconds (cumulative count 999983)
99.999% <= 0.903 milliseconds (cumulative count 999985)
100.000% <= 1.007 milliseconds (cumulative count 999999)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.169     0.024     0.167     0.215     0.327     1.047
====== ZPOPMIN ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 17)
50.000% <= 0.159 milliseconds (cumulative count 806358)
87.500% <= 0.167 milliseconds (cumulative count 919035)
93.750% <= 0.191 milliseconds (cumulative count 947687)
96.875% <= 0.207 milliseconds (cumulative count 969215)
98.438% <= 0.311 milliseconds (cumulative count 991957)
99.219% <= 0.319 milliseconds (cumulative count 998293)
99.902% <= 0.327 milliseconds (cumulative count 999497)
99.951% <= 0.335 milliseconds (cumulative count 999675)
99.976% <= 0.343 milliseconds (cumulative count 999764)
99.988% <= 0.359 milliseconds (cumulative count 999894)
99.994% <= 0.375 milliseconds (cumulative count 999942)
99.997% <= 0.391 milliseconds (cumulative count 999975)
99.998% <= 0.399 milliseconds (cumulative count 999986)
99.999% <= 0.407 milliseconds (cumulative count 999994)
100.000% <= 0.415 milliseconds (cumulative count 999999)
100.000% <= 0.423 milliseconds (cumulative count 1000000)
100.000% <= 0.423 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.670% <= 0.103 milliseconds (cumulative count 6700)
96.922% <= 0.207 milliseconds (cumulative count 969215)
98.159% <= 0.303 milliseconds (cumulative count 981595)
99.999% <= 0.407 milliseconds (cumulative count 999994)
100.000% <= 0.503 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285714.28 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.160     0.024     0.159     0.199     0.311     0.423
====== LPUSH (needed to benchmark LRANGE) ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 13)
50.000% <= 0.167 milliseconds (cumulative count 865871)
87.500% <= 0.175 milliseconds (cumulative count 917174)
93.750% <= 0.199 milliseconds (cumulative count 938255)
96.875% <= 0.223 milliseconds (cumulative count 968890)
98.438% <= 0.319 milliseconds (cumulative count 986185)
99.219% <= 0.327 milliseconds (cumulative count 995405)
99.609% <= 0.335 milliseconds (cumulative count 998994)
99.902% <= 0.343 milliseconds (cumulative count 999520)
99.976% <= 0.367 milliseconds (cumulative count 999766)
99.988% <= 0.407 milliseconds (cumulative count 999897)
99.994% <= 0.455 milliseconds (cumulative count 999942)
99.997% <= 0.487 milliseconds (cumulative count 999977)
99.998% <= 0.495 milliseconds (cumulative count 999986)
99.999% <= 0.623 milliseconds (cumulative count 999993)
100.000% <= 0.903 milliseconds (cumulative count 999997)
100.000% <= 0.919 milliseconds (cumulative count 999999)
100.000% <= 0.927 milliseconds (cumulative count 1000000)
100.000% <= 0.927 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.520% <= 0.103 milliseconds (cumulative count 5197)
95.713% <= 0.207 milliseconds (cumulative count 957127)
97.501% <= 0.303 milliseconds (cumulative count 975006)
99.990% <= 0.407 milliseconds (cumulative count 999897)
99.999% <= 0.503 milliseconds (cumulative count 999988)
99.999% <= 0.607 milliseconds (cumulative count 999990)
99.999% <= 0.703 milliseconds (cumulative count 999994)
99.999% <= 0.807 milliseconds (cumulative count 999995)
100.000% <= 0.903 milliseconds (cumulative count 999997)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.167     0.024     0.167     0.207     0.327     0.927
====== LRANGE_100 (first 100 elements) ======                                                     
  1000000 requests completed in 6.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.103 milliseconds (cumulative count 1)
50.000% <= 0.303 milliseconds (cumulative count 504770)
75.000% <= 0.367 milliseconds (cumulative count 779870)
87.500% <= 0.391 milliseconds (cumulative count 909354)
93.750% <= 0.399 milliseconds (cumulative count 940699)
96.875% <= 0.415 milliseconds (cumulative count 975308)
98.438% <= 0.431 milliseconds (cumulative count 984615)
99.219% <= 0.455 milliseconds (cumulative count 992539)
99.609% <= 0.471 milliseconds (cumulative count 997148)
99.805% <= 0.479 milliseconds (cumulative count 998470)
99.902% <= 0.487 milliseconds (cumulative count 999249)
99.951% <= 0.495 milliseconds (cumulative count 999619)
99.976% <= 0.503 milliseconds (cumulative count 999790)
99.988% <= 0.519 milliseconds (cumulative count 999891)
99.994% <= 0.543 milliseconds (cumulative count 999942)
99.997% <= 0.631 milliseconds (cumulative count 999970)
99.998% <= 0.719 milliseconds (cumulative count 999986)
99.999% <= 0.775 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 0.879 milliseconds (cumulative count 999999)
100.000% <= 0.903 milliseconds (cumulative count 1000000)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 1)
0.544% <= 0.207 milliseconds (cumulative count 5445)
50.477% <= 0.303 milliseconds (cumulative count 504770)
96.281% <= 0.407 milliseconds (cumulative count 962808)
99.979% <= 0.503 milliseconds (cumulative count 999790)
99.997% <= 0.607 milliseconds (cumulative count 999966)
99.998% <= 0.703 milliseconds (cumulative count 999982)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 148104.27 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.306     0.096     0.303     0.407     0.455     0.903
====== LRANGE_300 (first 300 elements) ======                                                   
  1000000 requests completed in 13.26 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.143 milliseconds (cumulative count 1)
50.000% <= 0.551 milliseconds (cumulative count 501424)
75.000% <= 0.607 milliseconds (cumulative count 751566)
87.500% <= 0.647 milliseconds (cumulative count 880668)
93.750% <= 0.687 milliseconds (cumulative count 951944)
96.875% <= 0.711 milliseconds (cumulative count 971643)
98.438% <= 0.727 milliseconds (cumulative count 984577)
99.219% <= 0.743 milliseconds (cumulative count 992779)
99.609% <= 0.759 milliseconds (cumulative count 996304)
99.805% <= 0.799 milliseconds (cumulative count 998162)
99.902% <= 0.839 milliseconds (cumulative count 999189)
99.951% <= 0.855 milliseconds (cumulative count 999539)
99.976% <= 0.871 milliseconds (cumulative count 999818)
99.988% <= 0.895 milliseconds (cumulative count 999889)
99.994% <= 0.943 milliseconds (cumulative count 999939)
99.997% <= 1.007 milliseconds (cumulative count 999970)
99.998% <= 1.071 milliseconds (cumulative count 999985)
99.999% <= 1.111 milliseconds (cumulative count 999993)
100.000% <= 1.135 milliseconds (cumulative count 999998)
100.000% <= 1.143 milliseconds (cumulative count 999999)
100.000% <= 1.151 milliseconds (cumulative count 1000000)
100.000% <= 1.151 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
0.001% <= 0.207 milliseconds (cumulative count 7)
0.120% <= 0.303 milliseconds (cumulative count 1200)
1.812% <= 0.407 milliseconds (cumulative count 18116)
26.293% <= 0.503 milliseconds (cumulative count 262925)
75.157% <= 0.607 milliseconds (cumulative count 751566)
96.568% <= 0.703 milliseconds (cumulative count 965684)
99.830% <= 0.807 milliseconds (cumulative count 998297)
99.990% <= 0.903 milliseconds (cumulative count 999898)
99.997% <= 1.007 milliseconds (cumulative count 999970)
99.999% <= 1.103 milliseconds (cumulative count 999992)
100.000% <= 1.207 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 75414.78 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.556     0.136     0.551     0.687     0.743     1.151
====== LRANGE_500 (first 500 elements) ======                                                   
  1000000 requests completed in 19.77 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.207 milliseconds (cumulative count 2)
50.000% <= 0.823 milliseconds (cumulative count 509422)
75.000% <= 0.895 milliseconds (cumulative count 751477)
87.500% <= 0.935 milliseconds (cumulative count 875675)
93.750% <= 0.951 milliseconds (cumulative count 955007)
96.875% <= 0.959 milliseconds (cumulative count 970065)
98.438% <= 0.983 milliseconds (cumulative count 988298)
99.219% <= 0.991 milliseconds (cumulative count 992540)
99.609% <= 1.039 milliseconds (cumulative count 996110)
99.805% <= 1.103 milliseconds (cumulative count 998078)
99.902% <= 1.391 milliseconds (cumulative count 999042)
99.951% <= 1.551 milliseconds (cumulative count 999533)
99.976% <= 1.631 milliseconds (cumulative count 999764)
99.988% <= 1.687 milliseconds (cumulative count 999891)
99.994% <= 1.719 milliseconds (cumulative count 999941)
99.997% <= 1.759 milliseconds (cumulative count 999971)
99.998% <= 1.807 milliseconds (cumulative count 999985)
99.999% <= 1.823 milliseconds (cumulative count 999993)
100.000% <= 1.879 milliseconds (cumulative count 999997)
100.000% <= 1.903 milliseconds (cumulative count 999999)
100.000% <= 1.983 milliseconds (cumulative count 1000000)
100.000% <= 1.983 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
0.000% <= 0.207 milliseconds (cumulative count 2)
0.001% <= 0.303 milliseconds (cumulative count 7)
0.002% <= 0.407 milliseconds (cumulative count 22)
0.007% <= 0.503 milliseconds (cumulative count 66)
1.348% <= 0.607 milliseconds (cumulative count 13479)
14.670% <= 0.703 milliseconds (cumulative count 146703)
41.975% <= 0.807 milliseconds (cumulative count 419752)
75.997% <= 0.903 milliseconds (cumulative count 759972)
99.453% <= 1.007 milliseconds (cumulative count 994525)
99.808% <= 1.103 milliseconds (cumulative count 998078)
99.857% <= 1.207 milliseconds (cumulative count 998568)
99.882% <= 1.303 milliseconds (cumulative count 998815)
99.908% <= 1.407 milliseconds (cumulative count 999077)
99.938% <= 1.503 milliseconds (cumulative count 999381)
99.970% <= 1.607 milliseconds (cumulative count 999703)
99.992% <= 1.703 milliseconds (cumulative count 999922)
99.999% <= 1.807 milliseconds (cumulative count 999985)
100.000% <= 1.903 milliseconds (cumulative count 999999)
100.000% <= 2.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 50586.81 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.816     0.200     0.823     0.951     0.991     1.983
====== LRANGE_600 (first 600 elements) ======                                                   
  1000000 requests completed in 23.02 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.087 milliseconds (cumulative count 5)
50.000% <= 0.959 milliseconds (cumulative count 512820)
75.000% <= 1.047 milliseconds (cumulative count 760517)
87.500% <= 1.087 milliseconds (cumulative count 875765)
93.750% <= 1.111 milliseconds (cumulative count 955086)
96.875% <= 1.127 milliseconds (cumulative count 975316)
98.438% <= 1.151 milliseconds (cumulative count 988761)
99.219% <= 1.167 milliseconds (cumulative count 992645)
99.609% <= 1.255 milliseconds (cumulative count 996214)
99.805% <= 1.367 milliseconds (cumulative count 998103)
99.902% <= 1.471 milliseconds (cumulative count 999071)
99.951% <= 1.567 milliseconds (cumulative count 999531)
99.976% <= 1.735 milliseconds (cumulative count 999756)
99.988% <= 1.863 milliseconds (cumulative count 999879)
99.994% <= 1.943 milliseconds (cumulative count 999942)
99.997% <= 1.991 milliseconds (cumulative count 999974)
99.998% <= 2.055 milliseconds (cumulative count 999986)
99.999% <= 2.191 milliseconds (cumulative count 999993)
100.000% <= 2.391 milliseconds (cumulative count 999997)
100.000% <= 2.439 milliseconds (cumulative count 999999)
100.000% <= 2.519 milliseconds (cumulative count 1000000)
100.000% <= 2.519 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.001% <= 0.103 milliseconds (cumulative count 8)
0.002% <= 0.207 milliseconds (cumulative count 18)
0.012% <= 0.303 milliseconds (cumulative count 119)
0.063% <= 0.407 milliseconds (cumulative count 627)
0.253% <= 0.503 milliseconds (cumulative count 2531)
0.748% <= 0.607 milliseconds (cumulative count 7481)
1.516% <= 0.703 milliseconds (cumulative count 15164)
12.739% <= 0.807 milliseconds (cumulative count 127390)
31.934% <= 0.903 milliseconds (cumulative count 319340)
64.284% <= 1.007 milliseconds (cumulative count 642836)
90.946% <= 1.103 milliseconds (cumulative count 909460)
99.521% <= 1.207 milliseconds (cumulative count 995208)
99.713% <= 1.303 milliseconds (cumulative count 997131)
99.849% <= 1.407 milliseconds (cumulative count 998493)
99.929% <= 1.503 milliseconds (cumulative count 999291)
99.961% <= 1.607 milliseconds (cumulative count 999612)
99.972% <= 1.703 milliseconds (cumulative count 999725)
99.983% <= 1.807 milliseconds (cumulative count 999828)
99.991% <= 1.903 milliseconds (cumulative count 999911)
99.998% <= 2.007 milliseconds (cumulative count 999980)
99.999% <= 2.103 milliseconds (cumulative count 999991)
100.000% <= 3.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 43438.60 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.951     0.080     0.959     1.111     1.159     2.519
====== MSET (10 keys) ======                                                     
  1000000 requests completed in 4.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.047 milliseconds (cumulative count 2)
50.000% <= 0.207 milliseconds (cumulative count 827658)
87.500% <= 0.215 milliseconds (cumulative count 924366)
93.750% <= 0.263 milliseconds (cumulative count 938070)
96.875% <= 0.295 milliseconds (cumulative count 971444)
98.438% <= 0.407 milliseconds (cumulative count 992400)
99.609% <= 0.415 milliseconds (cumulative count 998437)
99.902% <= 0.423 milliseconds (cumulative count 999556)
99.976% <= 0.439 milliseconds (cumulative count 999813)
99.988% <= 0.455 milliseconds (cumulative count 999899)
99.994% <= 0.471 milliseconds (cumulative count 999968)
99.997% <= 0.479 milliseconds (cumulative count 999987)
99.999% <= 0.487 milliseconds (cumulative count 999993)
100.000% <= 0.511 milliseconds (cumulative count 999997)
100.000% <= 0.519 milliseconds (cumulative count 999999)
100.000% <= 0.567 milliseconds (cumulative count 1000000)
100.000% <= 0.567 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.041% <= 0.103 milliseconds (cumulative count 409)
82.766% <= 0.207 milliseconds (cumulative count 827658)
97.288% <= 0.303 milliseconds (cumulative count 972876)
99.240% <= 0.407 milliseconds (cumulative count 992400)
100.000% <= 0.503 milliseconds (cumulative count 999996)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 222222.22 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.211     0.040     0.207     0.287     0.407     0.567
====== XADD ======                                                     
  1000000 requests completed in 4.00 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 1)
50.000% <= 0.183 milliseconds (cumulative count 673124)
75.000% <= 0.191 milliseconds (cumulative count 911542)
93.750% <= 0.239 milliseconds (cumulative count 944968)
96.875% <= 0.255 milliseconds (cumulative count 969819)
98.438% <= 0.359 milliseconds (cumulative count 986871)
99.219% <= 0.367 milliseconds (cumulative count 995362)
99.609% <= 0.375 milliseconds (cumulative count 999060)
99.951% <= 0.383 milliseconds (cumulative count 999562)
99.976% <= 0.407 milliseconds (cumulative count 999793)
99.988% <= 0.431 milliseconds (cumulative count 999896)
99.994% <= 0.471 milliseconds (cumulative count 999939)
99.997% <= 0.871 milliseconds (cumulative count 999970)
99.998% <= 0.911 milliseconds (cumulative count 999995)
100.000% <= 0.919 milliseconds (cumulative count 999998)
100.000% <= 0.927 milliseconds (cumulative count 999999)
100.000% <= 1.295 milliseconds (cumulative count 1000000)
100.000% <= 1.295 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.198% <= 0.103 milliseconds (cumulative count 1983)
92.552% <= 0.207 milliseconds (cumulative count 925515)
97.219% <= 0.303 milliseconds (cumulative count 972190)
99.979% <= 0.407 milliseconds (cumulative count 999793)
99.994% <= 0.503 milliseconds (cumulative count 999941)
99.995% <= 0.607 milliseconds (cumulative count 999950)
99.996% <= 0.807 milliseconds (cumulative count 999958)
99.998% <= 0.903 milliseconds (cumulative count 999984)
100.000% <= 1.007 milliseconds (cumulative count 999999)
100.000% <= 1.303 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 249812.66 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.188     0.024     0.183     0.247     0.367     1.295
  ```
</details>

<details>
  <summary>Release + PGO</summary>

  ```
====== PING_INLINE ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.023 milliseconds (cumulative count 1)
50.000% <= 0.151 milliseconds (cumulative count 710369)
75.000% <= 0.159 milliseconds (cumulative count 905620)
93.750% <= 0.183 milliseconds (cumulative count 949740)
96.875% <= 0.207 milliseconds (cumulative count 969018)
98.438% <= 0.295 milliseconds (cumulative count 987291)
99.219% <= 0.303 milliseconds (cumulative count 996199)
99.805% <= 0.311 milliseconds (cumulative count 998891)
99.902% <= 0.319 milliseconds (cumulative count 999301)
99.951% <= 0.335 milliseconds (cumulative count 999526)
99.976% <= 0.359 milliseconds (cumulative count 999843)
99.988% <= 0.367 milliseconds (cumulative count 999927)
99.994% <= 0.375 milliseconds (cumulative count 999950)
99.997% <= 0.391 milliseconds (cumulative count 999970)
99.998% <= 0.503 milliseconds (cumulative count 999990)
99.999% <= 0.511 milliseconds (cumulative count 999995)
100.000% <= 0.519 milliseconds (cumulative count 999997)
100.000% <= 0.535 milliseconds (cumulative count 999999)
100.000% <= 0.551 milliseconds (cumulative count 1000000)
100.000% <= 0.551 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.646% <= 0.103 milliseconds (cumulative count 6460)
96.902% <= 0.207 milliseconds (cumulative count 969018)
99.620% <= 0.303 milliseconds (cumulative count 996199)
99.998% <= 0.407 milliseconds (cumulative count 999977)
99.999% <= 0.503 milliseconds (cumulative count 999990)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307692.31 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.154     0.016     0.151     0.191     0.303     0.551
====== PING_MBULK ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 55)
50.000% <= 0.151 milliseconds (cumulative count 811456)
87.500% <= 0.159 milliseconds (cumulative count 919376)
93.750% <= 0.175 milliseconds (cumulative count 941096)
96.875% <= 0.207 milliseconds (cumulative count 969129)
98.438% <= 0.295 milliseconds (cumulative count 992520)
99.609% <= 0.303 milliseconds (cumulative count 998121)
99.902% <= 0.311 milliseconds (cumulative count 999344)
99.951% <= 0.319 milliseconds (cumulative count 999531)
99.976% <= 0.343 milliseconds (cumulative count 999764)
99.988% <= 0.367 milliseconds (cumulative count 999878)
99.994% <= 0.391 milliseconds (cumulative count 999944)
99.997% <= 0.479 milliseconds (cumulative count 999971)
99.998% <= 0.511 milliseconds (cumulative count 999988)
99.999% <= 0.559 milliseconds (cumulative count 999993)
100.000% <= 0.679 milliseconds (cumulative count 999997)
100.000% <= 0.719 milliseconds (cumulative count 999999)
100.000% <= 0.743 milliseconds (cumulative count 1000000)
100.000% <= 0.743 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.887% <= 0.103 milliseconds (cumulative count 8867)
96.913% <= 0.207 milliseconds (cumulative count 969129)
99.812% <= 0.303 milliseconds (cumulative count 998121)
99.995% <= 0.407 milliseconds (cumulative count 999951)
99.998% <= 0.503 milliseconds (cumulative count 999984)
99.999% <= 0.607 milliseconds (cumulative count 999993)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307692.31 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.152     0.024     0.151     0.183     0.295     0.743
====== SET ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 10)
50.000% <= 0.151 milliseconds (cumulative count 574970)
75.000% <= 0.159 milliseconds (cumulative count 901671)
93.750% <= 0.183 milliseconds (cumulative count 940794)
96.875% <= 0.215 milliseconds (cumulative count 969363)
98.438% <= 0.303 milliseconds (cumulative count 993132)
99.609% <= 0.311 milliseconds (cumulative count 998365)
99.902% <= 0.319 milliseconds (cumulative count 999348)
99.951% <= 0.327 milliseconds (cumulative count 999545)
99.976% <= 0.351 milliseconds (cumulative count 999770)
99.988% <= 0.391 milliseconds (cumulative count 999892)
99.994% <= 0.415 milliseconds (cumulative count 999939)
99.997% <= 0.479 milliseconds (cumulative count 999970)
99.998% <= 0.519 milliseconds (cumulative count 999988)
99.999% <= 0.543 milliseconds (cumulative count 999995)
100.000% <= 0.767 milliseconds (cumulative count 999999)
100.000% <= 0.775 milliseconds (cumulative count 1000000)
100.000% <= 0.775 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.587% <= 0.103 milliseconds (cumulative count 5869)
96.849% <= 0.207 milliseconds (cumulative count 968485)
99.313% <= 0.303 milliseconds (cumulative count 993132)
99.992% <= 0.407 milliseconds (cumulative count 999922)
99.998% <= 0.503 milliseconds (cumulative count 999976)
99.999% <= 0.607 milliseconds (cumulative count 999995)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307503.06 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.156     0.024     0.151     0.191     0.303     0.775
====== GET ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 20)
50.000% <= 0.151 milliseconds (cumulative count 707775)
75.000% <= 0.159 milliseconds (cumulative count 911754)
93.750% <= 0.183 milliseconds (cumulative count 949782)
96.875% <= 0.215 milliseconds (cumulative count 969351)
98.438% <= 0.295 milliseconds (cumulative count 987115)
99.219% <= 0.303 milliseconds (cumulative count 996295)
99.805% <= 0.311 milliseconds (cumulative count 998988)
99.902% <= 0.319 milliseconds (cumulative count 999520)
99.976% <= 0.343 milliseconds (cumulative count 999810)
99.988% <= 0.359 milliseconds (cumulative count 999903)
99.994% <= 0.383 milliseconds (cumulative count 999944)
99.997% <= 0.455 milliseconds (cumulative count 999972)
99.998% <= 0.519 milliseconds (cumulative count 999990)
99.999% <= 0.527 milliseconds (cumulative count 999997)
100.000% <= 0.535 milliseconds (cumulative count 1000000)
100.000% <= 0.535 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.687% <= 0.103 milliseconds (cumulative count 6874)
96.855% <= 0.207 milliseconds (cumulative count 968554)
99.630% <= 0.303 milliseconds (cumulative count 996295)
99.996% <= 0.407 milliseconds (cumulative count 999962)
99.998% <= 0.503 milliseconds (cumulative count 999978)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307503.06 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.154     0.024     0.151     0.191     0.303     0.535
====== INCR ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 14)
50.000% <= 0.151 milliseconds (cumulative count 632520)
75.000% <= 0.159 milliseconds (cumulative count 908953)
93.750% <= 0.183 milliseconds (cumulative count 946504)
96.875% <= 0.207 milliseconds (cumulative count 969175)
98.438% <= 0.303 milliseconds (cumulative count 994459)
99.609% <= 0.311 milliseconds (cumulative count 998720)
99.902% <= 0.319 milliseconds (cumulative count 999395)
99.951% <= 0.327 milliseconds (cumulative count 999541)
99.976% <= 0.351 milliseconds (cumulative count 999773)
99.988% <= 0.383 milliseconds (cumulative count 999879)
99.994% <= 0.407 milliseconds (cumulative count 999950)
99.997% <= 0.479 milliseconds (cumulative count 999971)
99.998% <= 0.519 milliseconds (cumulative count 999990)
99.999% <= 0.543 milliseconds (cumulative count 999993)
100.000% <= 0.727 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 1000000)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.632% <= 0.103 milliseconds (cumulative count 6319)
96.918% <= 0.207 milliseconds (cumulative count 969175)
99.446% <= 0.303 milliseconds (cumulative count 994459)
99.995% <= 0.407 milliseconds (cumulative count 999950)
99.997% <= 0.503 milliseconds (cumulative count 999972)
100.000% <= 0.607 milliseconds (cumulative count 999996)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307692.31 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.155     0.024     0.151     0.191     0.303     0.807
====== LPUSH ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 22)
50.000% <= 0.159 milliseconds (cumulative count 828497)
87.500% <= 0.167 milliseconds (cumulative count 916489)
93.750% <= 0.191 milliseconds (cumulative count 947479)
96.875% <= 0.223 milliseconds (cumulative count 969132)
98.438% <= 0.303 milliseconds (cumulative count 985008)
99.219% <= 0.311 milliseconds (cumulative count 994023)
99.609% <= 0.319 milliseconds (cumulative count 998240)
99.902% <= 0.327 milliseconds (cumulative count 999303)
99.951% <= 0.335 milliseconds (cumulative count 999528)
99.976% <= 0.367 milliseconds (cumulative count 999796)
99.988% <= 0.391 milliseconds (cumulative count 999881)
99.994% <= 0.439 milliseconds (cumulative count 999946)
99.997% <= 0.519 milliseconds (cumulative count 999974)
99.998% <= 0.535 milliseconds (cumulative count 999990)
99.999% <= 0.583 milliseconds (cumulative count 999993)
100.000% <= 0.799 milliseconds (cumulative count 999997)
100.000% <= 0.815 milliseconds (cumulative count 999999)
100.000% <= 0.839 milliseconds (cumulative count 1000000)
100.000% <= 0.839 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.725% <= 0.103 milliseconds (cumulative count 7248)
96.693% <= 0.207 milliseconds (cumulative count 966931)
98.501% <= 0.303 milliseconds (cumulative count 985008)
99.991% <= 0.407 milliseconds (cumulative count 999911)
99.997% <= 0.503 milliseconds (cumulative count 999969)
99.999% <= 0.607 milliseconds (cumulative count 999993)
100.000% <= 0.703 milliseconds (cumulative count 999996)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285632.69 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.159     0.024     0.159     0.199     0.311     0.839
====== RPUSH ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 14)
50.000% <= 0.159 milliseconds (cumulative count 882051)
93.750% <= 0.191 milliseconds (cumulative count 956027)
96.875% <= 0.215 milliseconds (cumulative count 969298)
98.438% <= 0.303 milliseconds (cumulative count 988698)
99.219% <= 0.311 milliseconds (cumulative count 996853)
99.805% <= 0.319 milliseconds (cumulative count 999217)
99.951% <= 0.327 milliseconds (cumulative count 999560)
99.976% <= 0.351 milliseconds (cumulative count 999797)
99.988% <= 0.375 milliseconds (cumulative count 999908)
99.994% <= 0.391 milliseconds (cumulative count 999941)
99.997% <= 0.471 milliseconds (cumulative count 999971)
99.998% <= 0.519 milliseconds (cumulative count 999989)
99.999% <= 0.527 milliseconds (cumulative count 999998)
100.000% <= 0.535 milliseconds (cumulative count 1000000)
100.000% <= 0.535 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.579% <= 0.103 milliseconds (cumulative count 5793)
96.843% <= 0.207 milliseconds (cumulative count 968430)
98.870% <= 0.303 milliseconds (cumulative count 988698)
99.995% <= 0.407 milliseconds (cumulative count 999951)
99.997% <= 0.503 milliseconds (cumulative count 999971)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307692.31 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.158     0.024     0.159     0.191     0.311     0.535
====== LPOP ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 5)
50.000% <= 0.159 milliseconds (cumulative count 632923)
75.000% <= 0.167 milliseconds (cumulative count 903696)
93.750% <= 0.199 milliseconds (cumulative count 951260)
96.875% <= 0.215 milliseconds (cumulative count 969470)
98.438% <= 0.311 milliseconds (cumulative count 985354)
99.219% <= 0.319 milliseconds (cumulative count 994183)
99.609% <= 0.327 milliseconds (cumulative count 998474)
99.902% <= 0.335 milliseconds (cumulative count 999449)
99.951% <= 0.343 milliseconds (cumulative count 999642)
99.976% <= 0.359 milliseconds (cumulative count 999790)
99.988% <= 0.415 milliseconds (cumulative count 999880)
99.994% <= 0.535 milliseconds (cumulative count 999940)
99.997% <= 0.575 milliseconds (cumulative count 999973)
99.998% <= 0.591 milliseconds (cumulative count 999987)
99.999% <= 0.703 milliseconds (cumulative count 999993)
100.000% <= 0.743 milliseconds (cumulative count 999997)
100.000% <= 1.031 milliseconds (cumulative count 1000000)
100.000% <= 1.031 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.491% <= 0.103 milliseconds (cumulative count 4915)
96.580% <= 0.207 milliseconds (cumulative count 965800)
97.785% <= 0.303 milliseconds (cumulative count 977846)
99.987% <= 0.407 milliseconds (cumulative count 999875)
99.993% <= 0.503 milliseconds (cumulative count 999929)
99.999% <= 0.607 milliseconds (cumulative count 999992)
99.999% <= 0.703 milliseconds (cumulative count 999993)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285632.69 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.163     0.024     0.159     0.199     0.319     1.031
====== RPOP ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 10)
50.000% <= 0.159 milliseconds (cumulative count 783755)
87.500% <= 0.167 milliseconds (cumulative count 916389)
93.750% <= 0.191 milliseconds (cumulative count 940386)
96.875% <= 0.215 milliseconds (cumulative count 969206)
98.438% <= 0.311 milliseconds (cumulative count 990744)
99.219% <= 0.319 milliseconds (cumulative count 997630)
99.805% <= 0.327 milliseconds (cumulative count 999359)
99.951% <= 0.335 milliseconds (cumulative count 999591)
99.976% <= 0.359 milliseconds (cumulative count 999781)
99.988% <= 0.391 milliseconds (cumulative count 999880)
99.994% <= 0.447 milliseconds (cumulative count 999941)
99.997% <= 0.495 milliseconds (cumulative count 999974)
99.998% <= 0.535 milliseconds (cumulative count 999989)
99.999% <= 0.615 milliseconds (cumulative count 999993)
100.000% <= 0.663 milliseconds (cumulative count 999997)
100.000% <= 0.879 milliseconds (cumulative count 999999)
100.000% <= 0.903 milliseconds (cumulative count 1000000)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.578% <= 0.103 milliseconds (cumulative count 5781)
96.804% <= 0.207 milliseconds (cumulative count 968039)
98.095% <= 0.303 milliseconds (cumulative count 980948)
99.990% <= 0.407 milliseconds (cumulative count 999905)
99.998% <= 0.503 milliseconds (cumulative count 999980)
99.999% <= 0.607 milliseconds (cumulative count 999992)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.161     0.024     0.159     0.199     0.311     0.903
====== SADD ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 34)
50.000% <= 0.151 milliseconds (cumulative count 574419)
75.000% <= 0.159 milliseconds (cumulative count 903703)
93.750% <= 0.183 milliseconds (cumulative count 944283)
96.875% <= 0.207 milliseconds (cumulative count 969130)
98.438% <= 0.303 milliseconds (cumulative count 993206)
99.609% <= 0.311 milliseconds (cumulative count 998225)
99.902% <= 0.319 milliseconds (cumulative count 999442)
99.951% <= 0.327 milliseconds (cumulative count 999646)
99.976% <= 0.343 milliseconds (cumulative count 999810)
99.988% <= 0.367 milliseconds (cumulative count 999893)
99.994% <= 0.391 milliseconds (cumulative count 999941)
99.997% <= 0.447 milliseconds (cumulative count 999992)
99.999% <= 0.455 milliseconds (cumulative count 999994)
100.000% <= 0.503 milliseconds (cumulative count 999998)
100.000% <= 0.599 milliseconds (cumulative count 999999)
100.000% <= 0.607 milliseconds (cumulative count 1000000)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.789% <= 0.103 milliseconds (cumulative count 7890)
96.913% <= 0.207 milliseconds (cumulative count 969130)
99.321% <= 0.303 milliseconds (cumulative count 993206)
99.995% <= 0.407 milliseconds (cumulative count 999947)
100.000% <= 0.503 milliseconds (cumulative count 999998)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307503.06 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.156     0.024     0.151     0.191     0.303     0.607
====== HSET ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 19)
50.000% <= 0.159 milliseconds (cumulative count 857212)
87.500% <= 0.167 milliseconds (cumulative count 918966)
93.750% <= 0.191 milliseconds (cumulative count 950651)
96.875% <= 0.215 milliseconds (cumulative count 969155)
98.438% <= 0.303 milliseconds (cumulative count 985898)
99.219% <= 0.311 milliseconds (cumulative count 994868)
99.609% <= 0.319 milliseconds (cumulative count 998787)
99.902% <= 0.327 milliseconds (cumulative count 999449)
99.951% <= 0.335 milliseconds (cumulative count 999564)
99.976% <= 0.359 milliseconds (cumulative count 999760)
99.988% <= 0.391 milliseconds (cumulative count 999886)
99.994% <= 0.407 milliseconds (cumulative count 999947)
99.997% <= 0.479 milliseconds (cumulative count 999971)
99.998% <= 0.607 milliseconds (cumulative count 999985)
99.999% <= 0.639 milliseconds (cumulative count 999999)
100.000% <= 0.655 milliseconds (cumulative count 1000000)
100.000% <= 0.655 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.647% <= 0.103 milliseconds (cumulative count 6470)
96.822% <= 0.207 milliseconds (cumulative count 968219)
98.590% <= 0.303 milliseconds (cumulative count 985898)
99.995% <= 0.407 milliseconds (cumulative count 999947)
99.998% <= 0.503 milliseconds (cumulative count 999977)
99.999% <= 0.607 milliseconds (cumulative count 999985)
100.000% <= 0.703 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307692.31 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.159     0.024     0.159     0.191     0.311     0.655
====== SPOP ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 28)
50.000% <= 0.151 milliseconds (cumulative count 710096)
75.000% <= 0.159 milliseconds (cumulative count 912731)
93.750% <= 0.183 milliseconds (cumulative count 953607)
96.875% <= 0.207 milliseconds (cumulative count 969654)
98.438% <= 0.295 milliseconds (cumulative count 987825)
99.219% <= 0.303 milliseconds (cumulative count 996281)
99.805% <= 0.311 milliseconds (cumulative count 999046)
99.951% <= 0.327 milliseconds (cumulative count 999616)
99.976% <= 0.343 milliseconds (cumulative count 999775)
99.988% <= 0.367 milliseconds (cumulative count 999909)
99.994% <= 0.383 milliseconds (cumulative count 999950)
99.997% <= 0.407 milliseconds (cumulative count 999978)
99.998% <= 0.471 milliseconds (cumulative count 999985)
99.999% <= 0.503 milliseconds (cumulative count 999993)
100.000% <= 0.607 milliseconds (cumulative count 999997)
100.000% <= 0.695 milliseconds (cumulative count 999999)
100.000% <= 0.743 milliseconds (cumulative count 1000000)
100.000% <= 0.743 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.737% <= 0.103 milliseconds (cumulative count 7366)
96.965% <= 0.207 milliseconds (cumulative count 969654)
99.628% <= 0.303 milliseconds (cumulative count 996281)
99.998% <= 0.407 milliseconds (cumulative count 999978)
99.999% <= 0.503 milliseconds (cumulative count 999993)
100.000% <= 0.607 milliseconds (cumulative count 999997)
100.000% <= 0.703 milliseconds (cumulative count 999999)
100.000% <= 0.807 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307597.66 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.154     0.024     0.151     0.183     0.303     0.743
====== ZADD ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 14)
50.000% <= 0.159 milliseconds (cumulative count 811384)
87.500% <= 0.167 milliseconds (cumulative count 914229)
93.750% <= 0.191 milliseconds (cumulative count 941643)
96.875% <= 0.223 milliseconds (cumulative count 969033)
98.438% <= 0.311 milliseconds (cumulative count 992745)
99.609% <= 0.319 milliseconds (cumulative count 998224)
99.902% <= 0.327 milliseconds (cumulative count 999468)
99.951% <= 0.335 milliseconds (cumulative count 999667)
99.976% <= 0.351 milliseconds (cumulative count 999804)
99.988% <= 0.367 milliseconds (cumulative count 999881)
99.994% <= 0.391 milliseconds (cumulative count 999939)
99.997% <= 0.431 milliseconds (cumulative count 999970)
99.998% <= 0.527 milliseconds (cumulative count 999987)
99.999% <= 0.535 milliseconds (cumulative count 999995)
100.000% <= 0.543 milliseconds (cumulative count 999997)
100.000% <= 0.575 milliseconds (cumulative count 999999)
100.000% <= 0.583 milliseconds (cumulative count 1000000)
100.000% <= 0.583 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.553% <= 0.103 milliseconds (cumulative count 5535)
96.730% <= 0.207 milliseconds (cumulative count 967304)
98.241% <= 0.303 milliseconds (cumulative count 982409)
99.996% <= 0.407 milliseconds (cumulative count 999960)
99.997% <= 0.503 milliseconds (cumulative count 999974)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.160     0.024     0.159     0.199     0.311     0.583
====== ZPOPMIN ======                                                     
  1000000 requests completed in 3.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 24)
50.000% <= 0.151 milliseconds (cumulative count 659240)
75.000% <= 0.159 milliseconds (cumulative count 903373)
93.750% <= 0.183 milliseconds (cumulative count 950007)
96.875% <= 0.215 milliseconds (cumulative count 969305)
98.438% <= 0.295 milliseconds (cumulative count 985661)
99.219% <= 0.303 milliseconds (cumulative count 994947)
99.609% <= 0.311 milliseconds (cumulative count 998549)
99.902% <= 0.319 milliseconds (cumulative count 999091)
99.951% <= 0.351 milliseconds (cumulative count 999544)
99.976% <= 0.367 milliseconds (cumulative count 999788)
99.988% <= 0.399 milliseconds (cumulative count 999880)
99.994% <= 0.415 milliseconds (cumulative count 999942)
99.997% <= 0.471 milliseconds (cumulative count 999970)
99.998% <= 0.511 milliseconds (cumulative count 999991)
99.999% <= 0.599 milliseconds (cumulative count 999995)
100.000% <= 0.735 milliseconds (cumulative count 999997)
100.000% <= 0.823 milliseconds (cumulative count 999999)
100.000% <= 0.871 milliseconds (cumulative count 1000000)
100.000% <= 0.871 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.741% <= 0.103 milliseconds (cumulative count 7408)
96.842% <= 0.207 milliseconds (cumulative count 968421)
99.495% <= 0.303 milliseconds (cumulative count 994947)
99.992% <= 0.407 milliseconds (cumulative count 999916)
99.998% <= 0.503 milliseconds (cumulative count 999984)
100.000% <= 0.607 milliseconds (cumulative count 999996)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 307503.06 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.155     0.024     0.151     0.183     0.303     0.871
====== LPUSH (needed to benchmark LRANGE) ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 12)
50.000% <= 0.159 milliseconds (cumulative count 831465)
87.500% <= 0.167 milliseconds (cumulative count 916066)
93.750% <= 0.191 milliseconds (cumulative count 947070)
96.875% <= 0.215 milliseconds (cumulative count 969288)
98.438% <= 0.311 milliseconds (cumulative count 993644)
99.609% <= 0.319 milliseconds (cumulative count 998413)
99.902% <= 0.327 milliseconds (cumulative count 999431)
99.951% <= 0.335 milliseconds (cumulative count 999629)
99.976% <= 0.351 milliseconds (cumulative count 999777)
99.988% <= 0.375 milliseconds (cumulative count 999882)
99.994% <= 0.431 milliseconds (cumulative count 999941)
99.997% <= 0.519 milliseconds (cumulative count 999972)
99.998% <= 0.543 milliseconds (cumulative count 999986)
99.999% <= 0.583 milliseconds (cumulative count 999993)
100.000% <= 0.735 milliseconds (cumulative count 999997)
100.000% <= 0.759 milliseconds (cumulative count 999999)
100.000% <= 0.879 milliseconds (cumulative count 1000000)
100.000% <= 0.879 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.566% <= 0.103 milliseconds (cumulative count 5664)
96.834% <= 0.207 milliseconds (cumulative count 968344)
98.421% <= 0.303 milliseconds (cumulative count 984215)
99.992% <= 0.407 milliseconds (cumulative count 999924)
99.997% <= 0.503 milliseconds (cumulative count 999968)
100.000% <= 0.607 milliseconds (cumulative count 999996)
100.000% <= 0.807 milliseconds (cumulative count 999999)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.160     0.024     0.159     0.199     0.311     0.879
====== LRANGE_100 (first 100 elements) ======                                                     
  1000000 requests completed in 4.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.055 milliseconds (cumulative count 6)
50.000% <= 0.199 milliseconds (cumulative count 522393)
75.000% <= 0.239 milliseconds (cumulative count 767506)
87.500% <= 0.263 milliseconds (cumulative count 904448)
93.750% <= 0.271 milliseconds (cumulative count 939504)
96.875% <= 0.287 milliseconds (cumulative count 976596)
98.438% <= 0.311 milliseconds (cumulative count 986532)
99.219% <= 0.327 milliseconds (cumulative count 993163)
99.609% <= 0.335 milliseconds (cumulative count 996154)
99.805% <= 0.343 milliseconds (cumulative count 998260)
99.902% <= 0.351 milliseconds (cumulative count 999304)
99.951% <= 0.359 milliseconds (cumulative count 999688)
99.976% <= 0.367 milliseconds (cumulative count 999792)
99.988% <= 0.391 milliseconds (cumulative count 999897)
99.994% <= 0.447 milliseconds (cumulative count 999948)
99.997% <= 0.511 milliseconds (cumulative count 999973)
99.998% <= 0.599 milliseconds (cumulative count 999985)
99.999% <= 0.647 milliseconds (cumulative count 999993)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.839 milliseconds (cumulative count 999999)
100.000% <= 0.879 milliseconds (cumulative count 1000000)
100.000% <= 0.879 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.115% <= 0.103 milliseconds (cumulative count 1147)
58.097% <= 0.207 milliseconds (cumulative count 580971)
98.398% <= 0.303 milliseconds (cumulative count 983979)
99.991% <= 0.407 milliseconds (cumulative count 999914)
99.997% <= 0.503 milliseconds (cumulative count 999968)
99.999% <= 0.607 milliseconds (cumulative count 999986)
99.999% <= 0.703 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 210393.45 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.209     0.048     0.199     0.279     0.327     0.879
====== LRANGE_300 (first 300 elements) ======                                                     
  1000000 requests completed in 7.76 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.063 milliseconds (cumulative count 2)
50.000% <= 0.279 milliseconds (cumulative count 537587)
75.000% <= 0.327 milliseconds (cumulative count 761655)
87.500% <= 0.375 milliseconds (cumulative count 877883)
93.750% <= 0.415 milliseconds (cumulative count 942294)
96.875% <= 0.447 milliseconds (cumulative count 971925)
98.438% <= 0.471 milliseconds (cumulative count 988166)
99.219% <= 0.487 milliseconds (cumulative count 995403)
99.609% <= 0.495 milliseconds (cumulative count 997117)
99.805% <= 0.511 milliseconds (cumulative count 998381)
99.902% <= 0.551 milliseconds (cumulative count 999076)
99.951% <= 0.631 milliseconds (cumulative count 999527)
99.976% <= 0.711 milliseconds (cumulative count 999772)
99.988% <= 0.759 milliseconds (cumulative count 999886)
99.994% <= 0.791 milliseconds (cumulative count 999951)
99.997% <= 0.823 milliseconds (cumulative count 999975)
99.998% <= 0.847 milliseconds (cumulative count 999985)
99.999% <= 0.887 milliseconds (cumulative count 999993)
100.000% <= 0.895 milliseconds (cumulative count 999997)
100.000% <= 0.927 milliseconds (cumulative count 999999)
100.000% <= 0.943 milliseconds (cumulative count 1000000)
100.000% <= 0.943 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.165% <= 0.103 milliseconds (cumulative count 1652)
17.500% <= 0.207 milliseconds (cumulative count 175001)
65.525% <= 0.303 milliseconds (cumulative count 655247)
93.211% <= 0.407 milliseconds (cumulative count 932110)
99.793% <= 0.503 milliseconds (cumulative count 997929)
99.942% <= 0.607 milliseconds (cumulative count 999422)
99.975% <= 0.703 milliseconds (cumulative count 999753)
99.997% <= 0.807 milliseconds (cumulative count 999967)
100.000% <= 0.903 milliseconds (cumulative count 999997)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 128949.06 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.281     0.056     0.279     0.423     0.479     0.943
====== LRANGE_500 (first 500 elements) ======                                                   
  1000000 requests completed in 11.51 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.071 milliseconds (cumulative count 1)
50.000% <= 0.279 milliseconds (cumulative count 518624)
75.000% <= 0.311 milliseconds (cumulative count 849073)
87.500% <= 0.319 milliseconds (cumulative count 886846)
93.750% <= 0.343 milliseconds (cumulative count 971798)
98.438% <= 0.367 milliseconds (cumulative count 989571)
99.219% <= 0.375 milliseconds (cumulative count 993818)
99.609% <= 0.399 milliseconds (cumulative count 996269)
99.805% <= 0.575 milliseconds (cumulative count 998050)
99.902% <= 0.807 milliseconds (cumulative count 999030)
99.951% <= 0.935 milliseconds (cumulative count 999521)
99.976% <= 1.031 milliseconds (cumulative count 999760)
99.988% <= 1.103 milliseconds (cumulative count 999886)
99.994% <= 1.151 milliseconds (cumulative count 999941)
99.997% <= 1.199 milliseconds (cumulative count 999972)
99.998% <= 1.231 milliseconds (cumulative count 999985)
99.999% <= 1.263 milliseconds (cumulative count 999993)
100.000% <= 1.279 milliseconds (cumulative count 999997)
100.000% <= 1.319 milliseconds (cumulative count 999999)
100.000% <= 1.327 milliseconds (cumulative count 1000000)
100.000% <= 1.327 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 2)
0.313% <= 0.207 milliseconds (cumulative count 3129)
70.168% <= 0.303 milliseconds (cumulative count 701683)
99.671% <= 0.407 milliseconds (cumulative count 996706)
99.775% <= 0.503 milliseconds (cumulative count 997751)
99.816% <= 0.607 milliseconds (cumulative count 998165)
99.856% <= 0.703 milliseconds (cumulative count 998560)
99.903% <= 0.807 milliseconds (cumulative count 999030)
99.942% <= 0.903 milliseconds (cumulative count 999425)
99.971% <= 1.007 milliseconds (cumulative count 999706)
99.989% <= 1.103 milliseconds (cumulative count 999886)
99.998% <= 1.207 milliseconds (cumulative count 999978)
100.000% <= 1.303 milliseconds (cumulative count 999997)
100.000% <= 1.407 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 86865.88 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.284     0.064     0.279     0.343     0.375     1.327
====== LRANGE_600 (first 600 elements) ======                                                   
  1000000 requests completed in 13.51 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.103 milliseconds (cumulative count 1)
50.000% <= 0.327 milliseconds (cumulative count 544465)
75.000% <= 0.359 milliseconds (cumulative count 765860)
87.500% <= 0.375 milliseconds (cumulative count 880350)
93.750% <= 0.399 milliseconds (cumulative count 960633)
96.875% <= 0.407 milliseconds (cumulative count 974489)
98.438% <= 0.431 milliseconds (cumulative count 989688)
99.219% <= 0.439 milliseconds (cumulative count 994689)
99.609% <= 0.455 milliseconds (cumulative count 996684)
99.805% <= 0.479 milliseconds (cumulative count 998412)
99.902% <= 0.559 milliseconds (cumulative count 999028)
99.951% <= 0.863 milliseconds (cumulative count 999516)
99.976% <= 1.023 milliseconds (cumulative count 999761)
99.988% <= 1.143 milliseconds (cumulative count 999884)
99.994% <= 1.247 milliseconds (cumulative count 999943)
99.997% <= 1.319 milliseconds (cumulative count 999971)
99.998% <= 1.367 milliseconds (cumulative count 999985)
99.999% <= 1.399 milliseconds (cumulative count 999993)
100.000% <= 1.447 milliseconds (cumulative count 999997)
100.000% <= 1.527 milliseconds (cumulative count 999999)
100.000% <= 1.639 milliseconds (cumulative count 1000000)
100.000% <= 1.639 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 1)
0.041% <= 0.207 milliseconds (cumulative count 413)
28.800% <= 0.303 milliseconds (cumulative count 288001)
97.449% <= 0.407 milliseconds (cumulative count 974489)
99.872% <= 0.503 milliseconds (cumulative count 998722)
99.914% <= 0.607 milliseconds (cumulative count 999139)
99.928% <= 0.703 milliseconds (cumulative count 999278)
99.943% <= 0.807 milliseconds (cumulative count 999434)
99.958% <= 0.903 milliseconds (cumulative count 999579)
99.974% <= 1.007 milliseconds (cumulative count 999741)
99.984% <= 1.103 milliseconds (cumulative count 999845)
99.992% <= 1.207 milliseconds (cumulative count 999925)
99.996% <= 1.303 milliseconds (cumulative count 999963)
99.999% <= 1.407 milliseconds (cumulative count 999993)
100.000% <= 1.503 milliseconds (cumulative count 999998)
100.000% <= 1.607 milliseconds (cumulative count 999999)
100.000% <= 1.703 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 73997.34 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.331     0.096     0.327     0.399     0.439     1.639
====== MSET (10 keys) ======                                                     
  1000000 requests completed in 4.00 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 1)
50.000% <= 0.183 milliseconds (cumulative count 613274)
75.000% <= 0.191 milliseconds (cumulative count 907843)
93.750% <= 0.239 milliseconds (cumulative count 941194)
96.875% <= 0.255 milliseconds (cumulative count 970162)
98.438% <= 0.367 milliseconds (cumulative count 994004)
99.609% <= 0.375 milliseconds (cumulative count 998821)
99.902% <= 0.383 milliseconds (cumulative count 999551)
99.976% <= 0.399 milliseconds (cumulative count 999796)
99.988% <= 0.415 milliseconds (cumulative count 999878)
99.994% <= 0.463 milliseconds (cumulative count 999942)
99.997% <= 0.543 milliseconds (cumulative count 999972)
99.998% <= 0.567 milliseconds (cumulative count 999985)
99.999% <= 0.663 milliseconds (cumulative count 999993)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.815 milliseconds (cumulative count 999999)
100.000% <= 1.007 milliseconds (cumulative count 1000000)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.062% <= 0.103 milliseconds (cumulative count 624)
92.616% <= 0.207 milliseconds (cumulative count 926156)
97.261% <= 0.303 milliseconds (cumulative count 972610)
99.985% <= 0.407 milliseconds (cumulative count 999848)
99.995% <= 0.503 milliseconds (cumulative count 999948)
99.999% <= 0.607 milliseconds (cumulative count 999988)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 249875.08 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.189     0.024     0.183     0.247     0.367     1.007
====== XADD ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.039 milliseconds (cumulative count 20)
50.000% <= 0.175 milliseconds (cumulative count 879940)
93.750% <= 0.215 milliseconds (cumulative count 944040)
96.875% <= 0.231 milliseconds (cumulative count 970645)
98.438% <= 0.335 milliseconds (cumulative count 987215)
99.219% <= 0.343 milliseconds (cumulative count 996304)
99.805% <= 0.351 milliseconds (cumulative count 999081)
99.951% <= 0.367 milliseconds (cumulative count 999632)
99.976% <= 0.383 milliseconds (cumulative count 999773)
99.988% <= 0.415 milliseconds (cumulative count 999882)
99.994% <= 0.455 milliseconds (cumulative count 999949)
99.997% <= 0.471 milliseconds (cumulative count 999978)
99.998% <= 0.535 milliseconds (cumulative count 999985)
99.999% <= 0.615 milliseconds (cumulative count 999994)
100.000% <= 0.823 milliseconds (cumulative count 999997)
100.000% <= 0.831 milliseconds (cumulative count 999999)
100.000% <= 0.847 milliseconds (cumulative count 1000000)
100.000% <= 0.847 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.185% <= 0.103 milliseconds (cumulative count 1850)
93.619% <= 0.207 milliseconds (cumulative count 936194)
97.345% <= 0.303 milliseconds (cumulative count 973448)
99.986% <= 0.407 milliseconds (cumulative count 999857)
99.998% <= 0.503 milliseconds (cumulative count 999984)
99.999% <= 0.607 milliseconds (cumulative count 999992)
99.999% <= 0.703 milliseconds (cumulative count 999995)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.175     0.032     0.175     0.223     0.343     0.847
  ```
</details>

<details>
  <summary>Instrumented</summary>

  ```
====== PING_INLINE ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 3)
50.000% <= 0.167 milliseconds (cumulative count 721305)
75.000% <= 0.175 milliseconds (cumulative count 909165)
93.750% <= 0.207 milliseconds (cumulative count 939905)
96.875% <= 0.223 milliseconds (cumulative count 969389)
98.438% <= 0.327 milliseconds (cumulative count 985980)
99.219% <= 0.335 milliseconds (cumulative count 996425)
99.805% <= 0.343 milliseconds (cumulative count 999125)
99.951% <= 0.367 milliseconds (cumulative count 999583)
99.976% <= 0.399 milliseconds (cumulative count 999775)
99.988% <= 0.415 milliseconds (cumulative count 999944)
99.997% <= 0.431 milliseconds (cumulative count 999979)
99.998% <= 0.455 milliseconds (cumulative count 999987)
99.999% <= 0.551 milliseconds (cumulative count 999994)
100.000% <= 0.559 milliseconds (cumulative count 999997)
100.000% <= 0.567 milliseconds (cumulative count 999999)
100.000% <= 0.615 milliseconds (cumulative count 1000000)
100.000% <= 0.615 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.206% <= 0.103 milliseconds (cumulative count 2057)
93.990% <= 0.207 milliseconds (cumulative count 939905)
97.339% <= 0.303 milliseconds (cumulative count 973394)
99.986% <= 0.407 milliseconds (cumulative count 999864)
99.999% <= 0.503 milliseconds (cumulative count 999991)
100.000% <= 0.607 milliseconds (cumulative count 999999)
100.000% <= 0.703 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.171     0.024     0.167     0.215     0.335     0.615
====== PING_MBULK ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 3)
50.000% <= 0.159 milliseconds (cumulative count 549075)
75.000% <= 0.167 milliseconds (cumulative count 907921)
93.750% <= 0.199 milliseconds (cumulative count 948334)
96.875% <= 0.215 milliseconds (cumulative count 970346)
98.438% <= 0.319 milliseconds (cumulative count 992754)
99.609% <= 0.327 milliseconds (cumulative count 998685)
99.902% <= 0.335 milliseconds (cumulative count 999410)
99.951% <= 0.343 milliseconds (cumulative count 999574)
99.976% <= 0.359 milliseconds (cumulative count 999757)
99.988% <= 0.399 milliseconds (cumulative count 999890)
99.994% <= 0.439 milliseconds (cumulative count 999944)
99.997% <= 0.463 milliseconds (cumulative count 999972)
99.998% <= 0.559 milliseconds (cumulative count 999985)
99.999% <= 0.647 milliseconds (cumulative count 999993)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.847 milliseconds (cumulative count 999999)
100.000% <= 0.879 milliseconds (cumulative count 1000000)
100.000% <= 0.879 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.400% <= 0.103 milliseconds (cumulative count 3996)
96.752% <= 0.207 milliseconds (cumulative count 967522)
97.618% <= 0.303 milliseconds (cumulative count 976178)
99.990% <= 0.407 milliseconds (cumulative count 999901)
99.998% <= 0.503 milliseconds (cumulative count 999981)
99.999% <= 0.607 milliseconds (cumulative count 999988)
100.000% <= 0.703 milliseconds (cumulative count 999997)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285632.69 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.165     0.024     0.159     0.207     0.319     0.879
====== SET ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.047 milliseconds (cumulative count 17)
50.000% <= 0.175 milliseconds (cumulative count 811932)
87.500% <= 0.183 milliseconds (cumulative count 918213)
93.750% <= 0.215 milliseconds (cumulative count 937617)
96.875% <= 0.239 milliseconds (cumulative count 970542)
98.438% <= 0.343 milliseconds (cumulative count 990474)
99.219% <= 0.351 milliseconds (cumulative count 998196)
99.902% <= 0.359 milliseconds (cumulative count 999342)
99.951% <= 0.367 milliseconds (cumulative count 999516)
99.976% <= 0.391 milliseconds (cumulative count 999777)
99.988% <= 0.431 milliseconds (cumulative count 999878)
99.994% <= 0.463 milliseconds (cumulative count 999943)
99.997% <= 0.487 milliseconds (cumulative count 999973)
99.998% <= 0.575 milliseconds (cumulative count 999986)
99.999% <= 0.663 milliseconds (cumulative count 999993)
100.000% <= 0.775 milliseconds (cumulative count 999997)
100.000% <= 0.895 milliseconds (cumulative count 999999)
100.000% <= 0.903 milliseconds (cumulative count 1000000)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.129% <= 0.103 milliseconds (cumulative count 1293)
93.305% <= 0.207 milliseconds (cumulative count 933045)
97.295% <= 0.303 milliseconds (cumulative count 972951)
99.984% <= 0.407 milliseconds (cumulative count 999836)
99.998% <= 0.503 milliseconds (cumulative count 999978)
99.999% <= 0.607 milliseconds (cumulative count 999986)
99.999% <= 0.703 milliseconds (cumulative count 999995)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266666.66 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.178     0.040     0.175     0.231     0.343     0.903
====== GET ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.039 milliseconds (cumulative count 21)
50.000% <= 0.167 milliseconds (cumulative count 633423)
75.000% <= 0.175 milliseconds (cumulative count 911898)
93.750% <= 0.207 milliseconds (cumulative count 938300)
96.875% <= 0.231 milliseconds (cumulative count 969332)
98.438% <= 0.335 milliseconds (cumulative count 994205)
99.609% <= 0.343 milliseconds (cumulative count 998693)
99.902% <= 0.351 milliseconds (cumulative count 999245)
99.951% <= 0.367 milliseconds (cumulative count 999632)
99.976% <= 0.391 milliseconds (cumulative count 999775)
99.988% <= 0.463 milliseconds (cumulative count 999890)
99.994% <= 0.535 milliseconds (cumulative count 999939)
99.997% <= 0.599 milliseconds (cumulative count 999974)
99.998% <= 0.615 milliseconds (cumulative count 999988)
99.999% <= 0.663 milliseconds (cumulative count 999993)
100.000% <= 0.743 milliseconds (cumulative count 999997)
100.000% <= 0.855 milliseconds (cumulative count 999999)
100.000% <= 1.039 milliseconds (cumulative count 1000000)
100.000% <= 1.039 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.225% <= 0.103 milliseconds (cumulative count 2254)
93.830% <= 0.207 milliseconds (cumulative count 938300)
97.285% <= 0.303 milliseconds (cumulative count 972854)
99.981% <= 0.407 milliseconds (cumulative count 999806)
99.992% <= 0.503 milliseconds (cumulative count 999925)
99.998% <= 0.607 milliseconds (cumulative count 999982)
100.000% <= 0.703 milliseconds (cumulative count 999996)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266666.66 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.172     0.032     0.167     0.215     0.335     1.039
====== INCR ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 1)
50.000% <= 0.167 milliseconds (cumulative count 540900)
75.000% <= 0.175 milliseconds (cumulative count 908590)
93.750% <= 0.215 milliseconds (cumulative count 952347)
96.875% <= 0.223 milliseconds (cumulative count 969212)
98.438% <= 0.335 milliseconds (cumulative count 992560)
99.609% <= 0.343 milliseconds (cumulative count 998706)
99.902% <= 0.351 milliseconds (cumulative count 999316)
99.951% <= 0.367 milliseconds (cumulative count 999537)
99.976% <= 0.407 milliseconds (cumulative count 999792)
99.988% <= 0.415 milliseconds (cumulative count 999895)
99.994% <= 0.431 milliseconds (cumulative count 999949)
99.997% <= 0.447 milliseconds (cumulative count 999971)
99.998% <= 0.471 milliseconds (cumulative count 999986)
99.999% <= 0.503 milliseconds (cumulative count 999993)
100.000% <= 0.551 milliseconds (cumulative count 999999)
100.000% <= 0.703 milliseconds (cumulative count 1000000)
100.000% <= 0.703 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.200% <= 0.103 milliseconds (cumulative count 2001)
93.724% <= 0.207 milliseconds (cumulative count 937242)
97.410% <= 0.303 milliseconds (cumulative count 974099)
99.979% <= 0.407 milliseconds (cumulative count 999792)
99.999% <= 0.503 milliseconds (cumulative count 999993)
100.000% <= 0.607 milliseconds (cumulative count 999999)
100.000% <= 0.703 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.173     0.024     0.167     0.215     0.335     0.703
====== LPUSH ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 2)
50.000% <= 0.175 milliseconds (cumulative count 570394)
75.000% <= 0.183 milliseconds (cumulative count 904803)
93.750% <= 0.223 milliseconds (cumulative count 938523)
96.875% <= 0.247 milliseconds (cumulative count 969699)
98.438% <= 0.351 milliseconds (cumulative count 992589)
99.609% <= 0.359 milliseconds (cumulative count 998322)
99.902% <= 0.367 milliseconds (cumulative count 999278)
99.951% <= 0.383 milliseconds (cumulative count 999646)
99.976% <= 0.399 milliseconds (cumulative count 999798)
99.988% <= 0.439 milliseconds (cumulative count 999883)
99.994% <= 0.495 milliseconds (cumulative count 999947)
99.997% <= 0.599 milliseconds (cumulative count 999972)
99.998% <= 0.623 milliseconds (cumulative count 999985)
99.999% <= 0.735 milliseconds (cumulative count 999994)
100.000% <= 0.903 milliseconds (cumulative count 999997)
100.000% <= 0.927 milliseconds (cumulative count 999999)
100.000% <= 1.047 milliseconds (cumulative count 1000000)
100.000% <= 1.047 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.098% <= 0.103 milliseconds (cumulative count 978)
92.903% <= 0.207 milliseconds (cumulative count 929033)
97.198% <= 0.303 milliseconds (cumulative count 971982)
99.984% <= 0.407 milliseconds (cumulative count 999838)
99.995% <= 0.503 milliseconds (cumulative count 999952)
99.998% <= 0.607 milliseconds (cumulative count 999976)
99.999% <= 0.703 milliseconds (cumulative count 999992)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 999997)
100.000% <= 1.007 milliseconds (cumulative count 999999)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.182     0.024     0.175     0.231     0.351     1.047
====== RPUSH ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 1)
50.000% <= 0.175 milliseconds (cumulative count 721666)
75.000% <= 0.183 milliseconds (cumulative count 917458)
93.750% <= 0.223 milliseconds (cumulative count 944744)
96.875% <= 0.239 milliseconds (cumulative count 971091)
98.438% <= 0.343 milliseconds (cumulative count 986121)
99.219% <= 0.351 milliseconds (cumulative count 996600)
99.805% <= 0.359 milliseconds (cumulative count 999242)
99.951% <= 0.367 milliseconds (cumulative count 999557)
99.976% <= 0.383 milliseconds (cumulative count 999773)
99.988% <= 0.407 milliseconds (cumulative count 999887)
99.994% <= 0.463 milliseconds (cumulative count 999943)
99.997% <= 0.487 milliseconds (cumulative count 999976)
99.998% <= 0.535 milliseconds (cumulative count 999986)
99.999% <= 0.695 milliseconds (cumulative count 999993)
100.000% <= 0.799 milliseconds (cumulative count 999997)
100.000% <= 0.855 milliseconds (cumulative count 999999)
100.000% <= 0.863 milliseconds (cumulative count 1000000)
100.000% <= 0.863 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.117% <= 0.103 milliseconds (cumulative count 1169)
93.359% <= 0.207 milliseconds (cumulative count 933588)
97.274% <= 0.303 milliseconds (cumulative count 972742)
99.989% <= 0.407 milliseconds (cumulative count 999887)
99.998% <= 0.503 milliseconds (cumulative count 999983)
99.999% <= 0.607 milliseconds (cumulative count 999989)
99.999% <= 0.703 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.179     0.024     0.175     0.231     0.351     0.863
====== LPOP ======                                                     
  1000000 requests completed in 4.00 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.047 milliseconds (cumulative count 5)
50.000% <= 0.183 milliseconds (cumulative count 638520)
75.000% <= 0.191 milliseconds (cumulative count 909438)
93.750% <= 0.231 milliseconds (cumulative count 938615)
96.875% <= 0.255 milliseconds (cumulative count 969301)
98.438% <= 0.367 milliseconds (cumulative count 993979)
99.609% <= 0.375 milliseconds (cumulative count 998675)
99.902% <= 0.383 milliseconds (cumulative count 999564)
99.976% <= 0.399 milliseconds (cumulative count 999800)
99.988% <= 0.431 milliseconds (cumulative count 999890)
99.994% <= 0.519 milliseconds (cumulative count 999943)
99.997% <= 0.551 milliseconds (cumulative count 999976)
99.998% <= 0.583 milliseconds (cumulative count 999986)
99.999% <= 0.727 milliseconds (cumulative count 999993)
100.000% <= 0.783 milliseconds (cumulative count 999997)
100.000% <= 0.991 milliseconds (cumulative count 999999)
100.000% <= 1.007 milliseconds (cumulative count 1000000)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.048% <= 0.103 milliseconds (cumulative count 483)
92.846% <= 0.207 milliseconds (cumulative count 928458)
97.197% <= 0.303 milliseconds (cumulative count 971966)
99.984% <= 0.407 milliseconds (cumulative count 999842)
99.993% <= 0.503 milliseconds (cumulative count 999931)
99.999% <= 0.607 milliseconds (cumulative count 999986)
99.999% <= 0.703 milliseconds (cumulative count 999992)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 249875.08 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.189     0.040     0.183     0.247     0.367     1.007
====== RPOP ======                                                     
  1000000 requests completed in 4.00 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.039 milliseconds (cumulative count 2)
50.000% <= 0.183 milliseconds (cumulative count 804191)
87.500% <= 0.191 milliseconds (cumulative count 917235)
93.750% <= 0.231 milliseconds (cumulative count 940374)
96.875% <= 0.255 milliseconds (cumulative count 971292)
98.438% <= 0.359 milliseconds (cumulative count 990672)
99.219% <= 0.367 milliseconds (cumulative count 998178)
99.902% <= 0.375 milliseconds (cumulative count 999558)
99.976% <= 0.391 milliseconds (cumulative count 999862)
99.988% <= 0.399 milliseconds (cumulative count 999928)
99.994% <= 0.407 milliseconds (cumulative count 999959)
99.997% <= 0.415 milliseconds (cumulative count 999982)
99.998% <= 0.423 milliseconds (cumulative count 999987)
99.999% <= 0.439 milliseconds (cumulative count 999995)
100.000% <= 0.447 milliseconds (cumulative count 999998)
100.000% <= 0.503 milliseconds (cumulative count 1000000)
100.000% <= 0.503 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.052% <= 0.103 milliseconds (cumulative count 516)
92.826% <= 0.207 milliseconds (cumulative count 928264)
97.221% <= 0.303 milliseconds (cumulative count 972205)
99.996% <= 0.407 milliseconds (cumulative count 999959)
100.000% <= 0.503 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 249875.08 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.186     0.032     0.183     0.239     0.359     0.503
====== SADD ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.039 milliseconds (cumulative count 21)
50.000% <= 0.175 milliseconds (cumulative count 888755)
93.750% <= 0.215 milliseconds (cumulative count 942757)
96.875% <= 0.231 milliseconds (cumulative count 970586)
98.438% <= 0.335 milliseconds (cumulative count 984662)
99.219% <= 0.343 milliseconds (cumulative count 995762)
99.609% <= 0.351 milliseconds (cumulative count 999020)
99.902% <= 0.359 milliseconds (cumulative count 999334)
99.951% <= 0.375 milliseconds (cumulative count 999612)
99.976% <= 0.399 milliseconds (cumulative count 999761)
99.988% <= 0.479 milliseconds (cumulative count 999878)
99.994% <= 0.511 milliseconds (cumulative count 999946)
99.997% <= 0.567 milliseconds (cumulative count 999970)
99.998% <= 0.639 milliseconds (cumulative count 999985)
99.999% <= 0.695 milliseconds (cumulative count 999993)
100.000% <= 0.783 milliseconds (cumulative count 999997)
100.000% <= 0.983 milliseconds (cumulative count 999999)
100.000% <= 1.015 milliseconds (cumulative count 1000000)
100.000% <= 1.015 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.136% <= 0.103 milliseconds (cumulative count 1356)
93.625% <= 0.207 milliseconds (cumulative count 936250)
97.291% <= 0.303 milliseconds (cumulative count 972909)
99.977% <= 0.407 milliseconds (cumulative count 999774)
99.990% <= 0.503 milliseconds (cumulative count 999900)
99.998% <= 0.607 milliseconds (cumulative count 999983)
99.999% <= 0.703 milliseconds (cumulative count 999995)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 1.007 milliseconds (cumulative count 999999)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.176     0.032     0.175     0.223     0.343     1.015
====== HSET ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.039 milliseconds (cumulative count 1)
50.000% <= 0.183 milliseconds (cumulative count 895730)
93.750% <= 0.231 milliseconds (cumulative count 946544)
96.875% <= 0.247 milliseconds (cumulative count 970363)
98.438% <= 0.351 milliseconds (cumulative count 988428)
99.219% <= 0.359 milliseconds (cumulative count 997193)
99.805% <= 0.367 milliseconds (cumulative count 999274)
99.951% <= 0.375 milliseconds (cumulative count 999537)
99.976% <= 0.391 milliseconds (cumulative count 999788)
99.988% <= 0.407 milliseconds (cumulative count 999894)
99.994% <= 0.447 milliseconds (cumulative count 999948)
99.997% <= 0.479 milliseconds (cumulative count 999972)
99.998% <= 0.599 milliseconds (cumulative count 999985)
99.999% <= 0.679 milliseconds (cumulative count 999994)
100.000% <= 0.759 milliseconds (cumulative count 999997)
100.000% <= 0.879 milliseconds (cumulative count 999999)
100.000% <= 1.063 milliseconds (cumulative count 1000000)
100.000% <= 1.063 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.065% <= 0.103 milliseconds (cumulative count 650)
92.796% <= 0.207 milliseconds (cumulative count 927956)
97.179% <= 0.303 milliseconds (cumulative count 971792)
99.989% <= 0.407 milliseconds (cumulative count 999894)
99.998% <= 0.503 milliseconds (cumulative count 999977)
99.999% <= 0.607 milliseconds (cumulative count 999986)
99.999% <= 0.703 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999998)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 1.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266524.50 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.183     0.032     0.183     0.239     0.359     1.063
====== SPOP ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 1)
50.000% <= 0.167 milliseconds (cumulative count 773087)
87.500% <= 0.175 milliseconds (cumulative count 920356)
93.750% <= 0.207 milliseconds (cumulative count 947424)
96.875% <= 0.223 milliseconds (cumulative count 970599)
98.438% <= 0.327 milliseconds (cumulative count 988245)
99.219% <= 0.335 milliseconds (cumulative count 997574)
99.805% <= 0.343 milliseconds (cumulative count 999446)
99.951% <= 0.351 milliseconds (cumulative count 999660)
99.976% <= 0.359 milliseconds (cumulative count 999764)
99.988% <= 0.383 milliseconds (cumulative count 999884)
99.994% <= 0.407 milliseconds (cumulative count 999941)
99.997% <= 0.431 milliseconds (cumulative count 999972)
99.998% <= 0.567 milliseconds (cumulative count 999985)
99.999% <= 0.679 milliseconds (cumulative count 999993)
100.000% <= 0.815 milliseconds (cumulative count 999997)
100.000% <= 0.831 milliseconds (cumulative count 999999)
100.000% <= 0.943 milliseconds (cumulative count 1000000)
100.000% <= 0.943 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.270% <= 0.103 milliseconds (cumulative count 2699)
94.742% <= 0.207 milliseconds (cumulative count 947424)
97.381% <= 0.303 milliseconds (cumulative count 973806)
99.994% <= 0.407 milliseconds (cumulative count 999941)
99.998% <= 0.503 milliseconds (cumulative count 999980)
99.999% <= 0.607 milliseconds (cumulative count 999987)
99.999% <= 0.703 milliseconds (cumulative count 999995)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.170     0.024     0.167     0.215     0.335     0.943
====== ZADD ======                                                     
  1000000 requests completed in 4.00 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.047 milliseconds (cumulative count 9)
50.000% <= 0.183 milliseconds (cumulative count 867275)
87.500% <= 0.191 milliseconds (cumulative count 917560)
93.750% <= 0.231 milliseconds (cumulative count 943197)
96.875% <= 0.247 milliseconds (cumulative count 970920)
98.438% <= 0.351 milliseconds (cumulative count 985026)
99.219% <= 0.359 milliseconds (cumulative count 995175)
99.609% <= 0.367 milliseconds (cumulative count 999028)
99.951% <= 0.375 milliseconds (cumulative count 999604)
99.976% <= 0.383 milliseconds (cumulative count 999778)
99.988% <= 0.399 milliseconds (cumulative count 999926)
99.994% <= 0.407 milliseconds (cumulative count 999960)
99.997% <= 0.415 milliseconds (cumulative count 999979)
99.998% <= 0.423 milliseconds (cumulative count 999985)
99.999% <= 0.439 milliseconds (cumulative count 999994)
100.000% <= 0.495 milliseconds (cumulative count 999997)
100.000% <= 0.551 milliseconds (cumulative count 999999)
100.000% <= 0.559 milliseconds (cumulative count 1000000)
100.000% <= 0.559 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.067% <= 0.103 milliseconds (cumulative count 671)
92.754% <= 0.207 milliseconds (cumulative count 927537)
97.254% <= 0.303 milliseconds (cumulative count 972537)
99.996% <= 0.407 milliseconds (cumulative count 999960)
100.000% <= 0.503 milliseconds (cumulative count 999997)
100.000% <= 0.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 249875.08 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.185     0.040     0.183     0.239     0.359     0.559
====== ZPOPMIN ======                                                     
  1000000 requests completed in 3.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.031 milliseconds (cumulative count 1)
50.000% <= 0.167 milliseconds (cumulative count 754764)
87.500% <= 0.175 milliseconds (cumulative count 918682)
93.750% <= 0.207 milliseconds (cumulative count 945403)
96.875% <= 0.223 milliseconds (cumulative count 970367)
98.438% <= 0.327 milliseconds (cumulative count 987513)
99.219% <= 0.335 milliseconds (cumulative count 997265)
99.805% <= 0.343 milliseconds (cumulative count 999416)
99.951% <= 0.351 milliseconds (cumulative count 999601)
99.976% <= 0.367 milliseconds (cumulative count 999766)
99.988% <= 0.415 milliseconds (cumulative count 999878)
99.994% <= 0.463 milliseconds (cumulative count 999945)
99.997% <= 0.623 milliseconds (cumulative count 999970)
99.998% <= 6.031 milliseconds (cumulative count 999986)
99.999% <= 6.063 milliseconds (cumulative count 999994)
100.000% <= 6.079 milliseconds (cumulative count 999997)
100.000% <= 6.087 milliseconds (cumulative count 999999)
100.000% <= 6.239 milliseconds (cumulative count 1000000)
100.000% <= 6.239 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.298% <= 0.103 milliseconds (cumulative count 2980)
94.540% <= 0.207 milliseconds (cumulative count 945403)
97.364% <= 0.303 milliseconds (cumulative count 973642)
99.987% <= 0.407 milliseconds (cumulative count 999870)
99.995% <= 0.503 milliseconds (cumulative count 999949)
99.996% <= 0.607 milliseconds (cumulative count 999965)
99.998% <= 0.703 milliseconds (cumulative count 999978)
99.998% <= 0.807 milliseconds (cumulative count 999980)
99.998% <= 0.903 milliseconds (cumulative count 999983)
100.000% <= 6.103 milliseconds (cumulative count 999999)
100.000% <= 7.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 285551.09 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.170     0.024     0.167     0.215     0.335     6.239
====== LPUSH (needed to benchmark LRANGE) ======                                                     
  1000000 requests completed in 3.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.039 milliseconds (cumulative count 1)
50.000% <= 0.175 milliseconds (cumulative count 542851)
75.000% <= 0.183 milliseconds (cumulative count 903726)
93.750% <= 0.223 milliseconds (cumulative count 939003)
96.875% <= 0.247 milliseconds (cumulative count 970837)
98.438% <= 0.351 milliseconds (cumulative count 991668)
99.219% <= 0.359 milliseconds (cumulative count 998190)
99.902% <= 0.367 milliseconds (cumulative count 999327)
99.951% <= 0.375 milliseconds (cumulative count 999580)
99.976% <= 0.391 milliseconds (cumulative count 999775)
99.988% <= 0.415 milliseconds (cumulative count 999878)
99.994% <= 0.495 milliseconds (cumulative count 999939)
99.997% <= 0.567 milliseconds (cumulative count 999971)
99.998% <= 0.687 milliseconds (cumulative count 999987)
99.999% <= 0.751 milliseconds (cumulative count 999994)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 1.071 milliseconds (cumulative count 999999)
100.000% <= 1.159 milliseconds (cumulative count 1000000)
100.000% <= 1.159 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.080% <= 0.103 milliseconds (cumulative count 797)
92.979% <= 0.207 milliseconds (cumulative count 929794)
97.205% <= 0.303 milliseconds (cumulative count 972050)
99.986% <= 0.407 milliseconds (cumulative count 999860)
99.995% <= 0.503 milliseconds (cumulative count 999945)
99.998% <= 0.607 milliseconds (cumulative count 999980)
99.999% <= 0.703 milliseconds (cumulative count 999989)
100.000% <= 0.807 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 999998)
100.000% <= 1.103 milliseconds (cumulative count 999999)
100.000% <= 1.207 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 266595.59 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.182     0.032     0.175     0.231     0.351     1.159
====== LRANGE_100 (first 100 elements) ======                                                     
  1000000 requests completed in 8.01 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.071 milliseconds (cumulative count 1)
50.000% <= 0.391 milliseconds (cumulative count 508461)
75.000% <= 0.455 milliseconds (cumulative count 775979)
87.500% <= 0.479 milliseconds (cumulative count 902295)
93.750% <= 0.495 milliseconds (cumulative count 956867)
96.875% <= 0.503 milliseconds (cumulative count 971568)
98.438% <= 0.527 milliseconds (cumulative count 986658)
99.219% <= 0.551 milliseconds (cumulative count 994355)
99.609% <= 0.559 milliseconds (cumulative count 996430)
99.805% <= 0.575 milliseconds (cumulative count 998852)
99.902% <= 0.583 milliseconds (cumulative count 999385)
99.951% <= 0.591 milliseconds (cumulative count 999649)
99.976% <= 0.599 milliseconds (cumulative count 999787)
99.988% <= 0.615 milliseconds (cumulative count 999878)
99.994% <= 0.719 milliseconds (cumulative count 999941)
99.997% <= 0.831 milliseconds (cumulative count 999970)
99.998% <= 0.911 milliseconds (cumulative count 999988)
99.999% <= 0.959 milliseconds (cumulative count 999993)
100.000% <= 1.015 milliseconds (cumulative count 999997)
100.000% <= 1.095 milliseconds (cumulative count 999999)
100.000% <= 1.151 milliseconds (cumulative count 1000000)
100.000% <= 1.151 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 4)
0.023% <= 0.207 milliseconds (cumulative count 230)
38.045% <= 0.303 milliseconds (cumulative count 380454)
54.858% <= 0.407 milliseconds (cumulative count 548582)
97.157% <= 0.503 milliseconds (cumulative count 971568)
99.986% <= 0.607 milliseconds (cumulative count 999855)
99.993% <= 0.703 milliseconds (cumulative count 999934)
99.996% <= 0.807 milliseconds (cumulative count 999964)
99.998% <= 0.903 milliseconds (cumulative count 999984)
100.000% <= 1.007 milliseconds (cumulative count 999996)
100.000% <= 1.103 milliseconds (cumulative count 999999)
100.000% <= 1.207 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 124921.92 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.373     0.064     0.391     0.495     0.543     1.151
====== LRANGE_300 (first 300 elements) ======                                                   
  1000000 requests completed in 16.76 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.231 milliseconds (cumulative count 1)
50.000% <= 0.727 milliseconds (cumulative count 514185)
75.000% <= 0.783 milliseconds (cumulative count 784006)
87.500% <= 0.815 milliseconds (cumulative count 880548)
93.750% <= 0.855 milliseconds (cumulative count 951931)
96.875% <= 0.871 milliseconds (cumulative count 969220)
98.438% <= 0.895 milliseconds (cumulative count 988458)
99.219% <= 0.911 milliseconds (cumulative count 994648)
99.609% <= 0.919 milliseconds (cumulative count 997641)
99.805% <= 0.927 milliseconds (cumulative count 998617)
99.902% <= 0.935 milliseconds (cumulative count 999238)
99.951% <= 0.951 milliseconds (cumulative count 999568)
99.976% <= 1.007 milliseconds (cumulative count 999758)
99.988% <= 1.119 milliseconds (cumulative count 999882)
99.994% <= 1.247 milliseconds (cumulative count 999939)
99.997% <= 1.327 milliseconds (cumulative count 999971)
99.998% <= 1.375 milliseconds (cumulative count 999985)
99.999% <= 1.439 milliseconds (cumulative count 999994)
100.000% <= 1.471 milliseconds (cumulative count 999997)
100.000% <= 1.495 milliseconds (cumulative count 999999)
100.000% <= 1.543 milliseconds (cumulative count 1000000)
100.000% <= 1.543 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
0.001% <= 0.303 milliseconds (cumulative count 6)
0.006% <= 0.407 milliseconds (cumulative count 61)
0.054% <= 0.503 milliseconds (cumulative count 539)
5.776% <= 0.607 milliseconds (cumulative count 57764)
39.845% <= 0.703 milliseconds (cumulative count 398452)
86.030% <= 0.807 milliseconds (cumulative count 860299)
99.017% <= 0.903 milliseconds (cumulative count 990171)
99.976% <= 1.007 milliseconds (cumulative count 999758)
99.987% <= 1.103 milliseconds (cumulative count 999870)
99.992% <= 1.207 milliseconds (cumulative count 999923)
99.996% <= 1.303 milliseconds (cumulative count 999964)
99.999% <= 1.407 milliseconds (cumulative count 999990)
100.000% <= 1.503 milliseconds (cumulative count 999999)
100.000% <= 1.607 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 59676.55 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.726     0.224     0.727     0.855     0.903     1.543
====== LRANGE_500 (first 500 elements) ======                                                   
  1000000 requests completed in 25.27 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.279 milliseconds (cumulative count 1)
50.000% <= 1.095 milliseconds (cumulative count 501278)
75.000% <= 1.175 milliseconds (cumulative count 757766)
87.500% <= 1.215 milliseconds (cumulative count 877817)
93.750% <= 1.231 milliseconds (cumulative count 945683)
96.875% <= 1.255 milliseconds (cumulative count 970532)
98.438% <= 1.287 milliseconds (cumulative count 986073)
99.219% <= 1.311 milliseconds (cumulative count 992920)
99.609% <= 1.335 milliseconds (cumulative count 996831)
99.805% <= 1.359 milliseconds (cumulative count 998296)
99.902% <= 1.407 milliseconds (cumulative count 999056)
99.951% <= 1.487 milliseconds (cumulative count 999544)
99.976% <= 1.559 milliseconds (cumulative count 999760)
99.988% <= 1.623 milliseconds (cumulative count 999894)
99.994% <= 1.663 milliseconds (cumulative count 999945)
99.997% <= 1.703 milliseconds (cumulative count 999975)
99.998% <= 1.743 milliseconds (cumulative count 999985)
99.999% <= 1.823 milliseconds (cumulative count 999993)
100.000% <= 1.903 milliseconds (cumulative count 999997)
100.000% <= 1.967 milliseconds (cumulative count 999999)
100.000% <= 1.999 milliseconds (cumulative count 1000000)
100.000% <= 1.999 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
0.000% <= 0.303 milliseconds (cumulative count 3)
0.002% <= 0.407 milliseconds (cumulative count 23)
0.011% <= 0.503 milliseconds (cumulative count 107)
0.027% <= 0.607 milliseconds (cumulative count 268)
0.075% <= 0.703 milliseconds (cumulative count 755)
0.939% <= 0.807 milliseconds (cumulative count 9386)
4.483% <= 0.903 milliseconds (cumulative count 44829)
22.258% <= 1.007 milliseconds (cumulative count 222579)
51.879% <= 1.103 milliseconds (cumulative count 518793)
86.574% <= 1.207 milliseconds (cumulative count 865742)
99.106% <= 1.303 milliseconds (cumulative count 991057)
99.906% <= 1.407 milliseconds (cumulative count 999056)
99.961% <= 1.503 milliseconds (cumulative count 999612)
99.986% <= 1.607 milliseconds (cumulative count 999858)
99.997% <= 1.703 milliseconds (cumulative count 999975)
99.999% <= 1.807 milliseconds (cumulative count 999992)
100.000% <= 1.903 milliseconds (cumulative count 999997)
100.000% <= 2.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 39575.75 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        1.090     0.272     1.095     1.239     1.303     1.999
====== LRANGE_600 (first 600 elements) ======                                                   
  1000000 requests completed in 29.28 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.119 milliseconds (cumulative count 1)
50.000% <= 1.271 milliseconds (cumulative count 502697)
75.000% <= 1.367 milliseconds (cumulative count 756433)
87.500% <= 1.415 milliseconds (cumulative count 886278)
93.750% <= 1.423 milliseconds (cumulative count 939613)
96.875% <= 1.447 milliseconds (cumulative count 974533)
98.438% <= 1.463 milliseconds (cumulative count 984479)
99.219% <= 1.527 milliseconds (cumulative count 992248)
99.609% <= 1.703 milliseconds (cumulative count 996104)
99.805% <= 1.823 milliseconds (cumulative count 998129)
99.902% <= 1.887 milliseconds (cumulative count 999057)
99.951% <= 1.951 milliseconds (cumulative count 999539)
99.976% <= 1.975 milliseconds (cumulative count 999760)
99.988% <= 2.015 milliseconds (cumulative count 999885)
99.994% <= 2.047 milliseconds (cumulative count 999944)
99.997% <= 2.071 milliseconds (cumulative count 999974)
99.998% <= 2.095 milliseconds (cumulative count 999985)
99.999% <= 2.111 milliseconds (cumulative count 999993)
100.000% <= 2.175 milliseconds (cumulative count 999997)
100.000% <= 2.255 milliseconds (cumulative count 999999)
100.000% <= 2.303 milliseconds (cumulative count 1000000)
100.000% <= 2.303 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.000% <= 0.103 milliseconds (cumulative count 0)
0.000% <= 0.207 milliseconds (cumulative count 2)
0.000% <= 0.303 milliseconds (cumulative count 3)
0.001% <= 0.407 milliseconds (cumulative count 8)
0.013% <= 0.503 milliseconds (cumulative count 131)
0.029% <= 0.607 milliseconds (cumulative count 287)
0.190% <= 0.703 milliseconds (cumulative count 1896)
0.541% <= 0.807 milliseconds (cumulative count 5406)
1.477% <= 0.903 milliseconds (cumulative count 14767)
3.623% <= 1.007 milliseconds (cumulative count 36233)
9.992% <= 1.103 milliseconds (cumulative count 99921)
30.939% <= 1.207 milliseconds (cumulative count 309389)
58.740% <= 1.303 milliseconds (cumulative count 587404)
87.091% <= 1.407 milliseconds (cumulative count 870909)
99.102% <= 1.503 milliseconds (cumulative count 991023)
99.430% <= 1.607 milliseconds (cumulative count 994295)
99.610% <= 1.703 milliseconds (cumulative count 996104)
99.792% <= 1.807 milliseconds (cumulative count 997918)
99.917% <= 1.903 milliseconds (cumulative count 999171)
99.987% <= 2.007 milliseconds (cumulative count 999867)
99.999% <= 2.103 milliseconds (cumulative count 999992)
100.000% <= 3.103 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 34148.34 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        1.263     0.112     1.271     1.431     1.495     2.303
====== MSET (10 keys) ======                                                     
  1000000 requests completed in 5.25 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.063 milliseconds (cumulative count 4)
50.000% <= 0.255 milliseconds (cumulative count 910479)
93.750% <= 0.295 milliseconds (cumulative count 938257)
96.875% <= 0.383 milliseconds (cumulative count 971622)
98.438% <= 0.495 milliseconds (cumulative count 990826)
99.219% <= 0.503 milliseconds (cumulative count 998023)
99.805% <= 0.511 milliseconds (cumulative count 999628)
99.976% <= 0.519 milliseconds (cumulative count 999807)
99.988% <= 0.527 milliseconds (cumulative count 999895)
99.994% <= 0.543 milliseconds (cumulative count 999955)
99.997% <= 0.591 milliseconds (cumulative count 999973)
99.998% <= 0.631 milliseconds (cumulative count 999985)
99.999% <= 0.711 milliseconds (cumulative count 999993)
100.000% <= 0.823 milliseconds (cumulative count 999997)
100.000% <= 0.879 milliseconds (cumulative count 999999)
100.000% <= 0.911 milliseconds (cumulative count 1000000)
100.000% <= 0.911 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.003% <= 0.103 milliseconds (cumulative count 27)
0.464% <= 0.207 milliseconds (cumulative count 4638)
93.872% <= 0.303 milliseconds (cumulative count 938720)
97.376% <= 0.407 milliseconds (cumulative count 973764)
99.802% <= 0.503 milliseconds (cumulative count 998023)
99.998% <= 0.607 milliseconds (cumulative count 999980)
99.999% <= 0.703 milliseconds (cumulative count 999992)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 190367.42 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.258     0.056     0.255     0.375     0.495     0.911
====== XADD ======                                                     
  1000000 requests completed in 4.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 
  host configuration "appendonly": no
  multi-thread: yes
  threads: 3

Latency by percentile distribution:
0.000% <= 0.063 milliseconds (cumulative count 1)
50.000% <= 0.207 milliseconds (cumulative count 587221)
75.000% <= 0.215 milliseconds (cumulative count 910834)
93.750% <= 0.255 milliseconds (cumulative count 938134)
96.875% <= 0.303 milliseconds (cumulative count 972089)
98.438% <= 0.415 milliseconds (cumulative count 993316)
99.609% <= 0.423 milliseconds (cumulative count 998871)
99.902% <= 0.431 milliseconds (cumulative count 999601)
99.976% <= 0.447 milliseconds (cumulative count 999845)
99.988% <= 0.455 milliseconds (cumulative count 999888)
99.994% <= 0.495 milliseconds (cumulative count 999939)
99.997% <= 0.599 milliseconds (cumulative count 999972)
99.998% <= 0.703 milliseconds (cumulative count 999985)
99.999% <= 0.759 milliseconds (cumulative count 999993)
100.000% <= 0.887 milliseconds (cumulative count 999997)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 0.983 milliseconds (cumulative count 1000000)
100.000% <= 0.983 milliseconds (cumulative count 1000000)

Cumulative distribution of latencies:
0.015% <= 0.103 milliseconds (cumulative count 152)
58.722% <= 0.207 milliseconds (cumulative count 587221)
97.209% <= 0.303 milliseconds (cumulative count 972089)
98.264% <= 0.407 milliseconds (cumulative count 982644)
99.994% <= 0.503 milliseconds (cumulative count 999940)
99.997% <= 0.607 milliseconds (cumulative count 999975)
99.999% <= 0.703 milliseconds (cumulative count 999985)
100.000% <= 0.807 milliseconds (cumulative count 999996)
100.000% <= 0.903 milliseconds (cumulative count 999999)
100.000% <= 1.007 milliseconds (cumulative count 1000000)

Summary:
  throughput summary: 222172.86 requests per second
  latency summary (msec):
          avg       min       p50       p95       p99       max
        0.215     0.056     0.207     0.295     0.415     0.983
  ```
</details>

As you see, throughput and latency are improved. Somewhere not so many, somewhere - a lot (IMO). So using PGO with Redis is a good thing to try.
