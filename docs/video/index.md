# Video Streaming

Sensor to display, on two boards that disagree about how to do it.

The Raspberry Pi 5 has a clean upstream kernel stack, a walkable media graph, a hardware ISP reachable through libcamera — and **no video encoder at all**. The Orange Pi 6 Plus has a Linlon V8 that sustains eighteen 1080p30 encode sessions, and a closed vendor daemon that resets the board on four of ten MIPI capture runs. Each board does something the other cannot, and that asymmetry decides how the code gets written.

## Pages

**[Camera Sensor Fundamentals](camera-sensor-fundamentals.md)** — photodiodes, Bayer colour filter arrays, bit depth, MIPI packed raw layouts, rolling shutter, sensor modes and HDR. Why a raw frame off `/dev/video0` decodes to green garbage, and what has to happen before it is an image.

**[The Linux Camera Stack](linux-camera-stack.md)** — V4L2 device nodes and capability bits, the Media Controller graph, stride and buffer ownership, and the block topology of the Pi 5 capture path. Measured on one board and one kernel, with the versions listed; the shapes generalise and the node numbers do not.

## What is coming

Measured, written up in the repos, not yet pages here:

- **What software encode actually costs on a Pi 5.** Two CSI cameras at 1080p30 cost 29% CPU because frames arrive from the ISP by DMA; adding one USB camera at 1080p30 costs another 53 points, because its MJPEG has to be decoded on the CPU and this board has no JPEG decoder either. Thermal throttling turns out not to be reachable from CPU load at all — saturation binds first, and the throttle point had to be provoked by switching the fan off.
- **Eighteen streams on one VPU, and why decode serialises.** Encode scales far past the four advertised hardware session slots. Decode does not scale at all, and starting an encode while a decode is streaming wedges the decode permanently, in either order.

Then capture in C++23, the RTSP and SRT transport slice, and end-to-end latency attributed per stage.

## Related

The two ISP architectures compared side by side — the Pi 5's kernel-described hardware against the CIX vendor daemon and its single private ioctl — is published as [MIPI Cameras on Raspberry Pi 5 and Orange Pi 6 Plus](https://sysdev.me/2026/11/23/mipi-isp-architectures/).
