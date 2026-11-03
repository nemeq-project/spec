# Architecture

Nemeq is organized as a small kernel (Libbu) plus a set of agents (Epishu) 
and four filesystem zones (Ersetu).

## Kernel (Libbu)

Six primitives. Nothing else.

1. Session loop — one turn of an agent against the LLM
2. Context RAM — what is resident in the current window
3. Dispatch — route a call to a tool, the LLM, or the user
4. Capability check — before any tool or platform call
5. Spawn and reap — agent processes
6. Supervise — restart the init agent if it dies (only for PID 1)

The kernel is deterministic. It does not think. It does not hold state 
beyond the session.

## Built-in agents (Epishu)

Long-lived:

- **init** — reads the manifest (Riksu), starts agents in order, holds the session
- **watchdog** (Reu) — restarts failed services, except init and itself
- **cron** (Adannu) — runs scheduled tasks, timer-based with optional polling
- **capd** — holds capability state, checks permissions
- **observer** (Massaru) — manages animas, bodies and embodiments; install, run, stop, commit, export

Short-lived, called as CLI tools:

- **himem** — memory pager: page-in, page-out, pin, evict, status
- **nd** — numeric agent: get, aggregate, compare, history, find; returns (value, link)
- **tabled** — data lake: merge, query, history
- **verify** (Kittu) — checks independent paths, quorum, minority report
- **consolidate** — daily to weekly to monthly, append-only, with preview

Every agent, long-lived or short-lived, has the same interface: CLI, JSON 
on stdout, exit code. Composition is via subprocess, not shell.

## Zones (Ersetu)

Four filesystem territories with different persistence rules:

| Zone | Path | Persistence | Purpose |
|---|---|---|---|
| proc | `/proc/nemeq` | volatile, generated on read | live state: context, agents, cost, capabilities |
| run | `/run/nemeq` | volatile, cleared at boot | handoff drafts, pid files, locks |
| state | `/var/nemeq/state` | persistent, atomic (tmp+rename) | handoff final, memory index, daily status |
| data | `/var/nemeq/data` | persistent, append-only | daily, atoms, logs, archive |

## Anima, body, embodiment

- An **anima** (Zi) is installed read-only under `/var/nemeq/installed/<name>/<version>/`.
- A **body** (Su) is a template plus an instance under `/var/nemeq/bodies/`.
- An **embodiment** (Tush) is the life of an anima in one body, under 
  `/var/nemeq/embodiments/<id>/`. It holds a counter of incarnations (Shanu) 
  and an append-only log of runs.
- An **incarnation** (Shanu) is one run. It starts by mounting the anima 
  read-only and the anima memory as an overlay; it ends with an automatic 
  commit of anima memory (Zikru Zi) and body memory (Zikru Su).

## What is not in the kernel

Logging, memory consolidation, verification, data handling, capability 
enforcement beyond the check itself — all live in agents, not in the kernel.
