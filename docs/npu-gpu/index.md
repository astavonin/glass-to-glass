# Accelerators

Four pieces of fixed-function silicon sit on one of these boards — an image signal processor, a video codec, a neural accelerator and a GPU — and three of the four are reached only through closed vendor software. Every frame crosses at least one of them.

The pattern repeats: the documentation describes an interface, the driver implements something adjacent to it, and the measurement contradicts both.

## Pages

**[Two ISP Architectures: Pi 5 and Orange Pi 6 Plus](isp-architectures.md)** — the same MIPI camera on two boards: ten captures out of ten on the Pi 5, six out of ten on the Orange Pi with every failure a board reset, and ten out of ten from a USB camera on that same board. The difference is not the silicon. It is that one kernel describes its hardware and the other hands you a private ioctl and a closed daemon that already owns the capture node.

## The silicon

Measured on an Orange Pi 6 Plus, CIX P1 / CD8180 "Sky1", vendor image on kernel `6.6.89-cix`.

| | What it is | State |
|---|---|---|
| **ISP** | closed vendor daemon owning the capture device, one private ioctl on the sensor subdev, entities registered with zero pads and zero links | measured; compared against the Pi 5's kernel-described stack |
| **VPU** | Linlon V8 — separate encode and decode engines, four advertised hardware session slots | measured: **18 × 1080p30 encode** sustained, ~537 fps ceiling at 1080p; decode runs **one session only** |
| **NPU** | Zhouyi V3 — 1 cluster, 3 cores, 4 TECs, 30 TOPS, KMD v5.11.0. Four 1 GiB address windows per ASID, carved by the driver; no per-process MMU, no demand paging, no preemption | measured: MobileNet v2 at **1.7–2.7 ms** with the CPU idle; Gemma 4 E2B at 2.51 GB zero-copy, prefill 47.3 tok/s, decode 15.38 tok/s, 52% of speculative drafts accepted |
| **GPU** | Immortalis-G720 MC10, 10 cores. Vulkan 1.3.296, OpenGL ES, 5 render nodes | present and working; nothing asked of it yet |

Two results change how you design around this hardware. Starting an encode while a decode is streaming **wedges the decode permanently**, in either start order — so the encoder's generosity and the decoder's rigidity cannot be combined. And the NPU's real win is prefill plus leaving twelve CPU cores free, not raw decode throughput, because decode is bandwidth-bound on the same LPDDR regardless of what runs it.

## Related

- [Video Streaming](../video/index.md) — the pipeline these accelerators sit inside, and the capture path that feeds them.
- [CI](../ci/index.md) — how work on a board that panics gets tested without a person watching it.
