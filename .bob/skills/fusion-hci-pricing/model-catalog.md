# Foundation Model Catalog (for VRAM sizing)

Use this table to look up parameter counts when a user names a model. Parameter count is
the single most important input to GPU sizing.

## ⚠️ Web verification protocol (read first)

This table is a **fast-path starting point, not the source of truth.** Model lineups change
constantly and any static list goes stale. Therefore:

1. **Model NOT in this table** → **search the web** for its parameter count before sizing
   (query e.g. `"<model name> parameters billion"` and prefer the model card / official
   announcement / Hugging Face). Do not guess. If you cannot find it, ask the user.
2. **Model IS in this table** → still do a **quick web double-check** of the parameter count
   as a failsafe, since the catalog may be out of date. If the web disagrees with the table,
   **trust the web**, use that number, and briefly note the correction to the user.
3. **For MoE models, verify TOTAL parameters** (and note active params separately) — total
   drives VRAM (see below).
4. If web search is unavailable in your environment, use the table value.

The numbers below are accurate as of early 2026; treat them accordingly.

## How to read this table
- **Params (B)** = billions of parameters. This drives VRAM.
- **MoE total / active** = Mixture-of-Experts. **Size VRAM on TOTAL params** — all experts
  must be resident in GPU memory — even though compute/latency behaves like the smaller
  *active* count. Sizing a MoE model on active params is the #1 sizing mistake.
- VRAM/GPU pre-computes elsewhere assume **FP16** + IBM's planning rule (params × 3, see
  SKILL.md). Always recompute if precision or context differs.

## IBM Granite (default / preferred on watsonx)
| Model | Params (B) | Notes |
|---|---|---|
| Granite 3.x 2B | 2 | Tiny; 1 GPU |
| Granite 3.x 8B | 8 | Common default; 1 GPU |
| Granite 4 Tiny | ~7 | Verify exact size |
| Granite 4 Small (granite-4-h-small) | ~32 | Verify exact size |
| Granite Code 3B / 8B / 20B / 34B | 3 / 8 / 20 / 34 | Code-assist family |
| Granite Guardian 8B | 8 | Safety/guardrail model |

## Meta Llama
| Model | Params (B) | Notes |
|---|---|---|
| Llama 3.1 / 3.3 8B | 8 | 1 GPU |
| Llama 3.1 / 3.3 70B | 70 | ~210 GB → 2 H200 |
| Llama 3.1 405B | 405 | Largest dense Llama; ~1.2 TB → 8+ H200, multi-node |
| Llama 4 Scout | 109 total (17 active, MoE) | Size on 109B |
| Llama 4 Maverick | 400 total (17 active, MoE) | Size on 400B |
| Llama 4 Behemoth | 2000 total (288 active, MoE) | **Research preview — NOT publicly released.** Cluster-scale; confirm availability before sizing |

## DeepSeek (all flagship models are 671B MoE)
| Model | Params (B) | Notes |
|---|---|---|
| DeepSeek-V3 / V3-0324 | 671 total (37 active, MoE) | 128K context |
| DeepSeek-R1 / R1-0528 | 671 total (37 active, MoE) | Reasoning; 128K context |
| DeepSeek-V3.1 | 671 total (37 active, MoE) | Hybrid think/non-think |
| DeepSeek-V4 | 671 total (37 active, MoE) | 1M context (Apr 2026) |
| DeepSeek-Prover-V2-671B | 671 total (37 active, MoE) | Theorem proving |
| DeepSeek-R1 Distill (Qwen/Llama) | 1.5 / 7 / 8 / 14 / 32 / 70 | Dense distillations; size as normal dense |

Note: there is no publicly released DeepSeek model larger than 671B; the big variants differ
in training/context, not parameter count. Full 671B weights ≈ ~700 GB at FP16/BF16.

## Alibaba Qwen
| Model | Params (B) | Notes |
|---|---|---|
| Qwen2.5 7B / 14B / 32B / 72B | 7 / 14 / 32 / 72 | Dense |
| Qwen3 dense (0.6B–32B) | 0.6 / 1.7 / 4 / 8 / 14 / 32 | Dense |
| Qwen3-30B-A3B | 30 total (3 active, MoE) | Size on 30B |
| Qwen3-235B-A22B | 235 total (22 active, MoE) | Size on 235B; ~700 GB |
| QwQ-32B | 32 | Reasoning, dense |

## Mistral
| Model | Params (B) | Notes |
|---|---|---|
| Mistral 7B | 7 | 1 GPU |
| Mistral NeMo | 12 | 1 GPU |
| Mistral Small 3 / 3.1 / 3.2 | 24 | 1 GPU |
| Codestral | 22 | Code |
| Mistral Large 2 | 123 | ~370 GB → 4 H200 |
| Mixtral 8x7B | 46.7 total (12.9 active, MoE) | Size on 46.7B |
| Mixtral 8x22B | 141 total (39 active, MoE) | Size on 141B |

## Other providers
| Model | Provider | Params (B) | Notes |
|---|---|---|---|
| Nemotron Ultra (Llama-3.1-based) | NVIDIA | 253 | Dense; verify variant |
| Llama-3.3-Nemotron Super 49B | NVIDIA | 49 | Dense |
| Command R | Cohere | 35 | Dense |
| Command R+ | Cohere | 104 | Dense |
| Command A | Cohere | 111 | Dense; verify |
| Jamba 1.5 Mini | AI21 | 52 total (12 active, MoE) | Hybrid SSM-Transformer; size on 52B |
| Jamba 1.5 Large | AI21 | 398 total (94 active, MoE) | Size on 398B |
| Falcon 180B | TII | 180 | Dense |
| Falcon3 / Falcon-H1 | TII | varies (1–40) | Verify variant |
| Yi-34B | 01.AI | 34 | Dense |
| Gemma 2 | Google | 9 / 27 | Dense |
| Gemma 3 | Google | 1 / 4 / 12 / 27 | Dense, multimodal |
| Phi-4 | Microsoft | 14 | Dense |
| Hunyuan-Large | Tencent | 389 total (52 active, MoE) | Size on 389B |

(For any model here, still apply the web double-check in the protocol above.)

## Precision quick reference (bytes per parameter)
| Precision | Bytes/param | VRAM for weights (per 1B params) |
|---|---|---|
| FP32 | 4 | 4 GB |
| **FP16 / BF16 (default)** | 2 | 2 GB |
| FP8 / INT8 | 1 | 1 GB |
| INT4 / 4-bit | 0.5 | 0.5 GB |

H200 (Hopper) supports down to FP8 in hardware but **not FP4** (Blackwell/B200 required).
Quantized weights reduce the *weight* portion of VRAM; the KV cache is unaffected unless
KV-cache quantization is also used.
