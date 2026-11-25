# NPU/GPU

Frames arrive from the capture path and eventually have to feed something. This section is what the silicon on the other end will and will not do.

An NPU is a DMA-fed VLIW cluster behind an SMMU, programmed by a closed cross-compiler. Not "an AI chip". Every surprise in this section follows from that one sentence.

If you have brought up an undocumented peripheral before — descriptor chains, an ioctl ABI that shifts when a struct grows, a firmware blob with no documentation, a fault that presents as a hang rather than a trap — then you already have the right instincts for this hardware. What you may not have is the vocabulary, so every ML term here arrives with its embedded equivalent beside it.

## The hardware

Measured on an Orange Pi 6 Plus, CIX P1 / CD8180, vendor image on kernel `6.6.89-cix`.

| | |
|---|---|
| NPU | Zhouyi V3 — 1 cluster, 3 cores, 4 TECs, 30 TOPS. KMD v5.11.0 |
| Address space | four 1 GiB windows per ASID, carved by the driver; no per-process MMU, no demand paging, no preemption |
| GPU | Immortalis-G720 MC10, 10 cores. Vulkan 1.3.296, OpenGL ES, 5 render nodes |
| Vision inference | MobileNet v2 at **1.7–2.7 ms**, with the CPU idle |
| LLM inference | Gemma 4 E2B, 2.51 GB zero-copy: prefill 47.3 tok/s, decode 15.38 tok/s, 2.35 tokens a pass, 52% of speculative drafts accepted |
| Toolchain | closed cross-compiler with a quantizer bolted on; op coverage is a kernel-library question, not a mathematical one |

The economics matter more than the peak number: this NPU's win is prefill and leaving twelve CPU cores free, not raw decode throughput. Decode is bandwidth-bound on the same LPDDR either way.

## What is coming

Five pages, written in dependency order, from a bring-up completed 2026-11-24. The measurements exist; the writing does not.

1. **The hardware contract** — what this device is, in terms an embedded engineer already has.
2. **How a model becomes a binary** — the compiler stages, and why a layer that will not compile is usually a missing kernel rather than unsupported maths.
3. **Four edits to make a real model compile** — a production vision model ported, with the honest note that accuracy was never validated because calibration was noise.
4. **Why LLMs break the graph model** — a growing KV cache is a realloc in a system with no allocator, and streaming 2.4 GB of weights through a 4 GB window is bank switching.
5. **Bringing up an accelerator with no visibility** — the method, when the failure mode is silence. This is the one that outlasts the SDK: the specific bugs age out, the bisection discipline does not.

**Status:** publication is gated on a licensing question about the vendor SDK, not on the writing. Nothing goes up until that is settled.

The GPU has no page scheduled. Vulkan and OpenGL ES work and the render nodes are there, but nothing has been asked of it yet — no measurement, no failure, no surprise, so no article. The likely first question is whether a compute shader earns its place ahead of encode, which would connect this section to Video Streaming.
