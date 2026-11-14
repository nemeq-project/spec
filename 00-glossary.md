# Glossary

Full vocabulary of Nemeq. **Brand term first, English term in parentheses,
then the definition.** Brand terms use Latin letters only; the diacritic
forms appear only in academic citations.

## Existence

| Term | Definition |
|---|---|
| **Zi (Anima)** | The soul: a portable package of skills, knowledge, style, psychomatrix and prompts, plus anima memory (`zikru-zi`). Read-only after install. Survives bodies. Carries `incarnations_count`. Distributed as a `.anima` package. Field name in code: `anima_id`. |
| **Su (Body)** | The body: the environment an anima lives in — folder structure, capability declaration, `qatum`, body memory (`zikru-su`), handoff (`paqidutu`), `days_count`. Bound to one installation. May be empty (no anima); then it works with a minimal base knowledge set. |
| **Tush (Incarnation)** | One anima plus one body. Lives as long as the body lives. Inside an incarnation there are many days and sleeps. `tush.toml` holds `days_count` and a mirror of `incarnations_count`. Technical path: `/var/nemeq/embodiments/<id>/`. |
| **Sittu (Sleep)** | The nightly cycle inside an incarnation. On sleep everything is processed and committed: `zikru-zi`, `zikru-su`, `paqidutu`, context RAM. The process may either terminate (hibernation) or pause. Sleep does not change `incarnations_count`. Sleep is not death — it is a pause with full state preservation. |
| **Tebu (Awakening)** | The transition from sleep to waking. A new day starts by reading `paqidutu` (handoff). `days_count` plus one. Context is loaded from disk. The process is either spawned (after hibernation) or resumed (after pause). |
| **Umu (Day)** | The waking period between two sleeps. Inside a day: sessions, LLM calls, work. `days_count` counts days within the current body. |
| **Ug (Body death)** | The body is lost — broken, lost or replaced. The anima is parked; it may be transplanted into a new body (a new incarnation) or back into the old body after restore from backup (continuation of the incarnation). Not to be confused with Zinuntu. Sumerian *ug* — to die, to be dead. |
| **Zinuntu (Anima death)** | Irreversible destruction of the anima artifact: `zikru-zi`, psychomatrix, prompts — everything. Recorded in `revoked_animas.txt`. The body is not necessarily lost; it may stay empty (Su without Zi) or be transplanted to another anima. Difference from Ug: the body can be restored from backup, the anima cannot. Sumerian *zi-nu-tuku* — one that has no soul. |

## Memory and time

| Term | Definition |
|---|---|
| **Zikru Zi (Anima memory)** | Memory of the soul. Lives in the anima, travels with export, committed at every sleep. Holds craft lessons, style evolution, self-knowledge as a type, corrected mistakes, generalised conclusions. Never holds personal data, quotes, names, dates or facts about a specific user (Q3). TTL in calendar days (values not fixed yet). |
| **Zikru Su (Body memory)** | Memory of the body. Lives in the body, lost when the body changes (unless returning to the old body from backup). Holds this user, project history, quotes, dates, personal preferences — everything about this incarnation in the moment. TTL in calendar days (values not fixed yet). |
| **Paqidutu (Handoff)** | Work transfer at sleep and awakening. Lives on the body's disk (`/run/nemeq/paqidutu-draft`, `/var/nemeq/state/paqidutu`). Written at sleep, read at awakening. Lost when the body changes: a new incarnation starts with full amnesia about the current task. Deliberate — confidential data does not travel with the body. |
| **Memory Bridge** | Mechanism at body change (Ug): generalised lessons are extracted from the old body's `zikru-su`, filtered for PII, and appended to `zikru-zi`. Mandatory, not on request. Four rules (extensible): generalisability, PII filter after generalisation, provenance (`source_incarnation` on every record), append-only irreversibility. Implementation variant C: LLM generalisation + PII filter + review at next export; `zikru-zi` is not considered clean until export. |
| **Incarnations Count** | Counter of incarnations. Lives in the anima (in the psychomatrix, inherited on export). Grows only when a new body is acquired. An anima installed but not yet in a body has zero. If the body is lost and the anima has no new body yet, the counter waits. Returning to the old body also adds one — it is a new incarnation even if the body is the same. |
| **Days Count** | Counter of days. Lives in the body (`tush.toml`). Grows at every awakening. A new body starts from zero. Returning to the old body continues from the previous value — the body remembers its days. |
| **TTL (Time To Live)** | Lifetime of a memory record since last access. Measured in calendar days. Concrete values are not fixed in the current revision. A second parameter is planned — "days of inactivity". |
| **Shanu (deprecated)** | Former name of the run counter ("one run inside an embodiment"). Replaced by **Tush (Incarnation)** and **Incarnations Count**. Must not appear in new documents. |

## Kernel and agents

