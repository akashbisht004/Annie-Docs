# Annie.io

Blazingly fast Approximate Nearest Neighbors in Rust

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

## Basic Usage























































































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

## Key Features























































































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

## Navigation

- [API Reference](api/ann_index.md) - Core classes and methods
- [Examples](examples.md) - Usage examples and tutorials
- [Concurrency](concurrency.md) - Thread-safe operations
- [Filtering](filtering.md) - Custom search filters

## Quick Links

- [GitHub Repository](https://github.com/Programmers-Paradise/Annie)
- [PyPI Package](https://pypi.org/project/rust-annie/)
- [Benchmark Results](https://programmers-paradise.github.io/Annie/)
