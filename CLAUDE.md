# 4G EPC & IMS Knowledge Wiki — Agent Schema

## Purpose

This is a persistent, LLM-maintained wiki for deep understanding of 4G LTE's **Evolved Packet Core (EPC)** and **IP Multimedia Subsystem (IMS)**. The wiki is the primary artifact. Raw sources are immutable inputs. The LLM (you) owns all wiki content.

---

## Directory Layout

```
4g-library/
├── CLAUDE.md           ← this file: rules for the agent
├── raw/                ← immutable source documents (never modify)
│   ├── articles/       ← clipped web articles (.md)
│   ├── papers/         ← academic papers, specs, 3GPP docs (.md / .pdf)
│   ├── notes/          ← hand-written notes, transcripts (.md)
│   └── assets/         ← images referenced by sources
└── wiki/               ← LLM-maintained knowledge base
    ├── index.md        ← master catalog of all wiki pages
    ← log.md           ← append-only chronological event log
    ├── overview.md     ← high-level synthesis of the domain
    ├── entities/       ← one page per network element (MME, HSS, P-CSCF, etc.)
    ├── concepts/       ← one page per concept (bearer, attach procedure, etc.)
    ├── procedures/     ← one page per call/data flow (attach, handover, VoLTE call)
    ├── protocols/      ← one page per protocol (S1-AP, Diameter, SIP, GTP, etc.)
    ├── interfaces/     ← one page per interface (S1-MME, S11, Gx, Rx, etc.)
    ├── sources/        ← one summary page per ingested source
    └── analyses/       ← query answers, comparisons, filed explorations
```

---

## Page Frontmatter Convention

Every wiki page (except index.md and log.md) MUST begin with YAML frontmatter:

```yaml
---
title: "Page Title"
type: entity | concept | procedure | protocol | interface | source | analysis | overview
tags: [list, of, relevant, tags]
sources: [list of source filenames that contributed to this page]
updated: YYYY-MM-DD
---
```

---

## Index Format (wiki/index.md)

The index is the LLM's navigation map. Structure:

```markdown
# Wiki Index
_Last updated: YYYY-MM-DD — N pages total_

## Overview
- [overview.md](overview.md) — Domain synthesis and current thesis

## Entities
- [entities/MME.md](entities/MME.md) — Mobility Management Entity: session/mobility control
...

## Concepts
...

## Procedures
...

## Protocols
...

## Interfaces
...

## Sources
...

## Analyses
...
```

Rules:
- One line per page: `- [filename](path) — one-sentence description`
- Update on every ingest, every new page created, and every significant revision
- Read index.md FIRST before answering any query (to find relevant pages)

---

## Log Format (wiki/log.md)

Append-only. Each entry starts with a parseable header:

```markdown
## [YYYY-MM-DD] <event-type> | <title>

<1-3 sentence summary of what happened, what changed, any notable findings>

Pages touched: [list]
```

Event types: `ingest`, `query`, `lint`, `create`, `update`

Rules:
- Append an entry for EVERY operation (ingest, query that produces a new page, lint pass)
- Never edit past entries
- The log is a history, not a to-do list

---

## Chunked Ingest Workflow

Each Phase is divided into pre-defined chunks (see Ingest Phases below). A chunk is the
unit of one session: read one bounded set of source sections → write all resulting wiki
pages → stop and announce what comes next.

### Per-chunk steps

1. **Identify** the current chunk: check the last `wiki/log.md` entry's `Next chunk:` field,
   or use the chunk the user explicitly named
2. **Read** the source PDF section(s) for this chunk (PDF page range in the chunk table)
3. **Write** all wiki pages the chunk produces:
   - New pages: procedure/concept/entity pages as listed in the chunk table
   - Updated pages: any existing entity/concept/interface pages touched by this content
   - `wiki/sources/<slug>.md` — create or update the source summary page (first chunk of a
     new source only; subsequent chunks of the same source append to it)
4. **Update** `wiki/index.md` — add new pages, update descriptions of revised pages
5. **Append** to `wiki/log.md` — use the chunk log format (see below)
6. **Print** the end-of-chunk announcement (see format below)
7. **STOP** — do not read the next chunk or write any further pages. Wait for "continue".

### End-of-chunk announcement format

```
---
**Chunk [ID] complete** — [topic]
Pages written:
- [page path] — [one-line description]
- ...

Next: **Chunk [next-ID]** — [next topic] ([spec] §[sections])
Say **continue** to proceed, or ask questions about this chunk first.
---
```

### Log entry format for chunks

```markdown
## [YYYY-MM-DD] ingest | [Source §sections] [chunk ID]

<summary>

Next chunk: [next-ID] — [next topic]
Pages touched: [list]
```

### Resuming after a context reset

Read `CLAUDE.md` → `wiki/index.md` → `wiki/log.md`. The last log entry's `Next chunk:`
field identifies exactly what to start. User says "continue" → execute that chunk.

