# CLAUDE.md

Working instructions for agents and contributors on glass-to-glass — the documentation site behind [gtog.dev](https://gtog.dev).

**What belongs here:** things with no other owner — how to reach the hardware, how to run the site, how to engage.

**What does not:** anything `planning/` already owns. Stream scope, article lists, per-article evidence, and workflow procedure all live in files maintained as work progresses. A copy here goes stale silently and then misleads every session, because this file is loaded into context and never verified. Write a pointer, not a copy.

Machine-specific state (local clone paths, personal preferences) goes in `CLAUDE.local.md`, which is gitignored.

## Context

What cheap ARM hardware actually does when you measure it, on a Raspberry Pi 5 and an Orange Pi 6 Plus. Three sections, with video as the spine: **Video Streaming** (the sensor-to-display pipeline), **CI** (how it stays tested on real boards), and **Accelerators** (the ISP, video codec, NPU and GPU every frame crosses or feeds).

Every published claim is measured on a board here, with the date, the kernel and the command recorded. Corrections stay visible with their date rather than being silently edited.

**This was a book until 2026-11-25 and is not one any more.** Ten parts, nine companion repos, and a `/spec` → `/review-spec` → `/write` → `brief.md` → `draft.md` chain per article. It did ship pages — A3 on 2026-10-01 was the last — but the cost per page kept rising: A2's spec was approved in August and still had no draft in November, and A9's ran to 227 KB across two non-converging review rounds for an article that then shipped in two weeks by skipping the chain entirely. The unit is now a stream's `status.md`, which doubles as its fact contract. Do not reintroduce the book machinery.

Single maintainer. Do not suggest consulting others or hedge about design choices.

## Communication

Terse and direct. No preamble, no summaries, no affirmations.

Skip fundamentals. Engage at the systems or architectural trade-off level — assume working depth in C++23, Bazel, V4L2/libcamera, embedded Linux, RTP/SRT/WebRTC, and MkDocs. Earlier work was Rust; the language argument is settled and published, so do not re-open it.

## Local Dev

```bash
source ./env.sh   # venv + MkDocs dependencies
mkdocs serve      # preview at http://localhost:8000
mkdocs build --strict   # what CI runs; fails on a broken internal link
```

**Restart `mkdocs serve` after editing.** Its watcher does not reliably pick up changes here — `mkdocs.yml` edits never do, and content edits were observed not to either. A preview that looks unchanged is usually a stale server, not a failed edit.

Pushing to `main` deploys: `.github/workflows/publish-latest.yml` runs `mkdocs build --strict` then `mkdocs gh-deploy --force`. A plugin added to `mkdocs.yml` must also be added to that workflow's `pip install` line, or the deploy fails.

## Hardware Devices

Two boards. The Pi 5 is the one most of the capture notes below are about; the Orange Pi 6 Plus has its own section after it.

### Raspberry Pi 5

Kernel `6.12.47+rpt-rpi-2712`. Two cameras attached:

| Sensor | Model | Stable identifier | Video nodes, no USB video at boot |
|---|---|---|---|
| IMX477 | HQ Camera, fixed focus | `platform:1f00128000.csi` | `/dev/video0`–`7` |
| IMX708 | Camera Module 3, VCM autofocus (dw9807) | `platform:1f00110000.csi` | `/dev/video8`–`15` |

**The bus address is the identifier. The node range is not** — it is an observation about one driver-binding order, and the last column holds only while no USB video device is present at power-up. Measured 2026-09-30 with a UVC webcam attached at boot: it enumerated ahead of the platform drivers, took `/dev/video0`–`1`, and both CFE blocks shifted up by two to `/dev/video2`–`9` and `/dev/video10`–`17`. Six media nodes existed on that boot rather than five.

Five things that change how code gets written. Everything else about this board — ISP topology, formats, strides, node roles — belongs in the published pages, not here.

- **Never hardcode `/dev/mediaN`, and do not hardcode `/dev/videoN` either.** Media numbers are boot artifacts: measured 2026-08-12, every media node moved across a plain reboot with no kernel or hardware change. Video numbers held across four such boots, then moved on the fifth when a USB camera was present at power-up. Resolve by bus info, then take the video nodes from the graph:
  ```bash
  for m in /dev/media*; do echo -n "$m "; media-ctl -d "$m" -p | grep '^bus info'; done
  media-ctl -d "$M" -p | grep -E '^- entity|device node name'   # entity -> /dev/videoN
  ```
- **Setting every pad format is not enough to stream.** A video node holds a format of its own that no pad write propagates into, and `cfe_video_link_validate()` checks it against the remote pad at `STREAMON`: a mismatch is `EINVAL` with `Wrong width or height` in `dmesg`. The bypass path needs four steps, not three — enable the link, set the sensor pad, set the `csi2` sink pad, then `VIDIOC_S_FMT` on the video node.
- **`rp1-cfe` nodes report `I/O MC`.** `open` + `VIDIOC_S_FMT` is not enough — `S_FMT` succeeds and `STREAMON` is what fails. `media-ctl` pipeline setup must come first, so hardware integration tests cannot call `CaptureSession::new(device_index)` directly.
- **`rp1-cfe` capture is single-plane** (a `csi2_ch0` node reads caps `0x24a00001`). `pispbe` is the multiplanar one (`0x04201000`) and carries no `I/O MC`. Do not assume MPLANE for "the Pi camera". In that word, `0x00200000` is `V4L2_CAP_EXT_PIX_FORMAT`, not `V4L2_CAP_DEVICE_CAPS` — the latter is `0x80000000` and never appears in `device_caps`.
- **There is no hardware video encoder on this board.** Any claim of a "Pi 5 V4L2 M2M encoder" is false. Encoding here is software or off-board. This fabrication has reached a published article once — treat it as a known trap.

Verified hardware facts live in `planning/book/appendix/issues/A3-linux-camera-stack/spec.md` §2/§5, with the command and captured output behind each one. Known-bad claims are tracked in `planning/book/appendix/status.md` → "Known defects in research notes". Both are **book-era files, kept deliberately** — they are the contract behind the published Linux Camera Stack page and the refutation ledger. Legacy location, current content.

### Orange Pi 6 Plus

CIX P1 / CD8180 "Sky1", vendor image on kernel `6.6.89-cix`, reached over ssh. Full notes and measurements are in the [`opi6`](https://github.com/astavonin/opi6) repo — `README.md` for the board, `docs/` for the ISP, VPU and NPU research. The alias is in `CLAUDE.local.md`.

Four things that change how code gets written:

- **MIPI capture panics the kernel.** Measured 2026-11-10: 4 of 10 MIPI capture runs reset the board, against 10 of 10 clean USB runs. It is a use-after-free on the capture node's queue pointer in the vendor driver, and it happens with or without the vendor daemon running. **If it must not fail, use a USB camera** — that path bypasses the vendor ISP entirely.
- **The vendor daemon owns the capture node.** `isp_app` holds `/dev/video9` from boot. There is no media graph to walk (every entity has zero pads and zero links), the node advertises no V4L2 controls, and the only sensor-configuration path is one private ioctl. libcamera cannot drive it.
- **There is a real hardware encoder, and it is generous.** 18 × 1080p30 H.264 sustained on one engine against four advertised session slots. **Decode is the opposite** — one session only, and starting an encode while a decode streams wedges the decode permanently, in either order. MJPEG input needs software `jpegdec`; hardware JPEG decode is the broken side.
- **The vendor BSP is the only stack that drives the accelerators.** Mainline and Armbian have no ISP, no NPU and no codec driver for this SoC. Check `uname -r` before trusting any note that predates a reflash.


## Companion Repos

Code lives outside this repo and is linked from the pages that discuss it. The implementation language is C++23, built with Bazel.

| Repo | Role |
|---|---|
| [`pi-bazel`](https://github.com/astavonin/pi-bazel) | The active implementation. Hermetic Bazel cross-compilation to the Pi 5, the device-test harness, and the CI work. Its `planning/bazel-cross-pi5/overview.md` carries decisions D1–D11 and the measured encode-capacity tables. |
| [`opi6`](https://github.com/astavonin/opi6) | Orange Pi 6 Plus board notes and measurements. `docs/` holds the ISP, VPU and NPU research; `planning/npu-articles/status.md` is the authoritative NPU article plan. |
| [`pi-cam-capture`](https://github.com/astavonin/pi-cam-capture) | **Frozen.** The earlier Rust capture implementation. Still referenced by published Rust posts; not being extended. |

The nine-repo Rust/Zig table that used to be here described the book plan and is gone with it. Parts, article numbers and per-part repos no longer exist.

## Where Everything Else Lives

`planning/` is not published to the site and is not in the reading flow; it is the working state behind it. It is also gitignored, so nothing in it reaches GitHub.

| Topic | Owner |
|---|---|
| The frame: streams, venues, the record convention | `planning/streams/overview.md` |
| Video stream — articles, evidence, open questions | `planning/streams/video/status.md` |
| CI stream | `planning/streams/ci/status.md` |
| Accelerators stream (folder and URL are `npu-gpu`; the label is "Accelerators") | `planning/streams/npu-gpu/status.md` |
| Active work and next steps | `planning/progress.md` |
| Prose and style rules | `planning/style-guide.md` |
| Book-era tree, kept for its fact contracts and the refutation ledger | `planning/book/` — legacy, frozen copy in `planning/archive/` |

**On the workflow:** a stream's `status.md` *is* its fact contract — the per-article "evidence on hand" field is the claim-to-evidence map, and the audience and register paragraphs are the style contract. Nothing else is written before the article. There is no `/spec`, no `/review-spec`, no `brief.md` and no `draft.md`.

The record convention, in full in `streams/overview.md`: result table first; date, board and kernel in the opening line; an explicit "what this does not measure" section; a reproduce command checked into the repo; "what is still open" listed rather than quietly promoted; corrections noted inline with a date.

**Two venues.** `sysdev.me` is the funnel — fast, ungated, where a finding lands first. This site is the destination: composed, measured, cross-linked. A post moves from one to the other rather than being copied to both; when it moves, the sysdev side gets a redirect.
