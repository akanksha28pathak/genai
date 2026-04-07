# 🧠 The VRAM Hierarchy Guide: Model Selection & Quantization Strategy

> A technical extension to [Native vs. Quantized Models](April_7th_2025_Native_Vs_Quantized_Models.md), focusing on hardware-specific scaling (16GB to 128GB+) and the "Powers of 2" model framework.

---

## 1. The Scaling Law: Precision vs. Parameter Count
The core trade-off in Local LLMs is between **Model Intelligence** (Parameter Count) and **Weight Sharpness** (Precision/Quantization). 

Assuming "Perfect Quantization" (where weights occupy exactly their bit-width), we use a hypothetical "Powers of 2" framework to determine what fits on your hardware:

| Model Size | FP16 (Native) | Int8 (8-bit) | Q4 (4-bit) |
| :--- | :--- | :--- | :--- |
| **8B** ($2^3$) | 16 GB | 8 GB | 4 GB |
| **16B** ($2^4$) | 32 GB | 16 GB | 8 GB |
| **32B** ($2^5$) | 64 GB | 32 GB | 16 GB |
| **64B** ($2^6$) | 128 GB | 64 GB | 32 GB |
| **128B** ($2^7$) | 256 GB | 128 GB | 64 GB |
| **256B** ($2^8$) | 512 GB | 256 GB | 128 GB |

### The "First Guess" Recommendation Matrix
Based on your total available VRAM, here is the optimal model selection:

| VRAM Tier | Best Native (FP16) Fit | Best Quantized (4-bit) Fit | Capability Level |
| :--- | :--- | :--- | :--- |
| **16 GB** | 8B Model | 32B Model | Entry-level / High-speed Coding |
| **24-32 GB** | 16B Model | 64B Model | Prosumer / High Reasoning |
| **48-96 GB** | 32B Model | 128B Model | Research / Complex Logic |
| **124 GB+** | 64B Model | 256B Model | Frontier / Datacenter Grade |

---

## 2. Quantization Deep-Dive: Q4_K_M vs. Q5_K_S
When fitting a model like a **20B** into **16GB**, the specific quantization method (K-Quants) matters. These methods use variable bit-rates to protect critical layers.

* **Q4_K_M (Medium 4-bit):** The "Gold Standard." It uses 6-bit for critical attention layers and 4-bit elsewhere. It preserves ~99% of native intelligence while reducing size by ~70%.
* **Q5_K_S (Small 5-bit):** The "Premium" choice. Offers noticeably better nuance and reduced hallucinations over 4-bit, but requires ~15-20% more VRAM.

**Fitting Tip for 16GB:** To fit a 20B model in 16GB, you must use **Q4_K_M** and enable **4-bit KV Cache** to ensure there is enough room for the conversation history (context window).

---

## 3. Dimension Comparison: When to choose what?

| Goal | Priority | Recommendation |
| :--- | :--- | :--- |
| **Deep Reasoning** | Logic / IQ | **Largest Model @ 4-bit.** A 64B model at Q4 will almost always out-think an 8B model at FP16. |
| **Maximum Speed** | Latency | **Smallest Model @ FP16.** Native models avoid the computational overhead of "de-quantizing" weights on the fly. |
| **Creative Writing** | Nuance / Tone | **8-bit (Int8).** Preserves subtle linguistic patterns better than 4-bit without the massive footprint of FP16. |
| **Large Documents** | Context Size | **Weights < 60% VRAM.** Ensure the model weights leave at least 40% of VRAM free for the KV Cache (the "working memory"). |

---

## 4. Key Takeaways for Hardware Optimization

1.  **The "Intelligence Jump":** Moving from 16GB Native to 16GB Quantized allows you to jump from an **8B** model to a **32B** model. This is a massive leap in reasoning capability for the same hardware cost.
2.  **Fine-Tuning:** Native FP16 is required for traditional training. For Quantized models, use **QLoRA** to add high-precision "adapter" layers.
3.  **The Context Trap:** Don't fill your VRAM 100% with model weights. A model that "fits" but has no room for KV Cache will crash as soon as you start a conversation. Always aim for at least 2GB-4GB of "headroom."