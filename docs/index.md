# Glass-to-Glass

A camera lens at one end, a display at the other, and a frame that has to cross everything in between. This site follows that path on cheap ARM hardware — what each stage actually costs, where it breaks, and which assumptions stop being true once you measure instead of reading the datasheet.

The pipeline is the project. The other two sections exist because it needs them: **CI**, because a pipeline nobody tests on real hardware decays without anyone noticing, and **NPU/GPU**, because the frames eventually have to feed something.

Every number here was measured on a board on my desk, with the date, the kernel and the command recorded. Where something is inferred rather than measured, it says so. Where an earlier version of a page was wrong, the correction stays visible with its date — the refutations tend to be more useful than the claims.

## The boards

| | Raspberry Pi 5 | Orange Pi 6 Plus |
|---|---|---|
| SoC | BCM2712, Cortex-A76 | CIX P1 / CD8180 "Sky1" |
| Camera | IMX477 and IMX708 over CSI-2 | 13 MP MIPI-CSI, plus USB |
| Video encode | **none — there is no encoder on this board** | Linlon V8, 18 × 1080p30 sustained |
| Inference | — | Zhouyi V3 NPU, 30 TOPS |
| Camera stack | upstream kernel drivers, walkable media graph | closed vendor daemon that panics the kernel |

That last row is why there are two boards. One has a clean kernel stack and no encoder; the other has a serious hardware encoder and a stack you cannot inspect. Almost everything worth writing down comes out of the gap between them.

## The sections

**[Video Streaming](video/index.md)** — the pipeline itself. What a frame is before anyone debayers it, how the kernel exposes the capture hardware, what software encode costs when the board has no encoder, and what a real one buys you.

**[CI](ci/index.md)** — how the pipeline stays honest. One pinned toolchain for the laptop and the runner, tests that exercise the kernel boundary with no camera attached, and real boards wired into the pipeline so nobody has to sit next to the hardware.

**[NPU/GPU](npu-gpu/index.md)** — what the frames feed. An NPU is a DMA-fed VLIW cluster behind an SMMU, programmed by a closed cross-compiler. Not an AI chip. Every surprise follows from that.

## How to read it

Pages are independent. Each answers one question and states what it does not cover, so there is no reading order and nothing is a prerequisite for anything else.

This is not a tutorial. It assumes Linux, C++, and some exposure to V4L2 or an equivalent driver API, and it spends its time on measured behaviour rather than on abstractions.

Code lives in separate repositories, linked from the pages that discuss it. The implementation language is C++23, built with Bazel. Earlier work was in Rust; where a page still shows Rust, it says so at the top.
