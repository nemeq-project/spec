# Architecture

Nemeq is organized as a small kernel (Libbu) plus a set of agents (Epishu), 
an immutable package store, and four filesystem zones (Ersetu).

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
- **scheduler** (Adannu) — runs scheduled tasks, timer-based with optional polling
- **capd** — holds capability state, checks permissions
- **observer** (Massaru) — manages animas, bodies and embodiments; 
  install, run, stop, commit, export

Short-lived, called as CLI tools:

- **himem** — memory pager: page-in, page-out, pin, evict, status
- **nd** — numeric agent: get, aggregate, compare, history, find; returns (value, link)
- **tabled** — data lake: merge, query, history
- **verify** (Kittu) — checks independent paths, quorum, minority report
- **consolidate** — daily to weekly to monthly, append-only, with preview

Every agent, long-lived or short-lived, has the same interface: CLI, JSON 
on stdout, exit code. Composition is via subprocess, not shell.

## Separation of domains

Three different list commands, three different domains:

| Command | Agent | What it lists |
|---|---|---|
| `massaru list animas` | observer (Massaru) | installed animas (Zi) |
| `massaru list bodies` | observer (Massaru) | bodies (Su), templates and instances |
| `massaru list embodiments` | observer (Massaru) | embodiments (Tush) |
| `adannu list` | scheduler (Adannu) | scheduled tasks (timer-based) |
| `himem status` | memory pager | resident set, budget, faults |
| `nemeq status` | (aggregator) | combined JSON: animas + embodiments + tasks + memory |

A combined view is provided by `nemeq status` (separate utility) that 
calls each agent and aggregates the JSON. The observer does not know 
about scheduled tasks; the scheduler does not know about animas. This is 
deliberate — domain isolation keeps each agent small and replaceable.

## Package store (immutable)

Read-only after install. Not in `/var` because nothing here changes 
at runtime.

| Path | Content |
|---|---|
| `/opt/nemeq/animas/<name>/<version>/` | installed animas (Zi): `anima.toml`, `skills/`, `knowledge/`, `style/`, `SIGNATURE` |
| `/opt/nemeq/bodies/templates/<name>/` | body templates (Su): `su.toml`, `structure/`, `base-knowledge/`, `SIGNATURE` |

Removal and reinstallation go through `massaru remove` and 
`massaru install`. Nothing writes here except the observer.

## Variable zones (Ersetu)

Five filesystem territories with different persistence rules.

| Zone | Path | Persistence | Writer | Purpose |
|---|---|---|---|---|
| proc | `/proc/nemeq` | volatile, generated on read | Libbu only | live state: context, agents, cost, qatum |
| run | `/run/nemeq` | volatile, cleared at boot | Epishu | paqidutu draft, pid files, locks |
| state | `/var/nemeq/state` | persistent, atomic (tmp+rename) | Epishu | paqidutu final, zikru-index, status-today |
| data | `/var/nemeq/data` | persistent, append-only | Epishu | daily, atoms, tuppu (logs), archive |
| var | `/var/nemeq/{bodies,embodiments,zikru-zi}` | persistent, mutable | Epishu | body instances, embodiments, anima memory |

The distinction between **state** (atomic replace), **data** 
(append-only) and **var** (mutable) is by write rule, not by path. 
All three live under `/var/nemeq`, but follow different contracts:

- **state** — replaced atomically (tmp + rename), last write wins
- **data** — appended, never modified, never deleted (archive is ADD-only)
- **var** — regular mutable files (`su.toml`, `tush.toml`, zikru files)

## Anima, body, embodiment

- An **anima** (Zi) is installed read-only under 
  `/opt/nemeq/animas/<name>/<version>/`.
- A **body** (Su) is a template under 
  `/opt/nemeq/bodies/templates/<name>/` plus an instance under 
  `/var/nemeq/bodies/instances/<id>/`.
- An **embodiment** (Tush) is the life of an anima in one body, under 
  `/var/nemeq/embodiments/<id>/`. It holds `tush.toml` 
  (with `shanu_count` and stats), `work/` (ephemeral), 
  and `shanu.log` (append-only log of runs).
- An **incarnation** (Shanu) is one run. It starts by mounting the 
  anima read-only and the anima memory as an overlay; it ends with an 
  automatic commit of anima memory (Zikru Zi) and body memory (Zikru Su).
- **Anima memory** (Zikru Zi) lives under 
  `/var/nemeq/zikru-zi/<name>@<version>/` and is committable — it 
  can be exported as a new anima with memory.
- **Body memory** (Zikru Su) lives under 
  `/var/nemeq/bodies/instances/<id>/zikru-su/` and never leaves 
  the installation.

## Riksu (manifest)

Single TOML file at `/etc/nemeq/riksu.toml`. It is both the SDD and 
the runtime config — no translation layer.

```
[profile]
name = "versta"
strict = true

[agents.init]
role = "init"
restart = "supervise"          # kernel restarts

[agents.reu]
role = "watchdog"
restart = "init"

[agents.adannu]
role = "scheduler"
restart = "reu"
poll_interval = 60

[agents.capd]
role = "capd"
restart = "reu"
qatum_dir = "/proc/nemeq/qatum"

[agents.massaru]
role = "observer"
restart = "reu"
animas_dir = "/opt/nemeq/animas"
templates_dir = "/opt/nemeq/bodies/templates"
instances_dir = "/var/nemeq/bodies/instances"
embodiments_dir = "/var/nemeq/embodiments"
zikru_zi_dir = "/var/nemeq/zikru-zi"

[agents.himem]
pinned = ["L0", "L1"]
budget = 16000

[agents.nd]
strict = true
taint_track = true

[agents.tabled]
root = "/var/nemeq/data"

[agents.kittu]
paths = 3
require_link = true

[animas]
artist-vangogh = { version = "2.1.0" }
programmer-base = { version = "1.2.0" }

[embodiments]
artist-vangogh = { body = "su-01" }
```

## Supervision hierarchy

```
Libbu ── supervise(PID 1) ──▶ init
                              │
                              ├──▶ reu ──▶ adannu, capd, massaru
                              │
                              └──▶ massaru ──▶ incarnation processes
                                               (one per running Tush)

reu does not watch init or itself.
init does not watch itself — Libbu holds it.
Cycle closed: Libbu -> init -> reu -> Epishu.
```

## Lifecycle: from install to stop

```
Mahiru (marketplace)
    │
    │ anima package (.anima, encrypted, signed)
    ▼
massaru install
    → /opt/nemeq/animas/<name>/<version>/         (Zi, read-only)
    │
    │ riksu declares [animas]
    ▼
init reads Riksu, starts agents in order
    │
    │ riksu declares [embodiments]
    ▼
massaru run
    → creates /var/nemeq/embodiments/<id>/
    → links: zi (ro) + zikru-zi (overlay) + zikru-su
    → shanu_count += 1
    → spawn Epishu process
    ▼
Incarnation (Shanu) runs
    │
    │ SIGTERM → agent flushes to pending buffers
    ▼
massaru stop
    → auto-commit:
      zikru-zi pending → /var/nemeq/zikru-zi/<name>@<ver>/
      zikru-su pending → /var/nemeq/bodies/instances/<id>/zikru-su/
    → shanu_count logged to shanu.log
    ▼
massaru export (optional)
    → new .anima with memory
    → new soul_id (content hash), new license, new recipient key
```

## What is not in the kernel

Logging, memory consolidation, verification, data handling, capability 
enforcement beyond the check itself, package management, scheduling — 
all live in agents, not in the kernel. Libbu stays at six primitives.
