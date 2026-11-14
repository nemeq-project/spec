# Architecture

Nemeq is organized as a small kernel (Libbu) plus a set of agents (Epishu),
an immutable package store, and five filesystem zones (Ersetu).

## Kernel (Libbu)

Six primitives. Nothing else.

1. Session loop — one turn of an agent against the LLM
2. Context RAM — what is resident in the current window
3. Dispatch — route a call to a tool, the LLM, or the user
4. Capability check — before any tool or platform call
5. Spawn and reap — agent processes
6. Supervise — restart the init agent if it dies (only for PID 1)

The kernel is deterministic. It does not think. It does not hold state
beyond the session. It knows nothing about sleep, incarnations or animas:
those are agent-level concepts (see below).

## Built-in agents (Epishu)

Long-lived:

- **init** — reads the manifest (Riksu), starts agents in order, holds the session
- **watchdog** (Reu) — restarts failed services, except init and itself
- **scheduler** (Adannu) — runs scheduled tasks; owns the sleep triggers
- **capd** — holds capability state, checks permissions
- **observer** (Massaru) — manages animas, bodies and incarnations;
  `install`, `remove`, `run` (wake), `sleep`, `stop` (park body), `commit`,
  `export`, `list`, `inspect`

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
| `massaru list incarnations` | observer (Massaru) | incarnations (Tush) |
| `adannu list` | scheduler (Adannu) | scheduled tasks (timer-based) |
| `himem status` | memory pager | resident set, budget, faults |
| `nemeq status` | (aggregator) | combined JSON: animas + incarnations + tasks + memory |

A combined view is provided by `nemeq status` (separate utility) that
calls each agent and aggregates the JSON. The observer does not know
about scheduled tasks; the scheduler does not know about animas. This is
deliberate — domain isolation keeps each agent small and replaceable.

## Package store (immutable)

Read-only after install. Not in `/var` because nothing here changes
at runtime.

| Path | Content |
|---|---|
| <code>/opt/nemeq/animas/&lt;name&gt;/&lt;version&gt;/</code> | installed animas (Zi): `anima.toml`, `skills/`, `knowledge/`, `style/`, `SIGNATURE` |
| <code>/opt/nemeq/bodies/templates/&lt;name&gt;/</code> | body templates (Su): `su.toml`, `structure/`, `base-knowledge/`, `SIGNATURE` |

Removal and reinstallation go through `massaru remove` and
`massaru install`. Nothing writes here except the observer.

An installed anima that is not in any body is **not active**: it exists as
an artifact with `incarnations_count = 0`. Activity requires an incarnation.

## Variable zones (Ersetu)

Five filesystem territories with different persistence rules.

| Zone | Path | Persistence | Writer | Purpose |
|---|---|---|---|---|
| proc | `/proc/nemeq` | volatile, generated on read | Libbu only | live state: context, agents, cost, qatum |
| run | `/run/nemeq` | volatile, cleared at boot | Epishu | paqidutu draft, pid files, locks |
| state | `/var/nemeq/state` | persistent, atomic (tmp+rename) | Epishu | paqidutu final, zikru-index, status-today |
| data | `/var/nemeq/data` | persistent, append-only | Epishu | daily, atoms, tuppu (logs), recoveries, archive |
| var | `/var/nemeq/{bodies,embodiments,zikru-zi}` | persistent, mutable | Epishu | body instances, incarnation dirs, anima memory |

The distinction between **state** (atomic replace), **data**
(append-only) and **var** (mutable) is by write rule, not by path.
All three live under `/var/nemeq`, but follow different contracts:

- **state** — replaced atomically (tmp + rename), last write wins
- **data** — appended, never modified, never deleted (archive is ADD-only)
- **var** — regular mutable files (`su.toml`, `tush.toml`, zikru files)

`/var/nemeq/embodiments/` keeps its historical name; **in prose the entity
is called an incarnation (Tush)**. The path is the only place where the word
"embodiment" survives.

## Anima, body, incarnation

- An **anima** (Zi) is installed read-only under
  <code>/opt/nemeq/animas/&lt;name&gt;/&lt;version&gt;/</code>. It carries
  `incarnations_count` in its psychomatrix.
- A **body** (Su) is a template under
  <code>/opt/nemeq/bodies/templates/&lt;name&gt;/</code> plus an instance under
  <code>/var/nemeq/bodies/instances/&lt;id&gt;/</code>. It carries `days_count`.
- An **incarnation** (Tush) is the pair "one anima + one body", under
  <code>/var/nemeq/embodiments/&lt;id&gt;/</code>. It lives as long as the body
  lives and contains many days and sleeps. It holds `tush.toml`
  (`days_count`, a read-only mirror of `incarnations_count`, and a reference
  to the anima), `work/` (ephemeral), and `days.log` (append-only log of
  awakenings, formerly `shanu.log`).
- **Anima memory** (Zikru Zi) lives under
  <code>/var/nemeq/zikru-zi/&lt;name&gt;@&lt;version&gt;/</code> and is
  committable — it can be exported as a new anima with memory.
