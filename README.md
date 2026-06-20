# RCCS-CloudAgent

Claude Code and Codex plugin for the RIKEN R-CCS Cloud supercomputer.

Provides two MCP servers:
- **rccs-cloud-hpc** — submit and monitor Slurm jobs, manage files on the cluster
- **rccs-cloud-docs** — search the built-in R-CCS Cloud guide

## What this plugin does

- Submit jobs to any of the ~20 partitions spanning CPU-only, NVIDIA GPU, AMD GPU, and Intel GPU hardware
- Monitor job status, read output files, and triage failures
- Manage files on the cluster (upload, download, copy, compress, etc.)
- Search documentation for partition specs, module commands, and conventions

## Quick start

1. Install the plugin in Claude Code or Codex (see marketplace instructions)
2. Run `/rccs-cloud-configuring` to set up SSH access
3. Run `/rccs-cloud-demo` for an end-to-end walkthrough

## Key concepts

**Partition selection is everything.** The R-CCS Cloud exposes many hardware
families through distinct Slurm partitions. Each partition requires its own
`system/<partition>` module loaded first — loading the wrong one produces wrong
results. See `/rccs-cloud-reference` for the quick-reference table.

**The default partition is `genoa`** (AMD EPYC x86_64, 16 nodes). For GPU work
you must explicitly choose the GPU partition that matches your codebase:
- CUDA/nvhpc → `a100`, `ai-l40s`, `qc-a100`, `qc-gh200`, `ng-dgx-m[0-3]`
- ROCm → `mi100`, `qc-mi250`, `fs-mi300a`, `fs-mi300x`
- oneAPI/Intel → `qc-pvc`
- Fujitsu A64FX → `fx700`

## SSH configuration

The plugin connects to `login.cloud.r-ccs.riken.jp`. Add an alias to `~/.ssh/config`:

```
Host rccs-cloud
    HostName login.cloud.r-ccs.riken.jp
    User <your-username>
    IdentityFile ~/.ssh/id_rsa
```

Then set `"ssh": {"host": "rccs-cloud"}` in `~/.rccs-cloud/config.json`.

## For developers

See `AGENTS.md` for architecture details, design rules, and the development workflow.
See `PORTING.md` for instructions on porting this plugin to a new machine.
