# CLAUDE.md

Working instructions for agents and contributors on glass-to-glass — the documentation site behind [gtog.dev](https://gtog.dev).

**What belongs here:** things with no other owner — how to reach the hardware, how to run the site, how to engage.

**What does not:** anything `planning/` already owns. Book structure, milestone contents, article scope, appendix facts, and workflow procedure all live in files maintained as work progresses. A copy here goes stale silently and then misleads every session, because this file is loaded into context and never verified. Write a pointer, not a copy.

Machine-specific state (local clone paths, personal preferences) goes in `CLAUDE.local.md`, which is gitignored.

## Context

End-to-end guide to building reliable camera-to-display video systems on constrained hardware. The site and the companion implementation repos are built in parallel — docs evolve as the code evolves, and every article is anchored to real, working code.

Single maintainer. Do not suggest consulting others or hedge about design choices.

## Communication

Terse and direct. No preamble, no summaries, no affirmations.

Skip fundamentals. Engage at the systems or architectural trade-off level — assume working depth in Rust, V4L2/libcamera, embedded Linux, RTP/QUIC/WebRTC, and MkDocs.

## Local Dev

```bash
source ./env.sh   # venv + MkDocs dependencies
mkdocs serve      # preview at http://localhost:8000
```

## Hardware Device

Raspberry Pi 5, kernel `6.12.47+rpt-rpi-2712`. Two cameras attached:

| Video nodes | Sensor | Model | Stable identifier |
|---|---|---|---|
| `/dev/video0`–`7` | IMX477 | HQ Camera, fixed focus | `platform:1f00128000.csi` |
| `/dev/video8`–`15` | IMX708 | Camera Module 3, VCM autofocus (dw9807) | `platform:1f00110000.csi` |

Four things that change how code gets written. Everything else about this board — ISP topology, formats, strides, node roles — belongs to the appendix docs, not here.

- **Never hardcode `/dev/mediaN`.** The numbers are boot artifacts: measured 2026-08-12, every media node moved across a plain reboot with no kernel or hardware change, while every video node held. Resolve by bus info:
  ```bash
  for m in /dev/media*; do echo -n "$m "; media-ctl -d "$m" -p | grep '^bus info'; done
  ```
- **`rp1-cfe` nodes report `I/O MC`.** `open` + `VIDIOC_S_FMT` is not enough — `S_FMT` succeeds and `STREAMON` is what fails. `media-ctl` pipeline setup must come first, so hardware integration tests cannot call `CaptureSession::new(device_index)` directly.
- **`rp1-cfe` capture is single-plane** (`/dev/video0` caps `0x24a00001`). `pispbe` is the multiplanar one (`0x04201000`) and carries no `I/O MC`. Do not assume MPLANE for "the Pi camera".
- **There is no hardware video encoder on this board.** Any claim of a "Pi 5 V4L2 M2M encoder" is false. Encoding here is software or off-board. This fabrication has reached a published article once — treat it as a known trap.

Verified hardware facts live in `planning/book/appendix/issues/A3-linux-camera-stack/spec.md` §2/§5, with the command and captured output behind each one. Known-bad claims in the research notes are tracked in `planning/book/appendix/status.md`.

## Companion Repos

Read by `/spec` to resolve the implementation repo for a main article. **Must stay in sync with `planning/book/overview.md` → Companion Repos, which is the source of truth.** Local clone paths are in `CLAUDE.local.md`.

| Repo | Parts |
|---|---|
| [`pi-cam-capture`](https://github.com/astavonin/pi-cam-capture) | Part 1 (Rust) |
| `pi-cam-capture-zig` | Part 1 article 11 (Zig) |
| `pi-video-pipeline` | Parts 2–3 (Rust) |
| `pi-rtp-zig` | Part 3 article 06 (Zig) |
| `pi-ingest` | Part 4 (Rust) |
| `pi-control` | Part 5 (Zig) |
| `pi-webrtc-streamer` | Part 6 (Rust) |
| `pi-ai-lite` | Part 8 (Rust) |
| `fleet-cam-reference-design` | Part 10 (capstone) |

Parts 7 and 9 are cross-cutting and have no repo of their own.

For code examples and diff links, `pi-cam-capture` is the current target.

## Where Everything Else Lives

`planning/` is not published to the site and is not in the reading flow; it is the working state behind it.

| Topic | Owner |
|---|---|
| Book structure, parts, milestone map | `planning/book/overview.md` |
| Article list, scope, and phase per part | `planning/book/milestone-XX-*/status.md` |
| Appendix pages, blocks, known defects | `planning/book/appendix/status.md` |
| Active work and next steps | `planning/progress.md` |
| Cross-article open items | `planning/book/todos.md` |
| Prose and style rules | `planning/style-guide.md` |

**On the workflow:** articles come in two page types — `main` (anchored to a companion repo) and `appendix` (a reference page whose facts come from hardware, kernel docs, and specifications). They take different paths through `/spec` → `/write` → `/review-article`, and the command definitions are authoritative for both. In particular, `/write` in book-article mode emits `brief.md`, not `draft.md` — `draft.md` is reserved for the article produced from the brief.