- **Body memory** (Zikru Su) lives under
  <code>/var/nemeq/bodies/instances/&lt;id&gt;/zikru-su/</code> and never leaves
  the installation.

Ownership rule: `incarnations_count` belongs to the anima (psychomatrix);
`tush.toml` holds only a mirror for reading. `days_count` belongs to the body
(`tush.toml`). On a mismatch the anima's psychomatrix wins.

Parallel incarnations of one anima are **forbidden in the PoC**: one anima —
one body at a time. Two bodies writing into one `zikru-zi` would leave no
single owner of memory or of the counter.

## Sleep, awakening, day

- **Sleep (Sittu)** is the nightly cycle inside an incarnation. On sleep
  everything is processed and committed: `zikru-zi`, `zikru-su`, `paqidutu`,
  context RAM. The process either terminates (hibernation) or pauses.
  Sleep does not change `incarnations_count`.
- **Awakening (Tebu)** starts a new day: `paqidutu` is read, `days_count`
  is incremented, context is loaded from disk. The process is spawned
  (after hibernation) or resumed (after pause).
- **Day (Umu)** is the waking period between two sleeps.

Triggers of sleep:

1. `adannu` — the schedule;
2. idle timeout — 30 minutes without activity by default;
3. explicit command — `massaru sleep`.

**Context overflow is not a trigger.** Overflow is refused by `himem` as
`ContextOverflow`; it never puts the incarnation to sleep.

A process crash is a **sharp sleep** (loss of consciousness): no commit; at
the next awakening the last `paqidutu` plus `zikru` are read, and work since
the last commit is lost. Every such event is recorded in `recoveries.jsonl`.

## Memory bridge

At body change (Ug) the observer runs the memory bridge. It is mandatory,
not on request. From the old body's `zikru-su` it extracts generalised
lessons, filters them for PII, and **appends** them to `zikru-zi`.

Four rules (the list is extensible):

1. **Generalisability** — will the lesson help another anima, with another
   user, in another project? If not, it stays in `zikru-su` and is lost.
2. **PII filter after generalisation** — the lesson is checked for personal
   data patterns (names, contacts, dates, quotes, identifiers). Rejected
   records are logged to `recoveries.jsonl`.
3. **Provenance** — every record that came through the bridge carries
   `source_incarnation`, so an export can show which bodies taught the anima.
4. **Append-only irreversibility** — the bridge never rewrites `zikru-zi`.
   A contradicting lesson is appended and the contradiction is recorded in
   the `tension` field.

Implementation variant **C**: LLM generalisation + PII filter + review at the
next export. `zikru-zi` is not considered clean until export; Q3 protection
fires on the way out, without blocking daily work.

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
idle_timeout_minutes = 30

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
embodiments_dir = "/var/nemeq/embodiments"   # technical path; entity = incarnation
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

[incarnations]
artist-vangogh = { body = "su-01" }
```

## Supervision hierarchy

```
Libbu ── supervise(PID 1) ──▶ init
                              │
                              ├──▶ reu ──▶ adannu, capd, massaru
                              │
                              └──▶ massaru ──▶ incarnation processes
                                               (one per active Tush)

reu does not watch init or itself.
init does not watch itself — Libbu holds it.
Cycle closed: Libbu -> init -> reu -> Epishu.
```

## Lifecycle: from install to Zinuntu

```
Mahiru (marketplace)
    │
    │ anima package (.anima, encrypted, signed)
    ▼
massaru install
    → /opt/nemeq/animas/<name>/<version>/          (Zi, read-only)
    → incarnations_count = 0 (not in a body yet)
    │
    │ riksu declares [animas] and [incarnations]
    ▼
init reads Riksu, starts agents in order
    │
    ▼
massaru run (wake)
    → creates /var/nemeq/embodiments/<id>/          (Tush: anima + body)
    → tush.toml: days_count = 0, mirror of incarnations_count
    → links: zi (ro) + zikru-zi (overlay) + zikru-su
    → incarnations_count += 1 (in the anima)
    → spawn Epishu process
    ▼
Day (Umu) — work, sessions, tool calls
    │
    │ sleep trigger: adannu | idle 30 min | massaru sleep
    ▼
Sittu (sleep)
    → commit: zikru-zi, zikru-su, paqidutu, context RAM
    → process hibernates or pauses
    ▼
Tebu (awakening)
    → read paqidutu, load context, days_count += 1
    → spend another day … (loop back to work)
    │
    ├── Ug (body lost)
    │     → anima parked; body may be restored from backup (same incarnation)
    │     → memory bridge: generalised lessons zikru-su → zikru-zi
    │     → new body ⇒ new incarnation, incarnations_count += 1
    │
    └── Zinuntu (anima destroyed)
          → artifact wiped, record in revoked_animas.txt
          → body may stay empty (Su without Zi) or take another anima
          ▼
massaru export (optional, before Zinuntu)
    → new .anima with memory
    → new anima_id (content hash), new license, new recipient key
```

## What is not in the kernel

Logging, memory consolidation, verification, data handling, capability
enforcement beyond the check itself, package management, scheduling, sleep
and incarnation lifecycle — all live in agents, not in the kernel. Libbu
stays at six primitives.
