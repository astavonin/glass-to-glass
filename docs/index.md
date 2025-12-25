# A practical reference for building glass-to-glass video pipelines on constrained hardware

This site exists to explain the code, not to replace it. Right now, the code lives in a single public repository:

* [https://github.com/astavonin/pi-cam-capture](https://github.com/astavonin/pi-cam-capture)

**This repository is the source of truth.**

The glass-to-glass video pipeline is built incrementally. What exists today is a working foundation: concrete hardware choices, an initial capture path, early transport experiments, and basic observability. Many parts are incomplete and some decisions will change as the system is exercised under real constraints.

The goal is to document the system while it is being built, not after it has been polished. Design decisions and failure modes are captured as they happen. If the documentation and the code ever diverge, the code wins.

## How to read this site

Each article focuses on a narrow, concrete problem and explains the reasoning behind a specific set of changes.

Every article links to a GitHub diff that shows the exact code changes discussed. The diff is the authoritative reference. The text exists to explain why those changes were made.

!!! example
    Like this! Each article starts with such block.

## Scope

This is not a tutorial series or a reference implementation. It assumes familiarity with Linux, V4L2, and video pipelines, and focuses on operational behavior rather than abstractions.

The site will change as the code changes. Sections will be revised, approaches will be dropped, and assumptions will be challenged.

The intent is to make the system’s evolution visible, not to present a finished design.
