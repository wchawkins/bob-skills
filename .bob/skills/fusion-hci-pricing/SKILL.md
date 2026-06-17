---
name: fusion-hci-pricing
description: Size and price an IBM Fusion HCI cluster, including GPU sizing for Watsonx / LLM inference workloads. Use this skill whenever a user asks about Fusion HCI pricing, cost, or sizing; whenever they want to run, host, serve, or deploy one or more LLMs / foundation models and need hardware for it; or whenever they ask "how many GPUs do I need" or "what cluster do I need to run model X." Trigger this even when the user only names models they want to run (e.g. "I need to serve Llama 3.3 70B and Granite 8B — what do I need?") without saying the words "Fusion" or "sizing." Developers often describe workloads in terms of models, not hardware — treat that as a sizing request.
---

# IBM Fusion HCI Sizing & Pricing

This skill sizes and prices an IBM Fusion HCI cluster against the **IBM BPv4.6 T-Shirt
Calculator** (`tshirt-calculator.md`). It supports two kinds of request:

- **Path A — Traditional infra sizing:** the user gives vCPUs, memory, storage, GPU yes/no.
- **Path B — LLM / Watsonx sizing:** the user names *models* they want to run. You derive
  the GPU (and therefore Performance-tier) requirement from VRAM math, then size the rest.

Pick the path from how the user talks. Naming models, "serve/host/run/deploy," "watsonx,"
"inference," or "how many GPUs" → **Path B**. Raw infra numbers → **Path A**. If a request
mixes both (e.g. "run these models *and* I need 20 TB usable"), do Path B for GPU/compute
and apply the storage step to their stated storage need.

---

## Core data: GPU count per Performance configuration

LLM workloads always land in the **Performance tier** (the only tier with GPUs). Every GPU
is an **NVIDIA H200 with 141 GB** of VRAM. Use this table to map a required GPU count to a
T-shirt size. (Derived from the calculator — do not re-derive it.)

### Lenovo Performance (H200, 141 GB each)
| Config | GPU servers | GPUs/server | **Total GPUs** | GPU VRAM | Worker vCPUs | Worker mem (GB) | Baseline usable TB |
|---|---|---|---|---|---|---|---|
| P01 | 1 | 2 | **2** | 282 GB | 384 | 2176 | 18.43 |
| P03 | 1 | 2 | **2** | 282 GB | 640 | 4224 | 27.65 |
| P05 | 1 | 4 | **4** | 564 GB | 896 | 6272 | 36.86 |
| P07 | 2 | 4 | **8** | 1128 GB | 1408 | 9472 | 46.08 |
| P09 | 2 | 8 | **16** | 2256 GB | 1664 | 11520 | 55.30 |
| P11 | 4 | 8 | **32** | 4512 GB | 2432 | 15872 | 64.51 |

### Dell Performance (H200 NVL, 141 GB each — XE7745 server is fixed at 8 GPUs)
| Config | GPU servers | GPUs/server | **Total GPUs** | GPU VRAM | Worker vCPUs | Worker mem (GB) | Baseline usable TB |
|---|---|---|---|---|---|---|---|
| DP01 | 1 | 8 | **8** | 1128 GB | 384 | 3584 | 18.43 |
| DP03 | 1 | 8 | **8** | 1128 GB | 640 | 7680 | 27.65 |
| DP05 | 2 | 8 | **16** | 2256 GB | 1152 | 13312 | 36.86 |
| DP07 | 2 | 8 | **16** | 2256 GB | 1408 | 17408 | 46.08 |
| DP09 | 3 | 8 | **24** | 3384 GB | 1920 | 23040 | 55.30 |
| DP11 | 4 | 8 | **32** | 4512 GB | 2432 | 28672 | 64.51 |

**Vendor guidance:** Lenovo can deliver small GPU counts (2 or 4), so for ≤4 GPUs it is
usually cheaper. Dell's XE7745 only comes in 8-GPU blocks, so Dell starts at 8 GPUs. For 8+
GPUs, present both and let the user choose (or follow a stated vendor preference).

---

## Path B — LLM / Watsonx sizing workflow

### Step B1 — Collect the workload
Ask for anything missing (ask all at once):
1. **Which models** do they want to run? (list each one)
2. **Precision** per model — default to **FP16/BF16** if unspecified (watsonx default).
3. **Context window** they need (default 8K if unspecified; matters above ~32K).
4. **Vendor preference** — Lenovo or Dell? (If none, present both where it matters.)

You do **not** need to ask for vCPUs/memory/storage in Path B — you derive compute from the
GPU config and only ask about storage if they have a specific data-volume requirement.

