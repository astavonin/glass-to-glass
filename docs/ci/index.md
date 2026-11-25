# CI

A video pipeline that nobody tests on real hardware decays without anyone noticing. This section is how the pipeline stays honest.

Embedded CI is usually bad, and the excuses are always the same. Cross-compiling is hard, so we build on the device. On-device tests cannot be automated, because the board is special and the flashing is manual. Neither holds up.

The target: a build on a laptop and a build in CI come out of the same pinned toolchain, through the same scripts, with the same flags — and CI does the full verification, including tests on real boards, with nobody sitting next to the hardware.

## Pages

**[Integration Testing for Linux Video Pipelines](integration-testing.md)** — the kernel boundary is where failures cluster, and it is the part neither unit tests nor hardware-in-the-loop exercises well. `vivid` gives you a V4L2 device with known frame content, so a capture test can assert on pixels with no camera attached.

## What is coming

**The pipeline, implemented.** The design is published; what is missing is a public implementation of it. The interesting part is how much of the machinery disappears when the build tool already does the work:

| Hand-built | With Bazel |
|---|---|
| an SDK image pinned by version, with three readers | no SDK image — toolchain and sysroot fetched and checksummed by the build |
| a dispatcher script wrapping Docker | `bazel test --config=pi`, with `--config=ci` adding network isolation and strict deps |
| a test selector plus audits to keep it honest | `bazel query` with `rdeps`, enforced by the build graph |
| a vendor debug channel for board control | ssh, plus UART for everything before the network exists |

**UART in the loop.** How do you get a failure trace off a board that reboots before it can write a log? The vendor image's volatile log eats the trace; disabling it is the first step, and the serial console is the only channel alive early enough to matter. Motivated by a board that resets on four of ten capture runs, with five identical panic traces captured so far.

One honest limit, since the published design assumes otherwise: this bench has two boards, but a Pi 5 and a CIX board are two *targets*, not two interchangeable slots. Per-board locking and pool parallelism need identical hardware, so those claims stay untested here.

## Related

Published on sysdev.me: [CI for Embedded Projects](https://sysdev.me/2026/11/12/ci-for-embedded-projects/) for the design — five rules, six stages, local/CI parity, board in the loop, nightly — and [Cross-Compiling C++ with Bazel](https://sysdev.me/2026/10/19/bazel-cross-compilation/) for the toolchain, with [`pi-bazel`](https://github.com/astavonin/pi-bazel) as its companion repo.
