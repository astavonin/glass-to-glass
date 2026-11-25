# Video Streaming

Sensor to display, on two boards that disagree about how to do it.

The Raspberry Pi 5 has a clean upstream kernel stack, a walkable media graph, a hardware ISP reachable through libcamera — and **no video encoder at all**. The Orange Pi 6 Plus has a Linlon V8 that sustains eighteen 1080p30 encode sessions, and a closed vendor daemon that resets the board on four of ten MIPI capture runs. Each board does something the other cannot, and that asymmetry decides how the code gets written.

## Pages

**[Camera Sensor Fundamentals](camera-sensor-fundamentals.md)** — photodiodes, Bayer colour filter arrays, bit depth, MIPI packed raw layouts, rolling shutter, sensor modes and HDR. Why a raw frame off `/dev/video0` decodes to green garbage, and what has to happen before it is an image.

**[The Linux Camera Stack](linux-camera-stack.md)** — V4L2 device nodes and capability bits, the Media Controller graph, stride and buffer ownership, and the block topology of the Pi 5 capture path. Measured on one board and one kernel, with the versions listed; the shapes generalise and the node numbers do not.

## Related

- [Two ISP Architectures: Pi 5 and Orange Pi 6 Plus](../npu-gpu/isp-architectures.md) — the two capture stacks compared side by side: the Pi 5's kernel-described hardware against the CIX vendor daemon and its single private ioctl. The reason one board is boring to capture from and the other resets.
- [Accelerators](../npu-gpu/index.md) — the ISP, encoder, NPU and GPU every frame crosses on its way through.
- [CI](../ci/index.md) — how any of this gets tested without a camera attached.
- [Cross-Compiling C++ with Bazel: Platforms, Toolchains and the Exec/Target Split](https://sysdev.me/2026/10/19/bazel-cross-compilation/) — how the C++23 code here is built for the board, with [`pi-bazel`](https://github.com/astavonin/pi-bazel) as its companion repo.