### Step B2 — VRAM per model (IBM's official planning rule)
For **each** model, compute minimum GPU memory using IBM's watsonx.ai guideline:

```
VRAM_min (GB) = (parameters_in_billions × 3) + (context_window_tokens ÷ 100,000)
```

- The **×3** bundles FP16 weights (×2) plus KV cache and runtime overhead (×1). It is the
  safe planning number IBM publishes for watsonx.ai.
- **Parameter counts — always web-verify.** Look up each model in
  `model-catalog.md` for a fast starting value, but the catalog is a starting point,
  not the source of truth: model lineups drift and the list can go stale. Follow the catalog's
  **web verification protocol** — (a) if a model is **not** in the catalog, search the web for
  its parameter count before sizing; (b) if it **is** in the catalog, still do a quick web
  double-check and trust the web if it disagrees, noting the correction to the user. If web
  search isn't available, use the catalog value. Never guess a parameter count; if you can't
  find it anywhere, ask the user.
- **MoE models:** use **total** parameters, not active (all experts sit in VRAM).
- **Quantization:** if the user asks for INT8/INT4, recompute the *weight* portion with
  `model-catalog.md` (params × 1 byte for INT8, × 0.5 for INT4) and add roughly
  the same KV/overhead, i.e. `VRAM ≈ params×(bytes+1) + context÷100,000`. Default to FP16.

### Step B3 — GPUs per model (round to a valid config)
```
GPUs_per_model = round_up(VRAM_min ÷ 141)  → to the nearest valid 1, 2, 4, or 8
```
- Valid per-model counts are **1, 2, 4, or 8** (watsonx constraint).
- **All GPUs for a single model must live on one worker node.** So one model can use at most
  8 H200s on a single node. A model that needs more than 8 GPUs (e.g. Llama 405B, very large
  MoE) requires multi-node sharding. Note this plainly and suggest, in one line, that
  confirming sharded serving with a Fusion specialist is a good next step.

