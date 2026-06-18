# Fork Purpose

This is a personal fork of [`ggml-org/llama.cpp`](https://github.com/ggml-org/llama.cpp) maintained for a specific hardware and model compatibility requirement in a private homelab environment. It is not intended for general use.

## Branch Overview

### `homelab`

This branch contains two custom commits applied on top of the upstream `b9548` snapshot (2026-06-07) to enable functional Gemma 4 vision on a Tesla V100:

1. **Gemma 4 12B vision fix** — resolves two bugs ([issue #24146](https://github.com/ggml-org/llama.cpp/issues/24146)) that caused Gemma 4 vision to emit `<unused49>` tokens instead of real text:
   - F16 overflow in `patch_embeddings_0` / `mm_input_proj_w` matmuls for high-contrast patches — fixed by forcing `GGML_PREC_F32`.
   - Wrong patch count in the `dyn_size` preprocessor — fixed by a new `mtmd_image_preprocessor_gemma4` that calls `calc_size_fill_budget()`.
   - Based on [chippydip's `gemma4uv-vision-fix` branch](https://github.com/chippydip/llama.cpp/tree/gemma4uv-vision-fix).

2. **Build fix for b9549** — upstream `b9549` made `clip_image_u8::nx/ny` private (adding `get_size()`/`set_size()` accessors). This commit updates the Gemma 4 preprocessor to use the new accessor so the branch still compiles.

### `master`

Vanilla upstream snapshot — zero custom commits. Exists only as the fork base.

## How it's used

`loon-ai-01` (Ubuntu 24.04, Ryzen 5 5500, Tesla V100 32 GB) must build llama.cpp for `sm_70` with CUDA 12.8 — CUDA 13 dropped Volta support. The bootstrap script `02-cuda-binaries.sh` in [`declanlooney/localllm`](https://github.com/declanlooney/localllm) clones this repo at the `homelab` branch and compiles a CUDA binary for the V100.

The compiled `llama-server` backs the `gemma-4-26b-chat` model (Q6\_K MoE + vision) running through llama-swap on `loon-ai-01`. Without the Gemma 4 vision patch, vision inference produces garbage tokens.

## Maintenance

**Retirement criteria:** once [ggml-org/llama.cpp#24146](https://github.com/ggml-org/llama.cpp/issues/24146) is closed and the fix is in an upstream release, this fork can be retired. The `homelab` branch should be rebased onto the upstream commit that includes the fix, the binary rebuilt on `loon-ai-01`, and `02-cuda-binaries.sh` updated to point at upstream.

The `homelab` branch is currently ~160 commits behind upstream master. The `master` branch has no custom changes and can be discarded or synced freely.
