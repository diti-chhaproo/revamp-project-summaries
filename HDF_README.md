# Revamp — HDF5 AI Sustainability Pipeline

> AI-assisted code sustainability tooling for the HDF5 C library.  
> Built by Revamp Engineering (ACFI) in collaboration with [HDF Group](https://www.hdfgroup.org/).

---

## What This Does

Revamp is a verification-gated pipeline that uses LLMs to generate, verify, and deliver complexity-reducing refactoring diffs for the HDF5 C library. In **generate** mode, every candidate diff must pass the full 6-gate stack before it can be submitted as a PR. A separate **score-existing-PR** mode scores a human GitHub PR against a reduced gate set (see [Modes](#modes)).

**Design guarantee (generate mode):** The worst case is that the tool produces nothing. It cannot make the codebase worse — only improved or untouched.

## Modes

| Mode | Entry point | Purpose |
|------|-------------|---------|
| **Generate** | `orchestrator.py` / `verify_e2e.py` | LLM produces a candidate refactoring; full 6-gate stack (apply → dual_llm → build → static → memory → test/score) |
| **Score-existing-PR** | `verify_live_pr.py` | Score a real GitHub PR (`HDFGroup/hdf5`) against build → static → memory → test; optional `human_review` labeled separately |

### Score-existing-PR (quick start)

```bash
# Preflight only (token + rate budget + base_sha)
GITHUB_TOKEN=... python3 verify_live_pr.py 6470 --preflight-only --repo-path /path/to/hdf5

# Full score run with atomic JSON (ScoreHarnessResult.to_dict())
GITHUB_TOKEN=... python3 verify_live_pr.py 6470 \
  --repo-path /path/to/hdf5 \
  --json reports/pr_6470_score.json

# Cap scored functions (remainder UNSCORED; M still = all mapped C identities)
GITHUB_TOKEN=... python3 verify_live_pr.py 6470 --repo-path /path/to/hdf5 --max-functions 10

# Skip optional LLM review of the human diff
GITHUB_TOKEN=... python3 verify_live_pr.py 6470 --repo-path /path/to/hdf5 --no-human-review
```

Orchestration is `run_score_harness` → `ScoreHarnessResult`. Coverage is reported as **`scored K/M`** (`scored_k` / `scored_m`): `K` = functions actually scored, `M` = all mapped C function identities (resolvable + unresolved). Capped functions are `UNSCORED` but still count in `M`. Score-mode never claims generate-mode “pipeline approved”; clearance wording is **“human diff cleared verification gates”** (and/or **“human diff reviewed by dual-LLM”** for the optional review).

`--workspace-root` is a **parent** directory only (`allocate_owned_workspace` creates a unique child and cleans that child; the parent is never deleted). Exit codes: `0` OK, `6` HARNESS_ERROR (bad paths/config/interrupt), `10`–`18` for token/rate/diff/repo/base/fetch/apply/materialize failures — see `python3 verify_live_pr.py --help`.

## Quick Start

```bash
git clone https://github.com/<your-org>/revamp.git
cd revamp

# Install Python dependencies
pip install -r requirements.txt

# Copy and edit config
cp config.yaml config.local.yaml
# Set your API keys and sandbox repo path in config.local.yaml

# Run the pipeline
python orchestrator.py
```

---

## Architecture

The pipeline has 5 layers, each in its own directory:

```
revamp/
├── orchestrator.py          # Generate-mode entry — coordinates all layers
├── config.yaml              # Central configuration
├── verify_e2e.py            # Generate-mode end-to-end gate runner
├── verify_live_pr.py        # Score-existing-PR harness (run_score_harness)
├── live_pr/                 # Score-mode helpers (fetch, map, gates, review)
│   ├── github_pr.py         # PR fetch + rate-limit preflight
│   ├── materialize.py       # Apply-once worktrees
│   ├── hunk_map.py          # Hunk → touched-function mapping
│   ├── score_gates.py       # build→static→memory→test once + GATE_SCOPES
│   ├── attribution.py       # Finding attribution + SCORED_* taxonomy
│   └── human_review.py      # Optional human_review (never labeled dual_llm)
│
├── indexing/                # Layer 1: Parse + embed HDF5 C source
│   ├── tree_sitter_parser.py
│   ├── embedder.py
│   └── chroma_store.py      # Style Knowledge Base (ChromaDB)
│
├── analysis/                # Layer 2: Static + security + memory analysis
│   ├── complexity.py        # Lizard CCN
│   ├── static_analysis.py   # cppcheck + clang-tidy
│   ├── security.py          # CodeQL
│   └── memory_safety.py     # AddressSanitizer
│
├── llm/                     # Layer 3: LLM refactoring + verification
│   ├── primary.py           # Claude Sonnet — generates diffs
│   ├── verifier.py          # GPT-4o — independent review
│   ├── rag.py               # RAG context retrieval
│   └── prompts/             # System prompt templates
│
├── gates/                   # Layer 4: Binary verification gates
│   ├── apply.py             # Diff application + rollback
│   ├── build_gate.py        # Gate 1: CMake build
│   ├── static_gate.py       # Gate 2: Static analysis
│   ├── dual_llm_gate.py     # Gate 3: Dual-LLM approval (generate mode)
│   ├── memory_gate.py       # Gate 4: Memory safety (ASan)
│   ├── test_gate.py         # Gate 5: CTest regression
│   └── scorer.py            # Gate 6: CCN improvement check (generate mode)
│
├── ci/                      # Layer 5: CI/CD configurations
│   ├── github_actions/      # Workflow YAMLs
│   └── docker/              # Container definitions
│
├── reports/                 # Generated outputs
│   ├── complexity_timeline/ # Interactive HTML dashboard (Week 1 deliverable)
│   ├── diff_queue/          # Diffs awaiting human review
│   └── evaluation/          # Pipeline accuracy metrics
│
└── docs/
    └── architecture.md      # Full architecture deep-dive
    └── ci_integration_plan.md  # Gate vs. HDF5 CI integration decisions
```

---

## Verification Gates

### Generate mode

All 6 gates must pass. Any failure silently discards the candidate diff.

| # | Gate | Tool | Blocks on |
|---|------|------|-----------|
| 1 | Build | CMake + Ninja | Compilation failure |
| 2 | Static | cppcheck + clang-tidy | New warnings |
| 3 | Dual-LLM | Claude Sonnet + GPT-4o | Either LLM rejects |
| 4 | Memory | AddressSanitizer | New memory errors |
| 5 | Test | CTest (serial) | Test regressions |
| 6 | Scorer | Lizard CCN delta | No complexity improvement |

### Score-existing-PR mode

Skips generate-mode **apply** and **dual_llm**. Applies the PR’s own patch once as setup, then runs **build → static → memory → test** once on the patched tree. Optional **`human_review`** (config `gates.human_review`; disable with `--no-human-review`) is labeled `human_review` everywhere — never `dual_llm` — and does not block verification dispositions.

| Gate | In score mode? | Scope (`GATE_SCOPES`) |
|------|----------------|------------------------|
| apply (gen) | No (PR patch applied once as setup) | — |
| dual_llm | No | — |
| build | Yes | whole-tree |
| static | Yes | file-scoped |
| memory | Yes | whole-tree |
| test | Yes | mixed |
| scorer | No | — |
| human_review | Optional (separate label) | per-function LLM review of human diff |

---

## Team & Role Ownership

| Name | Role | Owns |
|------|------|------|
| **Abhinav** | Pipeline & CI Lead / Scrum Lead | `orchestrator.py`, `verify_e2e.py`, `verify_live_pr.py`, `live_pr/`, `/gates`, `/ci`, `/reports` |
| **Diti** | Project Manager / PRD Owner | Client comms, PRD, sprint planning |
| **Firdavs** | Indexing & RAG Lead | `/indexing` |
| **Sydney** | Static Analysis & Safety Lead | `/analysis` |
| **Basil** | LLM Engine Lead | `/llm` |
| **William** | Pipeline Support | `/gates`, `/ci` (under Abhinav) |

**HDF Group contacts:**
- Scot Breitenfeld — Director of Engineering (monitoring, PR reviewer)
- Gerd Heber — Executive Director (stakeholder)

---

## Scope Constraints

| Allowed | Not Allowed |
|---------|-------------|
| Refactoring existing C functions | New public API or new exports |
| Test de-duplication | New files |
| Complexity reduction | C++ / Fortran / Java changes |
| Static analysis improvements | Subsystem rewrites |
| Delivering diffs as PRs | Direct commits to upstream |

---

## Sandbox Repository

All development happens against the HDF5 sandbox clone:  
`https://github.com/sp26-hdfgroup/hdf5-sandbox`

Changes are delivered to the main HDF5 repo via PR:  
`https://github.com/HDFGroup/hdf5`

---

## Contact

- Diti (PM): djc11@illinois.edu
- Abhinav (Tech Lead): ag135@illinois.edu