### Step B4 — Aggregate and map to a config
1. **Total GPUs = sum of GPUs_per_model** across all models. (Each served model gets its own
   dedicated GPUs by default. If the user wants to pack several small models onto shared GPUs
   via MIG, mention it as an option but don't assume it.)
2. Pick the **smallest Performance config whose Total GPUs ≥ your total** from the table above.
3. **Sanity-check compute:** confirm that config's worker vCPUs/memory comfortably exceed what
   the models + watsonx control plane need. The GPU-matched config is almost always generous
   on vCPU/memory; if not, step up to the next size.

### Step B5 — Storage
Two storage demands:
- **Model artifacts on disk** (safetensors): ≈ `params_in_billions × 2 GB` per model at FP16
  (×1 for INT8, ×0.5 for INT4). Sum across models.
- **Data / vector DB / RAG / tuning data:** whatever the user states.

Compare the total to the matched config's **baseline usable TB**. If it fits, you're done.
If the storage need exceeds baseline, **do not jump to a bigger compute config** — follow the
**Storage Expansion Path** (see Path A, Step 5): keep the GPU/compute-matched config and tell
the user to have their Fusion specialist add drives.

### Step B6 — Price and present
Return the **Total Net Price** from the matched config in `tshirt-calculator.md`
(see Path A, Step 4 for which field). Then present using the **Output format** below.

---

## Path A — Traditional infra sizing workflow

### Step 1 — Collect requirements
If missing, ask all at once: **vCPUs**, **worker memory (GB)**, **usable storage (TB)**,
**GPU/AI workloads? (yes = Performance, no = Economic)**, **vendor (Lenovo or Dell)**. Don't
price until you have all four, or the user says "just give me the small/a few configs."

### Step 2 — Pick the tier
GPUs needed → **Performance**; no GPUs → **Economic**. Both vendors offer both tiers. If
Performance with no vendor preference, present both Lenovo and Dell.

### Step 3 — Match on compute first
Find the smallest config (E01/E03/…, P01/P03/…) meeting or exceeding **vCPUs**, **memory**,
and **GPUs**. Storage is handled next — never undershoot vCPUs/memory/GPUs.

### Step 4 — Return Total Net Price
Every config is priced with a baseline of **2 drives/node**. If the matched config's baseline
usable storage already covers the requirement, return its **Total Net Price** (bottom row of
the config in the calculator) as the final, all-inclusive price. Use no other price field.

### Step 5 — Storage Expansion Path
If the storage requirement **exceeds** the compute-matched config's baseline:
1. Name the config that satisfies vCPU/memory/GPU (e.g. "P03").
2. Tell the user it meets compute but its baseline storage is below their need.
3. Explain that **more drives can be added without changing the node/compute footprint**.
4. Suggest that a good next step is to have a Fusion specialist add drives and produce the
   final price.
5. Don't estimate the expanded price yourself. You may share the baseline Total Net Price as a
   starting reference, clearly labeled as **excluding the added drives**.

Storage shortfalls **never** trigger an "exceeds largest config" message — only a shortfall on
vCPUs, memory, or GPUs does (then tell them to contact their IBM rep).

---

## Output format

Write like an engineer briefing a colleague, not a vendor pitching a deal. Lead with the
answer, keep it tight, and let the numbers speak.

**Tone and formatting rules:**
- **No emojis.** None — not in headers, bullets, or price lines.
- **Don't narrate your work.** Skip "let me check," "first I'll," and step-by-step internal
  reasoning. Show the final sizing math once, compactly; don't replay every intermediate value.
- **No filler sections.** Don't add "Next Steps," "Pricing Notes," or similar headed sections
  unless they carry information the user actually needs. Fold any genuinely useful note into a
  single short line.
- **Don't oversell.** State the options and their numbers. A one-line neutral comparison
  (e.g. which is cheaper) is fine; avoid "recommended," "best value," and marketing adjectives.

**Include, concisely:**
- The matched **config(s)** — vendor, tier, label (e.g. "Lenovo Performance P05").
- A compact **sizing line**: model(s), params, total VRAM, total GPUs → config.
- Key specs: **GPUs, vCPUs, memory, baseline usable storage**.
- **Total Net Price** (all-inclusive per BPv4.6), one line.
- Assumptions made (precision, context) in one short line so the user can correct them.

If a specialist hand-off is genuinely warranted (multi-node sharding, storage expansion),
phrase it as a suggestion in one line — e.g. "A good next step is to confirm sharded serving
with your Fusion specialist." Never phrase it as a mandatory gate ("you must... before
proceeding").

### Path B example (shape and length to aim for, not a quote)
> **Llama 3.3 70B + Granite 3.3 8B** — FP16, 8K context.
> Sizing: 70B → 2 H200, 8B → 1 H200 = **3 GPUs total**.
>
> **Lenovo Performance P05** (smallest config with ≥3 GPUs): 4× H200, 896 vCPU,
> 6272 GB memory, 36.86 TB usable. Model artifacts on disk ~156 GB.
> **Total Net Price: $3,881,841.45** (all-inclusive, BPv4.6).

---

## Sizing best practices & gotchas (reference)

- **Inference is memory-bound, not compute-bound.** VRAM capacity and memory bandwidth, not
  FLOPS, decide whether a model fits and how many concurrent requests it serves. Size on VRAM.
- **KV cache is the hidden cost.** It grows linearly with context length × concurrent
  requests and can exceed the model weights at long context / high batch. IBM's ×3 rule covers
  typical context; for very long context (128K+) or high concurrency, lean toward the next GPU
  count up, or compute KV cache explicitly:
  `KV ≈ 2 × layers × kv_heads × head_dim × 2 bytes × context × batch`.
- **MoE: size on total params, run on active.** Mixtral 8x7B needs ~47B of VRAM though it
  computes like ~13B. Don't undersize MoE.
- **One model's GPUs stay on one node** (watsonx). Per-model max is 8 H200 on a single node;
  beyond that needs multi-node tensor/pipeline parallelism — flag it.
- **Fine-tuning ≠ inference.** Tuning uses 3–4× the inference VRAM. If the user mentions
  tuning/training, size for that headroom or recommend a larger GPU config.
- **H200 supports FP8, not FP4.** FP4 needs Blackwell. Don't promise FP4 on these configs.
- **Quantization trades accuracy for memory.** INT8 roughly halves weight VRAM vs FP16, INT4
  roughly quarters it — useful to fit a big model on fewer GPUs, but confirm the user accepts
  the quality tradeoff and that watsonx supports the quantized format for that model.
- **Fusion HCI is the validated watsonx appliance** — it ships OpenShift + Fusion Data
  Foundation pre-configured for watsonx/Cloud Pak for Data, so the non-GPU compute in each
  Performance config is already provisioned for the control plane and data services.

---

## Reference files
- `tshirt-calculator.md` — full IBM BPv4.6 pricing tables (Lenovo/Dell, Economic/
  Performance). Read it to pull the **Total Net Price** of the matched config.
- `model-catalog.md` — parameter counts for common foundation models (Granite,
  Llama, DeepSeek, Qwen, Mistral, and other providers), MoE notes, the precision table, and
  the **web verification protocol**. Read it in Step B2.
