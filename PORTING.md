# Porting guide

This repo follows [hpc-agent-core's `PORTING.md`](https://github.com/william-dawson/hpc-agent-core/blob/main/PORTING.md)
for the general process (mental model, the no-write-access-to-core and
clarity-over-cleverness rules, machine-facts checklist, repo layout,
`config.py`/`compute.py` wiring, validation, and the standing invariants).

It is **not copied here** — a copy is a second place for it to go stale in,
which is exactly the mistake this guide itself warns against. Read the
canonical version at the link above and follow it.

This machine is the **R-CCS Cloud**, a RIKEN R-CCS heterogeneous Slurm
cluster: ~20 partitions spanning several CPU architectures and multiple GPU
vendors (NVIDIA / AMD / Intel), with partition-specific modules. The
official documentation you should build from is in [`docs/`](docs/). Pay
close attention to the GPU-request dialect and module story — they are the
parts most likely to differ from a simpler single-partition machine. Once
you have implemented the port, record everything specific to *this* machine
— its cluster facts, any decisions you made under uncertainty, and the repo
map — in an `AGENTS.md` you create, the way the canonical guide describes.
