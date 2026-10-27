# sd.cpp-cuda

[![Build stable-diffusion.cpp with CUDA](https://github.com/mmxkarimi/sd.cpp-cuda/actions/workflows/build-cuda.yml/badge.svg)](https://github.com/mmxkarimi/sd.cpp-cuda/actions/workflows/build-cuda.yml)

Pre-built [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) (sd.cpp) binaries with **CUDA** support for multiple GPU architectures, for **Linux**.

A scheduled GitHub Actions workflow ([.github/workflows/build-cuda.yml](.github/workflows/build-cuda.yml)) watches for new upstream releases, builds them in an official `nvidia/cuda` container, and publishes the result as a GitHub Release here.

## Download

Grab the latest tarball:

```bash
curl -LO $(curl -s https://api.github.com/repos/mmxkarimi/sd.cpp.cuda/releases/latest \
  | jq -r '.assets[] | select(.name | contains("amd64.tar.gz")) | .browser_download_url')

tar -xzf sd.cpp-*-cuda-12.8-*.tar.gz && cd cuda-12.8
```

## What's inside

| File | Purpose |
| --- | --- |
| `sd-cli` | CLI for image / video generation |
| `sd-server` | Web UI server (frontend embedded at build time) |
| `libstable-diffusion.so` | sd.cpp shared library |
| CPU-variant + CUDA backend modules | ggml dynamic backends (loaded automatically via `$ORIGIN` rpath) |

Includes WebP image I/O, WebM video output, and native SageAttention (`--sage-attn`, requires compute capability ≥ 8.0).

## Build configuration

- **CUDA:** 12.8.1 (container `nvidia/cuda:12.8.1-cudnn-devel-ubuntu22.04`)
- **Compute capabilities:** `75, 80, 86, 89, 90, 100, 120` (PTX only — JIT-compiled by the driver on first run, so a recent driver, R570+, is required)
- **Backends:** `SD_CUDA=ON`, shared libs with `GGML_BACKEND_DL` + `GGML_CPU_ALL_VARIANTS` (portable across CPU generations), patched GGML submodule
- **Host arch:** amd64 (x86_64)

### GPU compute capability reference

| CC | GPUs |
| --- | --- |
| 7.5 | Tesla T4, RTX 20xx, GTX 16xx, Quadro RTX |
| 8.0 | A100 |
| 8.6 | RTX 3000 series |
| 8.9 | RTX 4000, L4, L40 |
| 9.0 | H100, H200, GH200 |
| 10.0 | B200, GB200 |
| 12.0 | RTX Pro series, RTX 50xx |

## Workflow behavior

- **Schedule:** runs daily at 00:00 UTC; builds only when a new upstream release is detected (each release is tagged here once built).
- **Manual run:** *Actions → Build stable-diffusion.cpp with CUDA → Run workflow* — set `force_build` to rebuild the current upstream release.
- Binaries are built in a container from the exact upstream release tag (with submodules), stripped, packed, and attached to a release named after the upstream tag.

## Usage

```bash
cd cuda-12.8
# Text-to-image
./sd-cli -m sd_xl_base_1.0.safetensors -p "a photo of a cat" --steps 30
# Web UI
./sd-server --models-path .
```

See the full documentation at the [upstream repository](https://github.com/leejet/stable-diffusion.cpp).

## License

Binaries are built from [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) (MIT) and its GGML dependency (MIT). This repository only contains build automation and release artifacts.
