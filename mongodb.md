## MongoDB PGO

Current results:
* `mongod` from the `master` branch and `r6.3.1` segfaults on the start if built in instrumentation mode (with `-fprofile-generate` compiler flag)
* `mongod` does not compile with Clang 12 and corresponding `libc++` in Fedora 38 due to multiple compilation errors on the current `master` branch
* `mongod` does not compile with Clang 16 and corresponding `libc++` in Fedora 38 due to multiple compilation errors on the current `master` branch
* `mongod` does not compile with Clang 12 and 16 and `libstdc++` from Fedora 38 due to multiple compilation errors on the current `master` branch
* `mongod` does not compile with Clang 16 and `libstdc++` from Fedora 38 due to multiple compilation errors on the current `master` branch
* MongoDB on the `master` branch does not correctly detect GCC 13 supports `-flto` flag with enabled `--lto` build option

Seems like `r6.3.1` MongoDB tag builds fine with Clang 16. The tests will continue. So the tests performed on this MongoDB version.

## Test environment

* Fedora 38
* Linux kernel 6.3.7 (default Linux kernel from Fedora repos)
* AMD Ryzen 9 5800x
* 48 Gib RAM
* SSD Samsung 980 Pro 2 Tib
* Clang 16 (from the Fedora repositories). I use Clang just because I prefer LLVM-based tooling
* MongoDB: `r6.3.1` git tag

## Tested configurations

I have tested the following MongoDB configurations (with corresponding `CFLAGS` and `CXXFLAGS`):

* Release + LTO: `python3 buildscripts/scons.py install-mongod --disable-warnings-as-errors --release --opt=on --variables-files=env_vars --linker=lld --lto`
* Release with LTO and PGO: `python3 buildscripts/scons.py install-mongod --disable-warnings-as-errors --release --opt=on --variables-files=env_vars --linker=lld --lto` + `-fprofile-generate/-fprofile-use` flags

## Benchmark

I use [ClickBench](https://github.com/ClickHouse/ClickBench/tree/main/mongodb) as a MongoDB benchmark. I am quite familiar with this bench quite already and it's proven by multiple people.

I understand ClickBench does not represent a "fully tuned" MongoDB instance but I have no aim to do this here. I just want to show you applicability of LTO and PGO optimization even for suboptimal MongoDB configurations.

So MongoDB with launched with "default" flags like `mongod --dbpath some_path`.

## Results

Here are results of running ClickBench test suite on different configurations. All configurations are benchmarked on the same machine with the same "background" load, with the same MongoDB configuration, multiple times, etc. Results are shown in seconds (per query). Each query is repeated 3 times to get a more precise time measure. Usually the first query is quite slower - I guess it's because some disk caching or something like that.

<details>
  <summary>Release + LTO</summary>

  ```
Q0: [35.826,31.335,31.351],
Q1: [0.435,0.292,0.29],
Q2: [51.567,45.054,45.163],
Q3: [67.35,52.151,52.34],
Q4: [19.269,17.533,17.854],
Q5: [12.768,6.964,6.804],
Q6: [0.033,0.002,0.001],
Q7: [0.329,0.32,0.326],
Q8: [115.489,101.501,100.989],
Q9: [749.65,740.987,752.106],
Q10: [83.841,85.164,85.723],
Q11: [85.356,84.906,84.612],
Q12: [37.401,33.398,35.375],
Q13: [88.28,80.455,80.258],
Q14: [42.821,41.439,40.919],
Q15: [119.933,104.6,102.567],
Q16: [196.342,179.439,179.191],
Q17: [168.416,167.818,167.587],
Q18: [349.98,355.375,353.483],
Q19: [0.012,0.001,0.002],
Q20: [90.853,68.424,67.836],
Q21: [38.432,10.618,10.963],
Q22: [13.825,12.667,12.975],
Q23: [75.023,73.102,72.047],
Q24: [11.454,8.889,8.916],
Q25: [0.013,0.003,0.002],
Q26: [12.565,11.824,11.839],
Q27: [84.827,84.547,84.554],
Q28: [147.623,126.348,126.21],
Q29: [751.974,727.093,718.933],
Q30: [941.83,927.874,919.392],
Q31: [960.018,954.19,949.981],
Q32: [531.894,504.072,498.701],
Q33: [215.041,154.51,150.639],
Q34: [154.783,156.363,155.096],
Q35: [86.615,73.763,72.729],
Q36: [3.441,2.319,2.274],
Q37: [0.959,0.993,0.985],
Q38: [1.848,1.637,1.61],
Q39: [6.574,6.528,6.52],
Q40: [0.345,0.237,0.222],
Q41: [0.273,0.197,0.2],
Q42: [0.919,0.903,0.897]
  ```
</details>
<details>
  <summary>Release + LTO + PGO</summary>

  ```
Q0: [23.394,20.967,20.957],
Q1: [0.375,0.17,0.169],
Q2: [36.872,31.18,31.232],
Q3: [49.96,36.729,36.409],
Q4: [14.707,12.583,12.614],
Q5: [10.628,5.36,5.362],
Q6: [0.032,0.003,0.002],
Q7: [0.193,0.191,0.194],
Q8: [90.35,78.976,79.443],
Q9: [682.167,683.089,680.439],
Q10: [80.194,81.111,80.765],
Q11: [81.13,79.63,82.055],
Q12: [31.475,27.82,28.278],
Q13: [76.463,66.725,68.11],
Q14: [38.545,37.55,37.775],
Q15: [93.688,79.706,81.342],
Q16: [173.557,155.02,153.02],
Q17: [143.592,146.857,150.647],
Q18: [370.004,362.816,367.933],
Q19: [0.012,0.002,0.002],
Q20: [65.832,42.293,42.392],
Q21: [34.714,6.932,6.835],
Q22: [9.337,8.335,8.391],
Q23: [47.847,44.624,44.203],
Q24: [5.62,5.438,5.546],
Q25: [0.003,0.003,0.002],
Q26: [7.932,7.926,7.84],
Q27: [54.349,54.582,54.442],
Q28: [114.345,93.111,93.605],
Q29: [443.32,435.165,430.915],
Q30: [894.697,877.858,880.349],
Q31: [905.968,897.467,895.725],
Q32: [470.077,446.707,439.211],
Q33: [186.18,128.77,129.988],
Q34: [124.363,125.038,127.563],
Q35: [71.119,55.715,54.314],
Q36: [2.852,2.071,2.058],
Q37: [0.832,0.828,0.824],
Q38: [1.58,1.371,1.337],
Q39: [6.333,6.335,6.175],
Q40: [0.308,0.202,0.196],
Q41: [0.242,0.172,0.171],
Q42: [0.768,0.752,0.751]
  ```
</details>

As you see, results are good. So there is a reason to try apply PGO to MongoDB in your case.
