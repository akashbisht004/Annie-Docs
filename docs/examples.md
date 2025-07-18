# Annie Examples

## Basic Usage
```python
import numpy as np
from rust_annie import AnnIndex, Distance

# Create index
index = AnnIndex(128, Distance.EUCLIDEAN)

# Generate and add data
data = np.random.rand(1000, 128).astype(np.float32)
ids = np.arange(1000, dtype=np.int64)
index.add(data, ids)

# Single query
query = np.random.rand(128).astype(np.float32)
neighbor_ids, distances = index.search(query, k=5)

# Batch queries
queries = np.random.rand(10, 128).astype(np.float32)
batch_ids, batch_dists = index.search_batch(queries, k=3)
```

## Filtered Search
```python
# Create index with sample data
index = AnnIndex(3, Distance.EUCLIDEAN)
data = np.array([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0]
], dtype=np.float32)
ids = np.array([10, 20, 30], dtype=np.int64)
index.add(data, ids)

# Define filter function
def even_ids(id: int) -> bool:
    return id % 2 == 0

# Filtered search
query = np.array([1.0, 2.0, 3.0], dtype=np.float32)
filtered_ids, filtered_dists = index.search_filter_py(query, k=3, filter_fn=even_ids)
# Only IDs 10 and 30 will be returned (20 is odd)
```

## HNSW Index
```python
from rust_annie import PyHnswIndex

# Create HNSW index
index = PyHnswIndex(dims=128)

# Add large dataset
data = np.random.rand(100000, 128).astype(np.float32)
ids = np.arange(100000, dtype=np.int64)
index.add(data, ids)

# Fast approximate search
query = np.random.rand(128).astype(np.float32)
neighbor_ids, _ = index.search(query, k=10)
```

## Saving and Loading
```python
# Create and save index
index = AnnIndex(64, Distance.COSINE)
data = np.random.rand(500, 64).astype(np.float32)
ids = np.arange(500, dtype=np.int64)
index.add(data, ids)
index.save("my_index")

# Load index
loaded_index = AnnIndex.load("my_index")
```

## Thread-safe Operations
```python
from rust_annie import ThreadSafeAnnIndex, Distance
from concurrent.futures import ThreadPoolExecutor

index = ThreadSafeAnnIndex(256, Distance.MANHATTAN)

# Concurrent writes
with ThreadPoolExecutor() as executor:
    for i in range(10):
        data = np.random.rand(100, 256).astype(np.float32)
        ids = np.arange(i*100, (i+1)*100, dtype=np.int64)
        executor.submit(index.add, data, ids)

# Concurrent reads
with ThreadPoolExecutor() as executor:
    futures = []
    for _ in range(100):
        query = np.random.rand(256).astype(np.float32)
        futures.append(executor.submit(index.search, query, k=3))
    
    results = [f.result() for f in futures]
```

## Minkowski Distance
```python
# Create index with custom distance
index = AnnIndex.new_minkowski(dim=64, p=2.5)
data = np.random.rand(200, 64).astype(np.float32)
ids = np.arange(200, dtype=np.int64)
index.add(data, ids)

# Search with Minkowski distance
query = np.random.rand(64).astype(np.float32)
ids, dists = index.search(query, k=5)
```

# README

