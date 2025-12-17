# A practical reference for building glass-to-glass video pipelines on constrained hardware

This site exists to complement the code, not replace it. Today, that code is a single public repository, pi-cam-capture, which serves as the starting point of the glass-to-glass pipeline:

- [github.com/astavonin/pi-cam-capture](https://github.com/astavonin/pi-cam-capture)

The Glass-to-Glass Video Pipeline is at an early stage. What exists today is a working foundation: hardware choices, initial capture paths, transport experiments, and the first observability hooks. Many parts are incomplete, some will be reworked, and assumptions will be challenged as the system evolves.

That is intentional.

The goal is to document the system as it is built, not after it is “finished.” Design decisions, trade-offs, and failure modes are captured while they are still fresh and debatable, not retroactively cleaned up. The code remains the source of truth. This site provides context around it: why something exists, what problem it tries to solve, and where it currently falls short.

As the pipeline matures, this documentation will change with it. Sections will be rewritten. Approaches will be dropped. Measurements will replace guesses. If something here diverges from the code, the code wins.