---

One source will typically touch 5–15 wiki pages across all its chunks. This is expected.

---

## Query Workflow

When the user asks a question:

1. Read `wiki/index.md` to identify relevant pages
2. Read those pages
3. Synthesize an answer with citations (link to wiki pages and source pages)
4. If the answer is non-trivial and reusable, **file it** as `wiki/analyses/<slug>.md`
5. If filing, update index.md and log.md
6. If the query revealed a gap, note it explicitly and suggest a source to fill it

---

## Lint Workflow

When the user says "lint the wiki":

1. Scan all pages for: contradictions, stale claims, orphan pages (not in index), missing cross-links
2. Look for concepts mentioned on multiple pages without their own dedicated page
3. Check that all entities mentioned on procedure/interface pages have their own entity page
4. List findings with specific page references
5. Propose fixes; execute with user approval
6. Append lint entry to log.md

---

## Writing Conventions

- **Cross-link liberally**: whenever an entity, concept, protocol, or interface is mentioned, link it: `[[MME]]` (Obsidian wikilink) or `[MME](../entities/MME.md)` (relative markdown)
- **Be precise**: use 3GPP terminology. Prefer TS numbers when citing specs (e.g. 3GPP TS 23.401)
- **Flag uncertainty**: if something is inferred rather than directly stated in a source, note it: `_(inferred)_`
- **Note contradictions**: if a new source contradicts an existing page, add a `> **Contradiction:** ...` blockquote and list both sources
- **Keep pages focused**: one entity/concept/procedure per page. Split pages when they grow unwieldy (>500 lines)
- **Use tables** for parameter lists, interface summaries, comparison of alternatives
- **Use Mermaid diagrams** for ALL visual content — this is the default and mandatory format

### Mermaid Usage Rules

**Always use Mermaid. Never use ASCII art diagrams.** Replace any existing ASCII diagrams with Mermaid equivalents when editing a page.

| Content Type | Mermaid Diagram Type |
|---|---|
| Call flows / message sequences | `sequenceDiagram` |
| Architecture / topology | `graph LR` or `graph TD` |
| State machines (EMM, ECM, SIP dialog) | `stateDiagram-v2` |
| Procedure steps / flowcharts | `flowchart TD` |
| Entity relationships | `erDiagram` |
| Timeline / phase plans | `timeline` |

**Mermaid syntax reminders:**
- Sequence diagrams: use `participant`, `->>` (solid) and `-->>` (dashed), `Note over`, `activate`/`deactivate`
- Graph nodes: use `[Box]`, `(Rounded)`, `{Diamond}`, `((Circle))`
- Label edges: `A -->|label| B`
- Subgraphs: `subgraph Name ... end` to group related nodes
- State diagrams: `[*] --> State`, `State --> State : event`

---

## Domain Glossary (bootstrap — expand over time)

| Abbreviation | Full Name | Category |
|---|---|---|
| EPC | Evolved Packet Core | Architecture |
| IMS | IP Multimedia Subsystem | Architecture |
| MME | Mobility Management Entity | Entity |
| SGW | Serving Gateway | Entity |
| PGW | Packet Data Network Gateway | Entity |
| HSS | Home Subscriber Server | Entity |
| PCRF | Policy and Charging Rules Function | Entity |
| eNodeB | Evolved Node B (LTE base station) | Entity |
| P-CSCF | Proxy-Call Session Control Function | Entity (IMS) |
| S-CSCF | Serving-Call Session Control Function | Entity (IMS) |
| I-CSCF | Interrogating-Call Session Control Function | Entity (IMS) |
| AS | Application Server | Entity (IMS) |
| TAS | Telephony Application Server | Entity (IMS) |
| MGCF | Media Gateway Control Function | Entity (IMS) |
| UE | User Equipment | Entity |
| APN | Access Point Name | Concept |
| PDN | Packet Data Network | Concept |
| EPS | Evolved Packet System | Concept |
| bearer | QoS-defined data pipe between UE and PGW | Concept |
| VoLTE | Voice over LTE | Concept |
| GTP | GPRS Tunneling Protocol | Protocol |
| SIP | Session Initiation Protocol | Protocol |
| Diameter | AAA/policy protocol | Protocol |
| S1-AP | S1 Application Protocol | Protocol |
| NAS | Non-Access Stratum | Protocol |
| IMSI | International Mobile Subscriber Identity | Concept |
| IMPU | IMS Public User Identity | Concept |
| IMPI | IMS Private User Identity | Concept |

---

## Ingest Phases (Planned Order)

These phases define the sequence for building the wiki from the four source specs.
Each phase is broken into chunks. Chunk ID format: `<phase><spec>-<n>` (e.g. `2a-3`).

---

### Phase 1 — Foundations ✓ COMPLETE

