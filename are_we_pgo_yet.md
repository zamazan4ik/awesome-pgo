# Are we PGO yet?

Just a list of PGO-related issues in different projects. So you can estimate the PGO state in your favourite open-source product.

## Browsers

* Brave: https://github.com/brave/brave-browser/issues/20560

## Compilers and interpreters

* Ruby: https://bugs.ruby-lang.org/issues/19256
* RustPython: https://github.com/RustPython/RustPython/issues/5000
* Red: https://github.com/red/red/issues/5333
* mrustc: https://github.com/thepowersgang/mrustc/issues/304
* Flang: https://github.com/llvm/llvm-project/issues/63376
* kphp: https://github.com/VKCOM/kphp/issues/862
* [Elena-lang](https://elena-lang.github.io/): https://github.com/ELENA-LANG/elena-lang/issues/578
* LPython: https://github.com/lcompilers/lpython/issues/2217
* LFortran: https://github.com/lcompilers/lpython/issues/2217
* Codon: https://github.com/exaloop/codon/issues/137
* Open Watcom: https://github.com/open-watcom/open-watcom-v2/issues/1113
* Micropython: https://github.com/micropython/micropython/issues/11965
* C2Rust: https://github.com/immunant/c2rust/issues/989
* Artichoke: https://github.com/artichoke/artichoke/issues/2627
* Fullstaq-ruby: https://github.com/fullstaq-ruby/server-edition/issues/5
* Lua: https://www.reddit.com/r/lua/comments/151dtyu/profileguided_optimization_pgo_on_lua_interpreters/
* Redscript: https://github.com/jac3km4/redscript/issues/94
* Janet: https://github.com/janet-lang/janet/issues/1221
* Espruino: https://github.com/espruino/Espruino/issues/2392
* Hashlink: https://github.com/HaxeFoundation/hashlink/issues/606
* Boa: https://github.com/boa-dev/boa/issues/3139
* loxcraft: https://github.com/ajeetdsouza/loxcraft/issues/18
* Deno: https://github.com/denoland/rusty_v8/pull/1063
* Piccolo: https://github.com/triplehex/piccolo/issues/30
* HHVM: https://github.com/facebook/hhvm/issues/9385
* Burst (Unity Engine): https://forum.unity.com/threads/profile-guided-optimisation-for-burst.931419/
* KCL: https://github.com/kcl-lang/kcl/issues/647
* Zig: https://github.com/ziglang/zig/issues/16759
* Kuroko: https://github.com/kuroko-lang/kuroko/issues/42
* Noir: https://github.com/noir-lang/noir/issues/2534
* Move: https://github.com/move-language/move/issues/1072
* Ezno: https://github.com/kaleidawave/ezno/issues/58
* Natalie: https://github.com/natalie-lang/natalie/issues/1251
* Qsharp: https://github.com/microsoft/qsharp/issues/752
* Uiua: https://github.com/uiua-lang/uiua/issues/48
* Xeus-clang-repl: https://github.com/compiler-research/xeus-clang-repl/issues/71
* Ferrocene: https://github.com/ferrocene/ferrocene/issues/22
* Rimu: https://github.com/ahdinosaur/rimu/issues/75
* Berry: https://github.com/berry-lang/berry/issues/367
* frawk: https://github.com/ezrosent/frawk/issues/103
* Chapel: https://github.com/chapel-lang/chapel/issues/23608
* Nature: https://github.com/nature-lang/nature/issues/34
* DPC++: https://github.com/intel/llvm/issues/11555
* Swift:
  - https://github.com/apple/swift/issues/69227
  - https://forums.swift.org/t/several-questions-regarding-profile-guided-optimization-pgo-and-llvm-bolt/67963
* AdaptiveCpp: https://github.com/AdaptiveCpp/AdaptiveCpp/issues/1198
* ISPC: https://github.com/ispc/ispc/issues/2687
* Hylo: https://github.com/hylo-lang/hylo/issues/1113
* SPIRV-LLVM: https://github.com/KhronosGroup/SPIRV-LLVM-Translator/issues/2191
* GraalVM: https://github.com/oracle/graal/discussions/7648
* Intel Graphics Compiler (IGC): https://github.com/intel/intel-graphics-compiler/issues/305
* Dart:
  - https://github.com/dart-lang/sdk/issues/53855
  - https://github.com/dart-lang/sdk/issues/53856
* Gravity: https://github.com/marcobambini/gravity/issues/414
* daScript: https://github.com/GaijinEntertainment/daScript/issues/831
* Quirrel: https://github.com/GaijinEntertainment/quirrel/issues/84
* Leo: https://github.com/AleoHQ/leo/issues/3226
* Julia: https://github.com/JuliaLang/julia/pull/45641
* D compilers (DMD, GDC, LDC):
  - All D compilers: https://forum.dlang.org/post/ukljubhhlosfygbdzfcm@forum.dlang.org
  - LDC: https://github.com/ldc-developers/ldc/discussions/4524 (general discussion) , https://github.com/ldc-developers/ldc/issues/4526 (CSIR PGO), https://github.com/ldc-developers/ldc/issues/4527 (AutoFDO)
* Erlang: https://erlangforums.com/t/profile-guided-optimization-pgo-benchmarks-on-erlang-otp/3028
* Inko:
  - https://github.com/inko-lang/inko/issues/650
  - https://github.com/inko-lang/inko/issues/651
* GnuCobol: https://sourceforge.net/p/gnucobol/feature-requests/456/
* gcc-cobol: Cannot create an issue in https://gitlab.cobolworx.com/COBOLworx/gcc-cobol
* numbat: https://github.com/sharkdp/numbat/issues/257
* R: https://stackoverflow.com/collectives/r-language/beta/discussions/77526179/adding-profile-guided-optimization-pgo-to-the-r-interpreter
* Onyx: https://github.com/onyx-lang/onyx/issues/41

## Developer tooling

* Clippy: https://github.com/rust-lang/rust-clippy/issues/10121
* mold: https://github.com/rui314/mold/issues/250
* LLVM projects (like Clangd, Clang-Tidy, lld): https://github.com/llvm/llvm-project/issues/63486
* cppcheck: https://trac.cppcheck.net/ticket/11672
* Rust Analyzer: https://github.com/rust-lang/rust-analyzer/issues/9412
* AutoFDO: https://github.com/google/autofdo/issues/170
* Sccache: https://github.com/mozilla/sccache/issues/1811
* BOLT: https://github.com/llvm/llvm-project/issues/63245
* ccls: https://github.com/MaskRay/ccls/issues/942
* Gleam: https://github.com/gleam-lang/gleam/issues/2237
* Gluon: https://github.com/gluon-lang/gluon/issues/954
* Millet: https://github.com/azdavis/millet/issues/37
* Sorbet: https://github.com/sorbet/sorbet/issues/7102
* Protobuf (`protoc`): https://github.com/protocolbuffers/protobuf/issues/13248
* Cap'n Proto: https://github.com/capnproto/capnproto/issues/1734
* Cling: https://github.com/root-project/cling/issues/508
* RetDec: https://github.com/avast/retdec/issues/1165
* Uncrustify: https://github.com/uncrustify/uncrustify/issues/4045
* SPIRV-Tools: https://github.com/KhronosGroup/SPIRV-Tools/issues/5303
* Clazy: https://bugs.kde.org/show_bug.cgi?id=472265
* Compressonator: https://github.com/GPUOpen-Tools/compressonator/issues/260
* IWYU: https://github.com/include-what-you-use/include-what-you-use/issues/1273
* Radare2: https://github.com/radareorg/radare2/issues/22032
* AFLplusplus: https://github.com/AFLplusplus/AFLplusplus/issues/1803
* Drill: https://github.com/fcsonline/drill/issues/185
* Oha: https://github.com/hatoo/oha/issues/264
* ROCm: https://github.com/RadeonOpenCompute/ROCm/issues/2325
* KDevelop: https://bugs.kde.org/show_bug.cgi?id=472554
* Klee: https://github.com/klee/klee/issues/1653
* SVF: https://github.com/SVF-tools/SVF/issues/1152
* IKOS: https://github.com/NASA-SW-VnV/ikos/issues/211
* Emscripten: https://github.com/emscripten-core/emscripten/issues/19671
* Redex: https://github.com/facebook/redex/issues/820
* Mariana-Trench: https://github.com/facebook/mariana-trench/issues/137
* FBThrift: https://github.com/facebook/fbthrift/issues/564
* Hermes: https://github.com/facebook/hermes/issues/1076
* Yoga: https://github.com/facebook/yoga/issues/1338
* Turbopack: https://github.com/vercel/turbo/issues/5667
* oxc: https://github.com/web-infra-dev/oxc/issues/812
* biome: https://github.com/biomejs/biome/discussions/85
* Ruff: https://github.com/astral-sh/ruff/issues/7055
* Redpen: https://github.com/estebank/redpen/issues/3
* Kani: https://github.com/model-checking/kani/issues/2751
* typos: https://github.com/crate-ci/typos/issues/827
* delta: https://github.com/dandavison/delta/issues/1540
* Enso: https://github.com/orgs/enso-org/discussions/7795
* Siege: https://github.com/JoeDog/siege/issues/227
* Prusti: https://github.com/viperproject/prusti-dev/issues/1454
* Mirai: https://github.com/facebookexperimental/MIRAI/issues/1237
* ESBMC: https://github.com/esbmc/esbmc/issues/1343
* Phasar: https://github.com/secure-software-engineering/phasar/issues/668
* Vera++: https://bitbucket.org/verateam/vera/issues/140/evaluate-profile-guided-optimization-pgo
* Creusot: https://github.com/xldenis/creusot/issues/884
* Rust-Code-Analysis: https://github.com/mozilla/rust-code-analysis/issues/1077
* grcov: https://github.com/mozilla/grcov/issues/1128
* ctags: https://github.com/universal-ctags/ctags/issues/3849
* ast-grep: https://github.com/ast-grep/ast-grep/discussions/738
* Symbolicator: https://github.com/getsentry/symbolicator/issues/1334

## Build systems

* CMake:
  - https://gitlab.kitware.com/cmake/cmake/-/issues/19273
  - https://gitlab.kitware.com/cmake/cmake/-/issues/25356
* Meson: https://github.com/mesonbuild/meson/issues/5251
* SCons: https://github.com/SCons/scons/discussions/4437
* Buck2: https://github.com/facebook/buck2/issues/454
* FPM: https://github.com/fortran-lang/fpm/discussions/968
* Cargo: https://github.com/rust-lang/cargo/issues/7618

## Gamedev

* Unreal Engine: https://forums.unrealengine.com/t/profile-guided-optimization-pgo-results-with-unreal-engine/1253240
* Bevy: https://github.com/bevyengine/bevy/issues/4586#issuecomment-1674097867
* Fyrox: https://github.com/FyroxEngine/Fyrox/issues/498
* Bitty: https://github.com/paladin-t/bitty/issues/38
* Hazel: https://github.com/TheCherno/Hazel/issues/645
* Open 3D Engine: https://github.com/o3de/o3de/discussions/16883
* DagorEngine: https://github.com/GaijinEntertainment/DagorEngine/issues/1
* Cocos Engine: https://github.com/cocos/cocos-engine/issues/16449
* Flax Engine: https://github.com/FlaxEngine/FlaxEngine/issues/1833
* Defold: https://github.com/defold/defold/issues/8193
* wgpu: https://github.com/gfx-rs/wgpu/discussions/4692
* glslang: https://github.com/KhronosGroup/glslang/issues/3400

## Operating systems

* FreeBSD: https://www.reddit.com/r/freebsd/comments/150codp/profileguided_optimization_pgo_on_freebsd_kernel/
* Zephyr: https://github.com/zephyrproject-rtos/zephyr/issues/60953
* SerenityOS: https://github.com/SerenityOS/serenity/issues/19549
* Embox: https://github.com/embox/embox/issues/2848
* Vesper: https://github.com/metta-systems/vesper/issues/187
* ToaruOS: https://github.com/klange/toaruos/issues/284
* SnarkOS: https://github.com/AleoHQ/snarkOS/issues/2636
* Aero: https://github.com/Andy-Python-Programmer/aero/issues/108
* Bottlerocket: https://github.com/bottlerocket-os/bottlerocket/discussions/3454
* lk: https://github.com/littlekernel/lk/issues/387
* Fuchsia: https://bugs.fuchsia.dev/p/fuchsia/issues/detail?id=137125

## Virtual machines and emulators

* Firecracker: https://github.com/firecracker-microvm/firecracker/issues/3456
* Ruffle: https://github.com/ruffle-rs/ruffle/issues/12094
* Cemu: https://github.com/cemu-project/Cemu/issues/797
* CrosVM: https://issuetracker.google.com/issues/297890486
* Cloud Hypervisor: https://github.com/cloud-hypervisor/cloud-hypervisor/issues/5768
* Hocus: https://github.com/hocus-dev/hocus/issues/141

## Databases

* ScyllaDB: https://github.com/scylladb/scylladb/pull/10808
* Tarantool: https://github.com/tarantool/tarantool/issues/8089
* ArangoDB: https://github.com/arangodb/arangodb/issues/17861
* DragonflyDB: https://github.com/dragonflydb/dragonfly/issues/592
* YDB: https://github.com/ydb-platform/ydb/issues/140
* Quickwit: https://github.com/quickwit-oss/quickwit/issues/3548
* YugabyteDB:
  - https://github.com/yugabyte/yugabyte-db/issues/12763
  - https://github.com/yugabyte/yugabyte-db/issues/18497
  - https://github.com/yugabyte/yugabyte-db/issues/20089
  - https://github.com/yugabyte/yugabyte-db/issues/20090
* comdb2: https://github.com/bloomberg/comdb2/issues/3597
* ClickHouse:
  - https://github.com/ClickHouse/ClickHouse/issues/44567
  - https://github.com/ClickHouse/clickhouse-docs/issues/1392
* SurrealDB: https://github.com/surrealdb/surrealdb/issues/1547
* Skytable: https://github.com/skytable/skytable/issues/300
* TiKV: https://github.com/tikv/tikv/issues/13990
  - RocksDB in TiKV: https://github.com/tikv/rocksdb/issues/339
* MongoDB:
  - https://feedback.mongodb.com/forums/924280-database/suggestions/46127320-build-mongodb-with-pgo
  - https://jira.mongodb.org/browse/SERVER-3733
  - https://www.mongodb.com/community/forums/t/profile-guided-optimization-pgo-and-mongodb/237501
* Greptimedb: https://github.com/GreptimeTeam/greptimedb/issues/1218
* Redis: https://github.com/redis/redis/issues/12371
* KeyDB: https://github.com/Snapchat/KeyDB/issues/683
* Memcached: https://github.com/memcached/memcached/issues/1054
* Databend:
  - https://github.com/datafuselabs/databend/issues/9387
  - https://github.com/datafuselabs/databend/issues/12318
* RocksDB:
  - https://groups.google.com/g/rocksdb/c/j9iMFskUnpA
  - Created a post on https://www.facebook.com/groups/rocksdb.dev/
* LevelDB: https://github.com/google/leveldb/issues/1133
* FoundationDB:
  - https://github.com/apple/foundationdb/issues/1334
  - https://forums.foundationdb.org/t/profile-guided-optimization/4043
* LanceDB: https://github.com/lancedb/lancedb/issues/255
* CeresDB: https://github.com/CeresDB/ceresdb/issues/1051
* CnosDB: https://github.com/cnosdb/cnosdb/issues/1327
* ReDB: https://github.com/cberner/redb/issues/638
* Memgraph: https://github.com/memgraph/memgraph/issues/1066
* ManticoreSearch: https://github.com/manticoresoftware/manticoresearch/issues/1247
* kvrocks: https://github.com/apache/kvrocks/issues/1551
* RethinkDB: https://github.com/rethinkdb/rethinkdb/issues/7123
* BaikalDB: https://github.com/baidu/BaikalDB/issues/222
* GridDB: https://github.com/griddb/griddb/issues/425
* FlashDB: https://github.com/armink/FlashDB/issues/223
* Aerospike: https://discuss.aerospike.com/t/profile-guided-optimization-pgo/10301
* Agensgraph: https://github.com/bitnine-oss/agensgraph/issues/622
* Hydra: https://github.com/hydradatabase/hydra/issues/108
* RonDB: https://github.com/logicalclocks/rondb/issues/335
* Kunlun: https://github.com/zettadb/kunlun/issues/912
* Lingo-DB: https://github.com/lingo-db/lingo-db/issues/2
* Cachegrand: https://github.com/danielealbano/cachegrand/issues/430
* Citus: https://github.com/citusdata/citus/issues/7046
* SplinterDB: https://github.com/vmware/splinterdb/issues/589
* TDengine: https://github.com/taosdata/TDengine/issues/21979
* MonetDB: https://github.com/MonetDB/MonetDB/issues/7392
* Neon: https://github.com/neondatabase/neon/issues/4649
* YTSaurus: https://github.com/ytsaurus/ytsaurus/issues/40
* DuckDB: https://github.com/duckdb/duckdb/discussions/7721
* EventStoreDb: https://github.com/EventStore/EventStore/issues/3897
* sqld: https://github.com/libsql/sqld/issues/524
* mvsqlite: https://github.com/losfair/mvsqlite/issues/118
* Qdrant: https://github.com/qdrant/qdrant/issues/2354
* Meilisearch: https://github.com/meilisearch/meilisearch/issues/3958
* Percona: https://forums.percona.com/t/profile-guided-optimization-pgo-for-databases/24035
* pgvector: https://github.com/pgvector/pgvector/issues/168
* GreenplumDB: https://github.com/greenplum-db/gpdb/issues/16046
* Speedb: https://github.com/speedb-io/speedb/issues/602
* OrioleDB: https://github.com/orioledb/postgres/issues/4
* InfluxDB: https://github.com/influxdata/influxdb/issues/24398
* Firebird: https://github.com/FirebirdSQL/firebird/issues/7631
* Groonga: https://github.com/groonga/groonga/issues/1585
* Typesense: https://github.com/typesense/typesense/issues/1103
* SQLite: https://sqlite.org/forum/forumpost/7fd3c9a43a
* Polars: https://github.com/pola-rs/polars/issues/9702
* pg_embedding: https://github.com/neondatabase/pg_embedding/issues/29
* CachewDB: https://github.com/theopfr/cachew-db/issues/1
* pgvector.rs: https://github.com/tensorchord/pgvecto.rs/issues/43
* SpacetimeDB: https://github.com/clockworklabs/SpacetimeDB/issues/174
* libmdbx: https://gitflic.ru/project/erthink/libmdbx/issue/14
* Risingwave: https://github.com/risingwavelabs/risingwave/issues/11859
* Pika: https://github.com/OpenAtomFoundation/pika/issues/1300
* cr-sqlite: https://github.com/vlcn-io/cr-sqlite/issues/345
* doris: https://github.com/apache/doris/issues/7657
* IndraDB: https://github.com/indradb/indradb/issues/309
* CozoDB: https://github.com/cozodb/cozo/issues/183
* Atomic server: https://github.com/atomicdata-dev/atomic-server/issues/662
* Materialize: https://github.com/MaterializeInc/materialize/discussions/21759
* TerminusDB: https://github.com/terminusdb/terminusdb-store/issues/143
* OceanBase: https://github.com/oceanbase/oceanbase/issues/1580
* Age: https://github.com/apache/age/discussions/1249
* rUniversalDB: https://github.com/pasindumuth/rUniversalDB/issues/1
* ParadeDB: https://github.com/paradedb/paradedb/issues/359
* Toshi: https://github.com/toshi-search/Toshi/issues/981
* chdb: https://github.com/chdb-io/chdb/issues/124
* Cube: https://github.com/cube-js/cube/issues/7354
* PostgresML: https://github.com/postgresml/postgresml/issues/1131
* Perspective: https://github.com/finos/perspective/issues/2405
* OpenMLDB: https://github.com/4paradigm/OpenMLDB/issues/3575
* ustore: https://github.com/unum-cloud/ustore/discussions/435
* HeavyDB: https://github.com/heavyai/heavydb/issues/818
* pgbouncer: https://github.com/pgbouncer/pgbouncer/discussions/990

## Logging

* Vector: https://github.com/vectordotdev/vector/issues/15631
* Fluent-bit: https://github.com/fluent/fluent-bit/issues/6577
* rsyslog: https://github.com/rsyslog/rsyslog/issues/5048
* syslog-ng: https://github.com/syslog-ng/syslog-ng/issues/4511
* parseable: https://github.com/parseablehq/parseable/issues/224

## Machine learning

* Tensorflow: https://github.com/tensorflow/tensorflow/issues/60944
* Tensorflow Serving: https://github.com/tensorflow/serving/issues/2192
* Tensorflow Text: https://github.com/tensorflow/text/issues/1227
* PyTorch:
  - https://github.com/pytorch/pytorch/issues/105281
  - https://github.com/pytorch/pytorch/issues/52655
* PyTorch TensorRT: https://github.com/pytorch/TensorRT/issues/2511
* Nvidia TensorRT: https://github.com/NVIDIA/TensorRT/issues/3512
* Glow: https://github.com/pytorch/glow/issues/6106
* PlaidML: https://github.com/plaidml/plaidml/issues/1996
* OpenVino: https://github.com/openvinotoolkit/openvino/issues/18572
* ApacheTVM: https://discuss.tvm.apache.org/t/profile-guided-optimization-pgo-and-apache-tvm/15332
* OnnxRuntime: https://github.com/microsoft/onnxruntime/discussions/16726
* Triton (OpenAI): https://github.com/openai/triton/issues/2288
* Triton (Nvidia): https://github.com/triton-inference-server/server/issues/6304
* Fuser: https://github.com/NVIDIA/Fuser/issues/1185

## Message brokers

* Redpanda: https://github.com/redpanda-data/redpanda/issues/7945
* BlazingMQ: https://github.com/bloomberg/blazingmq/issues/42
* rumqttd: https://github.com/bytebeamio/rumqtt/discussions/717
* Akasa: https://github.com/akasamq/akasa/issues/3

## Proxies

* Envoy: https://github.com/envoyproxy/envoy/issues/25500
* linkerd-proxy: https://github.com/linkerd/linkerd2/issues/10308
* Haproxy: https://github.com/haproxy/haproxy/issues/2047
* Rathole: https://github.com/rapiz1/rathole/discussions/287
* TinyWebServer: https://github.com/qinguoyi/TinyWebServer/issues/247
* Quilkin: https://github.com/googleforgames/quilkin/issues/834
* g3: https://github.com/bytedance/g3/issues/126
* Angie: https://github.com/webserver-llc/angie/issues/51
* Nginx VTS: https://github.com/vozlt/nginx-module-vts/issues/283

## Other

* Alacritty: https://github.com/alacritty/alacritty/issues/6598
* Tesseract: https://github.com/tesseract-ocr/tesseract/issues/3744
* OpenObserve: https://github.com/openobserve/openobserve/issues/911
* Nushell: https://github.com/nushell/nushell/issues/6943
* PgCat: https://github.com/postgresml/pgcat/issues/479
* Odyssey: https://github.com/yandex/odyssey/issues/507
* Tauri: https://github.com/tauri-apps/tauri/issues/7284
* Godot: https://github.com/godotengine/godot-proposals/issues/2610
* Rspamd: https://github.com/rspamd/rspamd/issues/4534
* Arroyo: https://github.com/ArroyoSystems/arroyo/issues/202
* ClamAV: https://github.com/Cisco-Talos/clamav/issues/1016
* OpenZFS: https://github.com/openzfs/zfs/issues/15069
* uutils: https://github.com/uutils/coreutils/issues/2730
* CephFS: https://tracker.ceph.com/issues/62032
* Prql: https://github.com/PRQL/prql/issues/3116
* Typst: https://github.com/typst/typst/issues/1733
* Tectonic: https://github.com/tectonic-typesetting/tectonic/issues/1072
* Miktex: https://github.com/MiKTeX/miktex/issues/1360
* Nebula: https://github.com/vesoft-inc/nebula/issues/5661
* Netdata: https://github.com/netdata/netdata/issues/15338#issuecomment-1629460584
* Wazuh: https://github.com/wazuh/wazuh/issues/18151
* FreeSwitch: https://github.com/signalwire/freeswitch/issues/2186
* Dora: https://github.com/dora-rs/dora/issues/331
* mozjpeg: https://github.com/mozilla/mozjpeg/issues/169
* Godbolt: https://github.com/compiler-explorer/compiler-explorer/issues/5414
* Bloop: https://github.com/BloopAI/bloop/issues/898
* Lychee: https://github.com/lycheeverse/lychee/issues/1247
* GraphScope: https://github.com/alibaba/GraphScope/issues/3173
* [Spin](https://github.com/fermyon/spin): didn't create an issue yet since Rustc does not support PGO via profiling in WASM (https://github.com/rust-lang/rust/issues/81684)
* Foundry-rs: didn't create a PGO issue since LTO + PGO build is broken: https://github.com/rust-lang/rust/issues/115344#issuecomment-1703825138
* Sui: https://github.com/MystenLabs/sui/discussions/13603
* hurl: https://github.com/Orange-OpenSource/hurl/issues/1918
* fd: https://github.com/sharkdp/fd/discussions/1384
* Thalassa: https://github.com/a-croco-stolyarov/thalassa/issues/1
* [dust](https://github.com/bootandy/dust): didn't test since PGO build is broken while enabling LTO on Linux machine
* broot: https://github.com/Canop/broot/issues/741
* Massa: https://github.com/massalabs/massa/discussions/4407
* Fuel-core: https://github.com/FuelLabs/fuel-core/issues/1369
* PuzzleFS: https://github.com/project-machine/puzzlefs/issues/113
* Proton: https://github.com/timeplus-io/proton/issues/122
* NFSserve: https://github.com/xetdata/nfsserve/issues/4
* Youki: https://github.com/containers/youki/issues/2386
* Hyperswitch: https://github.com/juspay/hyperswitch/issues/2399
* ClickHouse tooling: https://github.com/ClickHouse/ClickHouse/discussions/55198
* Eza: https://github.com/eza-community/eza/issues/479
* sd: https://github.com/chmln/sd/issues/237
* bat: https://github.com/sharkdp/bat/issues/2701
* jql: https://github.com/yamafaktory/jql/issues/236
* htmlq: https://github.com/mgdm/htmlq/issues/70
* ouch: https://github.com/ouch-org/ouch/issues/537
* czkawka: https://github.com/qarmin/czkawka/issues/1099
* blaze: https://github.com/blaze-init/blaze/issues/275
* gluten: https://github.com/oap-project/gluten/issues/3478
* difftastic: https://github.com/Wilfred/difftastic/issues/588
* Solidity: https://github.com/ethereum/solidity/issues/11085
* legba: https://github.com/evilsocket/legba/issues/10
* Slint: https://github.com/slint-ui/slint/issues/3909
* Clingo: https://github.com/potassco/clingo/issues/468
* Mesa: https://gitlab.freedesktop.org/mesa/mesa/-/issues/10155
* lingua-rs: https://github.com/pemistahl/lingua-rs/discussions/273
* tokei: https://github.com/XAMPPRocky/tokei/issues/1039
* qsv: https://github.com/jqnatividad/qsv/discussions/1433
* pydantic-core: https://github.com/pydantic/pydantic-core/issues/732
* vtracer: https://github.com/visioncortex/vtracer/discussions/67
* Qt: https://bugreports.qt.io/browse/QTCREATORBUG-29968
* fish: https://github.com/fish-shell/fish-shell/discussions/10119
* Lapce: https://github.com/lapce/lapce/discussions/2817
* OpenSIPS: https://github.com/OpenSIPS/opensips/issues/3261
* workerd: https://github.com/cloudflare/workerd/discussions/1463
* Carla: https://github.com/carla-simulator/carla/discussions/6970
* or-tools: https://github.com/google/or-tools/discussions/4012

## PGO in Go ecosystem

PGO was added to Go in 1.20 (Preview) and 1.21 (GA). Here we track PGO adoption across the Go ecosystem.

Go proposal about dumping PGO profiles at exit: https://github.com/golang/go/issues/62444

* pushup: https://github.com/adhocteam/pushup/issues/106
* goProbe: https://github.com/els0r/goProbe/issues/170
* rymdport: https://github.com/Jacalz/rymdport/issues/109

## Are we PGO yet in OS repositories

Here we track information on enablement PGO in OS-specific package build scripts

* Clang:
  - Fedora: https://bugzilla.redhat.com/show_bug.cgi?id=2156679

## PGO documentation

Here we track the issues regarding PGO documentation in different projects:

* Fluent-Bit: https://github.com/fluent/fluent-bit/discussions/6638#discussioncomment-6822318
* YugabyteDB: https://github.com/yugabyte/yugabyte-db/issues/18497
* Rsyslog: https://github.com/rsyslog/rsyslog/issues/5048#issuecomment-1694349501
* Rustc:
  - https://github.com/rust-lang/rust/issues/115251
  - https://github.com/rust-lang/rust/issues/114995
  - https://github.com/rust-lang/rust/issues/117023
* GreptimeDB: https://github.com/GreptimeTeam/docs/issues/544
* RonDB: https://github.com/logicalclocks/rondb/issues/335#issuecomment-1695636890
* ReDB: https://github.com/cberner/redb/issues/638#issuecomment-1695601594
* Memcached: https://github.com/memcached/memcached/issues/1054#issuecomment-1696426432
* Qdrant: TODO

## BOLT package availability in repositories

Here we track LLVM BOLT existence progress for Linux distributions. It can be checked with this Repology query: https://repology.org/project/llvm-bolt/versions

* Fedora: supports
* Solus: supports
* Chromebrew: https://github.com/chromebrew/chromebrew/issues/8940
* Gentoo: https://bugs.gentoo.org/895326
* Void Linux: https://github.com/void-linux/void-packages/issues/47226
* NixOS: https://github.com/NixOS/nixpkgs/issues/176536
* Alpine Linux: https://gitlab.alpinelinux.org/alpine/aports/-/issues/15472
* Mageia: https://bugs.mageia.org/show_bug.cgi?id=32519
* Spack: https://github.com/spack/spack/issues/41088

## BOLT documentation

Here we track the issues regarding BOLT documentation in different projects:

* Clang:
  - https://github.com/llvm/llvm-project/issues/65010
  - https://github.com/llvm/llvm-project/issues/69846

## Other

* Rustc discussion about Propeller: https://internals.rust-lang.org/t/add-propeller-support-to-the-rustc-compiler/19757
