---
name: rccs-cloud-reproducing
description: Use when the user asks to make a result reproducible, shareable, or turned into a notebook/script — or proactively suggest it after a chunk of exploratory job/file work that produced something worth preserving. Builds a Jupyter notebook that replays the real R-CCS Cloud workflow via hpc_agent_core.client, not a hand-copied SSH script.
user-invocable: true
---

# Turning a session into a reproducible notebook

A chat transcript is not something a co-author (or you, in six months) can
re-run. This builds a linear Jupyter notebook that calls the *same* MCP tool
surface you've been using — via `hpc_agent_core.client` — instead of a
hand-copied `ssh`/`scp` script.

## Do not go looking for a local copy of this plugin's source

You are running from an installed plugin, not a checkout of this repo —
there is normally nothing on disk to find. **If a directory that looks like
this repo (or `hpc-agent-core`) happens to exist somewhere on the machine,
it is not part of your installation and not something to `cd` into, read
example files from, or run its `.venv/bin/python` against.** Everything you
need is in this file. Don't search the filesystem for it.

Likewise: `hpc_agent_core.client`/`connect_sync` is code that goes *inside
the notebook you're writing* — it is not a tool for you to shell out to
during your own exploration. While you're doing the actual task (before the
user asks for a notebook), just use your normal MCP tool calls
(`get_resources`, `submit_job`, `fs_download`, ...) exactly as always. Don't
write and run a Python one-liner importing `hpc_agent_core.client` to "test"
something yourself — that's the deliverable's code, not yours to execute.

## What the notebook needs

Write the notebook to the user's own working directory (or wherever they
say) — not back into any plugin/repo location. Self-contained skeleton, no
external file needed to see the pattern:

```python
from hpc_agent_core.client import connect_sync, dev_params, pinned_params

CACHE_DIR = "./.hpc_cache/<short-name-for-this-notebook>"

# If the person running this notebook has this repo's server/ checked out
# and pip-installed (pip install -e path/to/RCCS-CloudAgent/server):
hpc = connect_sync(dev_params("cloud_mcp", "hpc_server"), mode="lazy", cache_dir=CACHE_DIR)

# Otherwise (no local checkout needed) — the same launch a fresh
# `/plugin install` uses under the hood:
hpc = connect_sync(
    pinned_params(
        "https://github.com/william-dawson/RCCS-CloudAgent.git",
        "rccs-cloud-hpc-mcp",
        ref="main",          # pin to a tag/commit instead, for a notebook meant to outlive today
        subdirectory="server",
    ),
    mode="lazy", cache_dir=CACHE_DIR,
)

# ... the real steps, whatever they actually were, e.g.:
job = hpc.submit_job(spec={...})
status = hpc.wait_for_job(job["job_id"])
result = hpc.fs_download(path="...", local_path="./downloads/result.json")

hpc.close()
```

Prefer `pinned_params` unless you know the notebook will only ever be run on
a machine with this repo's `server/` already checked out and installed — it's
the version that works for literally anyone, no setup beyond `pip install
hpc-agent-core` (well, `hpc_agent_core.client` needs no `cloud_mcp` install
at all for `pinned_params`, since `uv tool run` fetches and runs the server
itself).

No `await`, no `async with` anywhere in the notebook — `connect_sync` blocks,
plain method calls.

Three caching modes, chosen once on `connect_sync`:
- `mode="live"` — always hits the real cluster.
- `mode="lazy"` — cache-first, live on a miss. What you iterate with.
- `mode="replay"` — cache-only, no SSH at all. What makes the notebook work
  for someone with no cluster account, once its `.hpc_cache/` is committed
  alongside it.

`wait_for_job` and `fs_download` have bespoke caching (terminal-poll-only,
and actual downloaded bytes respectively) — you don't need to know the
mechanism, just that calling them normally is correct; don't hand-roll your
own polling loop or file copy around them.

## Workflow

1. **Re-derive the narrative, don't transcribe the chat.** Minimal clean
   sequence of calls that produces the user's actual result — cut dead ends.
   One honest sentence in markdown for anything genuinely informative (a
   partition that turned out to be busy, say), not a replay of every
   detour.

2. **Structure**: markdown cell explaining the next step, then the code cell
   that does it.

3. **Never author a cell's output.** Actually run the code (a plain script
   is fine for this step — real execution is what matters, not the `.ipynb`
   format specifically), capture the real stdout/return values, and only
   then write them into the notebook's saved outputs.

4. **Validate with `mode="live"` from a clean cache directory** before
   calling it done. Then copy that real cache directory's contents next to
   the notebook and mention it should be committed (if the user is putting
   this under version control) — that's what makes `mode="replay"` work
   later for someone else.

5. If the job needs a partition choice and it's not obvious, check
   `get_resources`/`sinfo` for idle capacity rather than guessing.