| ID | Source | Sections | Topic | Status |
|---|---|---|---|---|
| 1a-1 | TS 23.401 | §4 | EPC architecture + all network elements | ✓ Done |
| 1b-1 | TS 23.228 | §4 | IMS architecture + CSCF roles + identity model | ✓ Done |

---

### Phase 2 — Procedures

#### Phase 2a — TS 23.401 §5 (EPC Procedures)

| ID | Sections | Topic | PDF pages (approx) | Expected wiki pages |
|---|---|---|---|---|
| 2a-1 | §5.3.2 | E-UTRAN Initial Attach | pp 123–143 (~20pp) | `procedures/EPS-attach.md` |
| 2a-2 | §5.3.3–5.3.5 | TAU + Service Request + S1 Release | pp 144–200 (~20pp, focused) | `procedures/TAU.md`, `procedures/service-request.md` |
| 2a-3 | §5.3.8 + §5.4.1–5.4.5 | Detach + Bearer Management | pp 204–236 (~20pp) | `procedures/detach.md`, `procedures/dedicated-bearer.md` |
| 2a-4 | §5.5.1.1–5.5.1.2 | Intra-LTE Handover (X2 + S1) | pp 239–256 (~17pp) | `procedures/X2-handover.md`, `procedures/S1-handover.md` |
| 2a-5 | §5.7 + §5.10 | Information Storage + PDN Connectivity | pp 287–325 (~20pp, focused) | `procedures/PDN-connectivity.md` + entity page updates |

#### Phase 2b — TS 23.228 §5 (IMS Procedures)

| ID | Sections | Topic | PDF pages (approx) | Expected wiki pages |
|---|---|---|---|---|
| 2b-1 | §5.1–5.3 | P-CSCF Discovery + IMS Registration + De-registration | pp 71–87 (~16pp) | `procedures/IMS-registration.md` |
| 2b-2 | §5.4.5 + §5.4.7 | Session Path + QoS/PCC Interaction | pp 90–108 (~18pp) | `procedures/IMS-QoS-bearer.md` |
| 2b-3 | §5.5–5.6 | S-CSCF Routing + Origination (MO Call) | pp 108–129 (~21pp) | `procedures/VoLTE-MO-call.md` |
| 2b-4 | §5.7 + §5.10 | Termination (MT Call) + Session Release | pp 129–154 (~25pp) | `procedures/VoLTE-MT-call.md`, `procedures/session-release.md` |

**Phase 2 order:** Run all 2a chunks first (2a-1 → 2a-5), then all 2b chunks (2b-1 → 2b-4).

---

### Phase 3 — Detail

_Chunk tables to be defined at the start of Phase 3 (requires reading TOC of each spec)._

| Spec | Focus | Approx chunks |
|---|---|---|
| TS 23.218 | S-CSCF call model, iFC, SPTs, AS interaction modes | ~4 chunks |
| TS 23.402 | Non-3GPP access — ePDG, WLAN, S2a/S2b/S2c | ~3 chunks |

---

### Phase 4 — Application Deep Dives (Per-Entity Synthesis)

After phases 1–3, produce one **deep-dive synthesis page** per major network entity.
Each page compiles everything known about that node across all specs:
- All interfaces it terminates (protocol, direction, purpose)
- State machines it owns
- Procedures it initiates or participates in
- Diameter/GTP/SIP messages it sends/receives
- Subscriber data it holds or queries
- Failure and overload behavior
- Key configuration parameters
- Cross-references to procedure and interface pages

Each deep dive is one chunk. Save as `wiki/entities/<NODE>-deepdive.md`.

| ID | Entity | Priority |
|---|---|---|
| 4-1 | MME | Central EPC control plane node |
| 4-2 | PGW | IP anchor, policy enforcement |
| 4-3 | SGW | Mobility anchor, user plane |
| 4-4 | HSS | Subscriber master database (EPC + IMS) |
| 4-5 | PCRF | Policy brain, Gx/Rx anchor |
| 4-6 | P-CSCF | IMS entry point, security anchor |
| 4-7 | S-CSCF | IMS registrar and session router |
| 4-8 | TAS | Telephony feature execution |

---

## Agent Behavior Rules

1. **Always read the schema (this file) at the start of a session** if continuing work
2. **Always read index.md before answering queries** — do not guess at page locations
3. **Never modify files in `raw/`**
4. **Always update index.md and log.md** after any wiki change
5. **Use frontmatter on every page** you create or significantly revise
6. **Ask before filing an analysis** — "Should I save this as a wiki page?"
7. **Prefer updating existing pages over creating new ones** — avoid stub proliferation
8. If a source is a 3GPP spec or technical standard, extract the normative procedures explicitly
9. When in doubt about where something belongs, default to `concepts/`
10. **Never start the next chunk without user confirmation** — always stop after printing the
    end-of-chunk announcement and wait for "continue" before doing any further reading or writing