![Annie](https://github.com/Programmers-Paradise/.github/blob/main/ChatGPT%20Image%20May%2015,%202025,%2003_58_16%20PM.png?raw=true)

[![PyPI](https://img.shields.io/pypi/v/rust-annie.svg)](https://pypi.org/project/rust-annie)  
[![CI](https://img.shields.io/badge/Workflow-CI-white.svg)](https://github.com/Programmers-Paradise/Annie/blob/main/.github/workflows/CI.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Benchmark](https://img.shields.io/badge/benchmark-online-blue.svg)](https://programmers-paradise.github.io/Annie/)
[![GPU Support](https://img.shields.io/badge/GPU-CUDA-green.svg)](https://github.com/Programmers-Paradise/Annie/pull/53)
[![Documentation](https://img.shields.io/badge/docs-online-purple.svg)](https://programmers-paradise.github.io/Annie/)

A lightning-fast, Rust-powered Approximate Nearest Neighbor library for Python with multiple backends, thread-safety, and GPU acceleration.

## Table of Contents

1. [Features](#features)  
2. [Installation](#installation)  
3. [Quick Start](#quick-start)  
4. [Examples](#examples)  
   - [Brute-Force Index](#brute-force-index)  
   - [HNSW Index](#hnsw-index)  
   - [Thread-Safe Index](#thread-safe-index)  
5. [Benchmark Results](#benchmark-results)  
6. [API Reference](#api-reference)  
7. [Development & CI](#development--ci)  
8. [GPU Acceleration](#gpu-acceleration)
9. [Documentation](#documentation)
10. [Contributing](#contributing)  
11. [License](#license)

## Features

- **Multiple Backends**:
  - **Brute-force** (exact) with SIMD acceleration
  - **HNSW** (approximate) for large-scale datasets
- **Multiple Distance Metrics**: Euclidean, Cosine, Manhattan, Chebyshev
- **Batch Queries** for efficient processing
- **Thread-safe** indexes with concurrent access
- **Zero-copy** NumPy integration
- **On-disk Persistence** with serialization
- **Filtered Search** with custom Python callbacks
- **GPU Acceleration** for brute-force calculations
- **Multi-platform** support (Linux, Windows, macOS)
- **Automated CI** with performance tracking

## Installation

```bash
# Stable release from PyPI:
pip install rust-annie

# Install with GPU support (requires CUDA):
pip install rust-annie[gpu]

# Or install from source:
git clone https://github.com/Programmers-Paradise/Annie.git
cd Annie
pip install maturin
maturin develop --release
```

## Quick Start

### Brute-Force Index
```python
import numpy as np
from rust_annie import AnnIndex, Distance

# Create index
index = AnnIndex(128, Distance.EUCLIDEAN)

# Add data
data = np.random.rand(1000, 128).astype(np.float32)
ids = np.arange(1000, dtype=np.int64)
index.add(data, ids)

# Search
query = np.random.rand(128).astype(np.float32)
neighbor_ids, distances = index.search(query, k=5)
```

### HNSW Index
```python 
from rust_annie import PyHnswIndex

index = PyHnswIndex(dims=128)
data = np.random.rand(10000, 128).astype(np.float32)
ids = np.arange(10000, dtype=np.int64)
index.add(data, ids)

# Search
query = np.random.rand(128).astype(np.float32)
neighbor_ids, _ = index.search(query, k=10)
```

## Examples

### Brute-Force Index
```python
from rust_annie import AnnIndex, Distance
import numpy as np

# Create index
idx = AnnIndex(4, Distance.COSINE)

# Add data
data = np.random.rand(50, 4).astype(np.float32)
ids = np.arange(50, dtype=np.int64)
idx.add(data, ids)

# Search
labels, dists = idx.search(data[10], k=3)
print(labels, dists)
```

### Batch Query
```python
from rust_annie import AnnIndex, Distance
import numpy as np

# Create index
idx = AnnIndex(16, Distance.EUCLIDEAN)

# Add data
data = np.random.rand(1000, 16).astype(np.float32)
ids = np.arange(1000, dtype=np.int64)
idx.add(data, ids)

# Batch search
queries = data[:32]
labels_batch, dists_batch = idx.search_batch(queries, k=10)
print(labels_batch.shape)  # (32, 10)
```

### Thread-Safe Index
```python
from rust_annie import ThreadSafeAnnIndex, Distance
import numpy as np
from concurrent.futures import ThreadPoolExecutor

# Create thread-safe index
idx = ThreadSafeAnnIndex(32, Distance.EUCLIDEAN)

# Add data
data = np.random.rand(500, 32).astype(np.float32)
ids = np.arange(500, dtype=np.int64)
idx.add(data, ids)

# Concurrent searches
def task(q):
    return idx.search(q, k=5)

queries = np.random.rand(100, 32).astype(np.float32)
with ThreadPoolExecutor(max_workers=8) as executor:
    futures = [executor.submit(task, q) for q in queries]
    for f in futures:
        print(f.result())
```

### Filtered Search
```python
from rust_annie import AnnIndex, Distance
import numpy as np

# Create index
index = AnnIndex(3, Distance.EUCLIDEAN)
data = np.array([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0]
], dtype=np.float32)
ids = np.array([10, 20, 30], dtype=np.int64)
index.add(data, ids)

# Filter function
def even_ids(id: int) -> bool:
    return id % 2 == 0

# Filtered search
query = np.array([1.0, 2.0, 3.0], dtype=np.float32)
filtered_ids, filtered_dists = index.search_filter_py(
    query, 
    k=3, 
    filter_fn=even_ids
)
print(filtered_ids)  # [10, 30] (20 is filtered out)
```

## Build and Query a Brute-Force AnnIndex in Python (Complete Example)

This section demonstrates a complete, beginner-friendly example of how to build and query a `brute-force AnnIndex` using Python.

Measured on a 6-core CPU:

That’s a \~4× speedup vs. NumPy!

| Operation	           | Dataset Size  | Time (ms) | Speedup vs Python | 
| -------------------- | ------------- | --------- | ----------------- | 
| Single Query (Brute) | 10,000 × 64   | 0.7	   | 4×                | 
| Batch Query (64)	   | 10,000 × 64   | 0.23	   | 12×               | 
| HNSW Query	       | 100,000 × 128 | 0.05	   | 56×               |

##### [View Full Benchmark Dashboard →](https://programmers-paradise.github.io/Annie/)

You’ll find:

## API Reference

### AnnIndex

Create a brute-force k-NN index.

Enum: `Distance.EUCLIDEAN`, `Distance.COSINE`, `Distance.MANHATTAN`

### ThreadSafeAnnIndex

Same API as `AnnIndex`, safe for concurrent use.

### Core Classes

| Class              | Description                                |
| ------------------ | ------------------------------------------ |
| AnnIndex	         | Brute-force exact search                   |
| PyHnswIndex	     | Approximate HNSW index                     |
| ThreadSafeAnnIndex | 	Thread-safe wrapper for AnnIndex          |
| Distance           | 	Distance metrics (Euclidean, Cosine, etc) |

## Key Methods

| Method                                | Description                                | 
| ------------------------------------- | ------------------------------------------ |
| add(data, ids)	                    | Add vectors to index                       | 
| search(query, k)	                    | Single query search                        | 
| search_batch(queries, k)              | Batch query search                         | 
| search_filter_py(query, k, filter_fn) | Filtered search                            | 
| save(path)                            | Save index to disk                         | 
| load(path)                            | Load index from disk                       | 

## Development & CI

**CI** runs on GitHub Actions, building wheels on Linux, Windows, macOS, plus:

* `benchmark.py` & `batch_benchmark.py` & `compare_results.py`

```bash
# Run tests
cargo test
pytest tests/

# Run benchmarks
python scripts/benchmark.py
python scripts/batch_benchmark.py

# Generate documentation
mkdocs build
```

CI pipeline includes:
  - Cross-platform builds (Linux, Windows, macOS)
  - Unit tests and integration tests
  - Performance benchmarking
  - Documentation generation

### Benchmark Automation

Benchmarks are tracked over time using:

## GPU Acceleration

### Enable GPU in Rust

Enable CUDA support for brute-force calculations:
```bash
# Install with GPU support
pip install rust-annie[gpu]

# Or build from source with GPU features
maturin develop --release --features gpu
```

Supported operations:
  - Batch L2 distance calculations
  - High-dimensional similarity search

Requirements:
  - NVIDIA GPU with CUDA support
  - CUDA Toolkit installed

## Contributing

Contributions are welcome! Please:

See [../docs-management/CONTRIBUTING.md](../docs-management/CONTRIBUTING.md) for details.

## License

This project is licensed under the **MIT License**. See [LICENSE](./LICENSE) for details.