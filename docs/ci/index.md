# CI

A video pipeline that nobody tests on real hardware decays without anyone noticing. This section is how the pipeline stays honest.

Embedded CI is usually bad, and the excuses are always the same. Cross-compiling is hard, so we build on the device. On-device tests cannot be automated, because the board is special and the flashing is manual. Neither holds up.

The target: a build on a laptop and a build in CI come out of the same pinned toolchain, through the same scripts, with the same flags — and CI does the full verification, including tests on real boards, with nobody sitting next to the hardware.

## Pages

**[Integration Testing for Linux Video Pipelines](integration-testing.md)** — the kernel boundary is where failures cluster, and it is the part neither unit tests nor hardware-in-the-loop exercises well. `vivid` gives you a V4L2 device with known frame content, so a capture test can assert on pixels with no camera attached.

## Related

Published on sysdev.me: [CI for Embedded Projects](https://sysdev.me/2026/11/12/ci-for-embedded-projects/) for the design — five rules, six stages, local/CI parity, board in the loop, nightly — and [Cross-Compiling C++ with Bazel](https://sysdev.me/2026/10/19/bazel-cross-compilation/) for the toolchain, with [`pi-bazel`](https://github.com/astavonin/pi-bazel) as its companion repo.
