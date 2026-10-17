# Environment

Recorded from `results/laptop-environment.json` and each seed's
`results/seed*/manifest.json`, not retyped from memory.

| Component | Value |
|---|---|
| Hardware | i5-13450HX, NVIDIA GeForce RTX 3050 6GB Laptop GPU, 24GB RAM |
| OS | Linux Mint 22.3 (`Linux-7.0.0-31-generic-x86_64-with-glibc2.39`) |
| Python | 3.12.3 (GCC 13.3.0) |
| NVIDIA driver / CUDA | CUDA 12.4 wheel (`torch==2.5.1+cu124`) |
| PyTorch | 2.5.1+cu124 |
| torchvision | 0.20.1+cu124 |
| NumPy | 1.26.4 |
| Precision | FP32, TF32 disabled, no AMP (`torch.backends.cuda.matmul.allow_tf32 = False`, `cudnn.allow_tf32 = False`) |
| Determinism | `cudnn.deterministic = True`, `cudnn.benchmark = False`, `torch.use_deterministic_algorithms(True)`, `CUBLAS_WORKSPACE_CONFIG=:4096:8` |

GPU driver version itself was not captured in any committed log (only
`nvidia-smi`'s `name, memory.total` query, see `results/laptop-environment.json`
`gpu_query` field) — noted here rather than invented.

CPU count: 16 logical. Total memory: `MemTotal: 24373188 kB` (~24GB).
