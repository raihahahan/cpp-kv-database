# Introduction

The aim of this project is to build a durable key-value database supporting two different storage engines: LSM-tree and B-tree.

From Gemini: An LSM tree (Log-Structured Merge-Tree) is a data structure optimized for write-heavy workloads, commonly used in databases and file systems. It prioritizes efficient sequential writes by buffering data in memory (Memtable) before flushing it to disk in sorted order (SSTables). This approach minimizes random disk I/O, which is slower than sequential access.

([Link to writeup](https://mraihan.dev/blog/LSM-tree-Key-Value-Store-in-CPP))

## Current Progress and Features

### LSM-tree version ✅

1. Memtable with skiplist ✅
2. Periodic flush to disk with segments of SSTables ✅
3. Periodic compaction to merge the SSTable segments ✅

### B-tree version (pending)

1. B-tree data structure
2. Page manager
3. Buffer manager

### Common ✅

1. Write-ahead log ✅
2. CLI ✅
3. Turn it into a daemon process ✅

# Pre-requisites

- CMake – for configuring the build system
- Make
- C++20-compatible compiler
  - macOS: Apple Clang
  - Linux: GCC or Clang

# Setup

## 1. Install Catch2 for testing

```
$ git clone https://github.com/catchorg/Catch2.git
$ cd Catch2
$ cmake -B build -S . -DBUILD_TESTING=OFF
$ sudo cmake --build build/ --target install
```

If you do not have superuser rights, you will also need to specify [CMAKE_INSTALL_PREFIX](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) when configuring the build, and then modify your calls to [find_package](https://cmake.org/cmake/help/latest/command/find_package.html) accordingly.

More instructions here: https://github.com/catchorg/Catch2/blob/devel/docs/cmake-integration.md#cmake-targets. Make sure to be at `devel` branch.

## 2. Makefiles

```
make           # Build (creates build/ and runs CMake+Make)
make run       # Build and run db_main
make clean     # Delete build/
make build     # Builds without tests
make test      # Builds and tests immediately
make rebuild   # Full clean + rebuild

```

## 3. Test CLI

Edit config in `config.hpp` before building.

```
cd build
./db_main # run the daemon process
./db_cli  # run the cli to interact with the daemon process
./stess_test # stress test the daemon
```

# Project overview

This project is divided into three phases:

1. **Phase 1:** Key-Value Store with LSM-Tree Storage ✅
2. **Phase 2:** Key-Value Store with B-Tree Storage
3. **Phase 3:** Turn it into a daemon process and allow communication via sockets ✅

# Phase 1: Key-Value Store with LSM-Tree Storage ✅

The first step is to build a basic key-value store using an LSM-tree storage engine, similar to LevelDB.

## Components

- **Write-Ahead Log (WAL):** Appends every write to disk for durability
- **Memtable:** An in-memory balanced tree (e.g. std::map) holding recent writes
- **SSTables:** Immutable, sorted files flushed from the memtable
- **Compactor:** Merges SSTables and drops overwritten/deleted keys

The get path checks the memtable, then recent SSTables, and so on. Each SSTable has a sparse in-memory index to reduce scan overhead. This gives me a fast, durable, and append-only key-value store.

# Phase 2: Key-Value Store with B-Tree Storage

Once the LSM version is working, I plan to add a second storage engine based on B-trees. This engine will share the same StorageEngine interface as the LSM version.

## Why B-Trees?

- Better read performance for point and range queries
- More suitable for in-place updates
- Serves as a foundation for future ACID semantics (via pages and MVCC) (future plans to fork this repo to create a relational version)

## Components

The B-tree structure enables efficient lookups and range scans, with nodes aligned to page boundaries for cache and I/O efficiency.

- **BTree Nodes:** Each node stores sorted keys and child pointers
- **Pager:** Manages reading/writing pages to disk
- **Buffer Manager**: Caches pages in memory and handles eviction

# Phase 3: Daemon process ✅

The goal is to allow this database to run as a background process, and the CLI tool allows users to choose the storage engine (B-tree or LSM-tree) and connect to the database via sockets.
