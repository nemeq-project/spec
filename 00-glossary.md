# Glossary

Key terms of the Nemeq system. English first, Sumero-Akkadian brand term 
in parentheses. All brand terms use Latin letters only.

| English term | Brand term | Definition |
|---|---|---|
| Anima | Zi | Portable package of skills, knowledge and style. Read-only after install. |
| Body | Su | Empty environment: folder structure and capabilities. Can run without an anima, with minimal base knowledge. |
| Embodiment | Tush | The life of an anima in one body. Persists across many runs. |
| Incarnation | Shanu | One run inside an embodiment. A counter, not a separate entity. |
| Observer | Massaru | Service agent managing animas, bodies and embodiments. Handles install, run, stop, commit, export. |
| Watchdog | Reu | Service agent that restarts failed services, except init and itself. |
| Scheduler | Adannu | Service agent that runs scheduled tasks. |
| Marketplace | Mahiru | Place where animas and bodies are distributed. Separate from the OS. |
| Kernel | Libbu | The six primitives at the core. |
| Agent | Epishu | Any running worker: built-in or user-installed. |
| Capability | Qatum | A granted right to perform an action. |
| Verification | Kittu | Tool and process that checks independent paths, quorum and minority report. |
| Log | Tuppu | Append-only record of events. |
| Memory | Zikru | Persistent record of what is known. Splits into anima memory (Zikru Zi) and body memory (Zikru Su). |
| Handoff | Paqidutu | Transfer of work between agents or between runs. |
| Manifest | Riksu | Single TOML file that is both SDD and runtime config. |
| Profile | Parsu | A zone's configuration: which agents run, which animas are loaded. |
| Zone | Ersetu | A filesystem territory: proc, run, state, data. |
| Quoriosity | — | Agreement-gated curiosity: an agent asks and acts only when its curiosity is confirmed by a sufficient quorum of independent grounds. |
| NEMEQ | — | Natural Embodied Multi-agent Ecosystem for Quoriosity. |

## Naming rules

- **Latin letters only** in code, filenames and brand terms. No diacritics.
- **English first** in prose, brand term in parentheses at first mention.
- **Built-in tools keep functional names** in code (`himem`, `nd`, `tabled`, 
  `verify`, `capd`, `init`, `cron`, `watchdog`). The brand terms are their 
  documented aliases.
- **Two-word compounds** are written with a space: `Zikru Zi`, `Zikru Su`.