| Term | Definition |
|---|---|
| **Libbu (Kernel)** | The deterministic kernel: six primitives — session loop, context RAM, dispatch, capability check, spawn/reap, supervise. Knows nothing about sleep, incarnations or animas. Operates on processes and sessions. |
| **Epishu (Agent)** | Any running agent: built-in or installed. |
| **Init** | Agent with PID 1. Reads Riksu, starts agents in order, holds the session. Restarted by the kernel if it dies. |
| **Reu (Watchdog)** | Service agent that restarts failed services, except init and itself. From Akkadian "shepherd". |
| **Adannu (Scheduler)** | Service agent that runs scheduled tasks. Sleep triggers: schedule, idle timeout, explicit command (`massaru sleep`). Context overflow does **not** trigger sleep. |
| **Massaru (Observer)** | Service agent managing animas, bodies and incarnations. Commands: `install`, `remove`, `run` (wake), `sleep`, `stop` (park body), `commit`, `export`, `list animas`, `list bodies`, `list incarnations`, `inspect`. On sleep it commits everything; on body change it runs the memory bridge. |
| **Capd (Capability service)** | Service agent owning the capability registry: `grant`, `revoke`, `check`. |
| **Himem (Memory pager)** | Memory agent: paging, eviction. Works inside a day; state is committed at sleep. |
| **Nd (Numeric agent)** | Numeric agent. Returns `(value, unit, link, nd_verified, exceptions)`. Four exception codes: `NO_DATA`, `NO_KEY`, `NO_UNIT`, `OUT_OF_RANGE`. Taint is sticky: a number used outside `nd` becomes `nd_unverified`. |
| **Tabled (Data lake)** | Local data lake of infrastructure metadata. |
| **Kittu (Verification)** | Independent-path verification (Q2, Q5). Five output fields: `verdict`, `effective_votes`, `independence`, `minority`, `honest_confidence`. Independent of sleep and incarnations. |
| **Baru (Extractor)** | Extractor: artifacts into a psychomatrix draft. Stochastic, uses an LLM. Not in the PoC. |
| **Banu (Compiler)** | Compiler of souls. Two operations: `synth` (LLM, stochastic, prompt draft) and `pack` (no LLM, deterministic, packaging into `.anima` with `anima_id`). |
| **Ummanu (Umbrella)** | Umbrella over Baru and Banu: a composite package of operations on souls and bodies. |

## Capability, zones, configuration

| Term | Definition |
|---|---|
| **Qatum (Capability)** | A right granted to an agent. Live-state directory: `/proc/nemeq/qatum`. The kernel answers from its own registry, never from caller-supplied data (P-01). |
| **Ersetu (Zones)** | Filesystem territories with different write rules: `proc`, `run`, `state`, `data`, `var`. |
| **Riksu (Manifest)** | Single TOML file that is both the SDD and the runtime configuration. |
| **Parsu (Profile)** | A zone's configuration: which agents run, which animas are loaded. |
| **Tuppu (Log)** | Append-only record of events. |
| **AAL (Agent Abstraction Layer)** | Boundary between Nemeq and the platform (LLM API, filesystem, spawn). Contract: `read`, `write`, `list`, `spawn`, `stop`, `status`, `schedule`, `notify`, `check_permission`, `context_usage`, `get_cost`, `log_event`, `call_llm`. |
| **Anima_id** | Identifier of an anima. Hash of the artifact (psychomatrix + `prompts-final`). Stable for an installed anima; does not include runtime memory. Export with memory produces a new `anima_id`. |
| **Revoked_animas.txt** | List of revoked animas (piracy, subject request). The marketplace and verification check it; the OS does not. |

## Ecosystem

| Term | Definition |
|---|---|
| **Mahiru (Marketplace)** | External service distributing animas and bodies. Not part of the OS. |
| **Psychomatrix** | Structured intermediate representation of a soul (TOML). Input of Banu, output of Baru. Two layers: `computed` (deterministic: astrology and similar) and `interpreted` (heuristics, approved by the author). Modes: `reconstructed`, `documented`, `composite`. Holds `subject_consent`, `hard_constraints`, `tensions`. |
| **Quoriosity** | Agreement-gated curiosity: an agent asks and acts only when its curiosity is confirmed by a sufficient quorum of independent grounds. A project neologism from *quorum* + *curiosity*. |
| **NEMEQ** | Natural Embodied Multi-agent Ecosystem for Quoriosity. From Akkadian *nemeq* (nēmequ) — wisdom, secret knowledge. |

## Naming rules

- **Latin letters only** in code, filenames and brand terms; no diacritics.
  Academic transliterations (`nēmequ`, `bārû`) may appear only in citations.
- **Brand term first** in this glossary: `Sittu (Sleep)`. In prose, the
  English term may lead at first mention: "sleep (Sittu)".
- **One concept — one name.** `incarnation` is the English term; `Tush` is
  its brand. `embodiment` survives only as a technical path
  (`/var/nemeq/embodiments/`), never as a prose term.
- **Built-in tools keep functional names** in code (`himem`, `nd`, `tabled`,
  `capd`, `init`). Brand terms are documented aliases.
- **Two-word compounds** are written with a space: `Zikru Zi`, `Zikru Su`.
- The word `soul` is forbidden in declarations; use `anima` / `anima_id`.
