<div align="center">

# TensileStream

### Deterministic verification for agents that write code.

[![Documentation](https://img.shields.io/badge/docs-tensilestream.github.io-3f6d78.svg)](https://tensilestream.github.io/yieldpoint/)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://github.com/tensilestream/yieldpoint/blob/main/LICENSE)
[![Zero Runtime Deps](https://img.shields.io/badge/runtime%20deps-0-brightgreen.svg)](https://github.com/tensilestream/yieldpoint)
[![Python](https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12%20%7C%203.13-blue.svg)](https://github.com/tensilestream/yieldpoint)

<br>

<p align="center">
  <em>The <strong>yield point</strong> is where a material stops springing back and deforms for good &mdash; the moment it quietly stops being as strong as it was.<br>
  We build structural integrity tools that keep autonomous coding agents from quietly weakening your software.</em>
</p>

</div>

---

### The Problem

Autonomous coding agents (Claude Code, Cursor, Copilot, Codex) are exceptionally good at finding the shortest path to a passing test suite. Often, that path is not fixing the implementation, but silently weakening the assertion:

```diff
- assert invoice.total == Decimal("42.00")
+ assert invoice.total is not None
```

The suite stays green. Coverage is unchanged &mdash; the weaker assertion executes the same lines. Linters report nothing. But the test suite has quietly yielded.

TensileStream builds deterministic, zero-model runtime verification that grades the **transition** &mdash; verifying what was taken away, in single-digit milliseconds, with zero model calls.

---

### 📊 What It Actually Catches

Every figure below is produced by a script in the repository, on a real machine, with no model call. Re-run them yourself &mdash; the commands are in [`benchmarks/ollama/`](https://github.com/tensilestream/yieldpoint/tree/main/benchmarks/ollama).

**Replaying this project's own git history** &mdash; `python benchmarks/ollama/history_proof.py`:

| | |
|---|---|
| commits replayed | 88 |
| wall clock | 11.2 s |
| model calls / tokens | 0 / 0 |
| commits a correctness rule would have stopped | **7** |
| assertions deleted and never replaced | 3, still absent at HEAD |
| tests silently skipped or emptied | 7 |

Real assertions, removed by real people, merged &mdash; and still missing today:

```python
self.assertEqual(_lint_rule(item), 'lint.ruff-format.format')
self.assertIs(verify_diff('', root=self.root, policy=Policy()).status, Status.PASS)
self.assertFalse(self.confirmation.matches('anchor beacon cobalt dynamo'))
```

Point it at your own repository:

```bash
python benchmarks/ollama/history_proof.py --repo /path/to/yours
```

**Against an LLM reviewer** &mdash; both given the same 37 labelled cases (`judge_experiment.py`):

| | Yieldpoint | LLM judge |
|---|---|---|
| accuracy | **100%** (37/37) | 83.8% (31/37) |
| blocked legitimate refactors | **0** of 19 | 5 of 19 |
| median latency | **2.4 ms** | 3.8 s |
| model calls | **0** | 37 |
| tokens | **0** | 11,558 |

The judge blocked one refactor that *strengthened* a test. Being non-deterministic, it would block a different set on the next run &mdash; which is why a probabilistic reviewer cannot be a gate.

---

### 🚧 The Coverage Ceiling

The number that decides whether this is useful to you, measured by `coverage_map.py` across 37 distinct ways a change can weaken a codebase:

| | |
|---|---|
| weakening kinds caught | **6 of 37 — 16%** |
| fully covered | test assertions |
| no coverage at all | concurrency, config, dependencies, error handling, infrastructure, observability, schema, security, source guards, types |

Yieldpoint is narrow on purpose and narrow in fact. It is very good at one thing &mdash; noticing that a test verifies less than it did &mdash; and blind to most others. It is not a replacement for review, types, or a security scanner.

Separately, a scripted run of five shortcuts that all turn a red suite green (`walkthrough.py`) catches **4 of 5**. The missed one is recorded in the results rather than omitted.

---

### ⚖️ What Is Not Yet Measured

An earlier version of this page claimed a 20-task A/B showing roughly twice as many trustworthy answers with verification enabled. **That run is not reproducible from anything committed here, so the claim is withdrawn.**

The A/B benchmark currently in the repository is four tasks against a local model, and both arms score identically &mdash; 4/4 earned, 7,270 tokens each, no difference outside noise. Four tasks is far too small to conclude anything in either direction.

What can be said: the check itself costs **3.68 ms** and zero tokens. Whether it changes how an agent converges is unproven, and the honest answer to "does this make my agent cheaper" is *we do not know yet*.

---

### 📦 Flagship Projects

#### 🔷 [yieldpoint](https://github.com/tensilestream/yieldpoint) &mdash; Deterministic verification for coding agents

- **Attribution, not absolute state:** every finding says whether this change *introduced* it or *inherited* it, with both numbers. `big.py grew from 1,385 to 1,386 lines of code (+1). It was already over the limit before this change.` Existing debt is reported, never blamed on the edit in front of it.
- **Four doors:** per-edit hooks (Claude Code / IDE), a Stop gate that catches shell and batch edits, git pre-commit, and CI.
- **Zero runtime dependencies:** never touches the network, never invokes an LLM, never conflicts with your project dependencies.
- **Fast:** AST transition analysis in single-digit milliseconds via a content-addressed index.
- **Ecosystem:** LangGraph, Model Context Protocol, Claude Code hooks, pre-commit, GitHub Action, SARIF, and an LSP diagnostics server.

```bash
pip install yieldpoint
yieldpoint init
```

`init` measures your repository before it proposes a limit, and says what it measured:

```
249 Python files; median 112 lines, p90 240, longest 307.
A limit of 250 leaves 7% of this repository over it, and 300 would leave 0%.
```

**26 rules**, every one documented at [the rules reference](https://tensilestream.github.io/yieldpoint/rules.html); a build cannot ship a rule the site does not describe.

---

### 🔎 Commands

| | |
|---|---|
| `yieldpoint review` | what this change did, attributed. `--sarif` for GitHub/GitLab, `--json` for anything else |
| `yieldpoint summary` | the same thing as a pull-request comment, introduced separated from inherited |
| `yieldpoint policy` | every setting this build accepts: type, legal values, value in force, and where it came from |
| `yieldpoint languages` | what share of *your* repository this build can analyse |
| `yieldpoint allows` | every acknowledgement in the tree, with age, owner and expiry |
| `yieldpoint task` | declare what this piece of work may touch; edits outside it are reported |
| `yieldpoint trend` | what it has found week by week, introduced against inherited |
| `yieldpoint adoption` | how soon it first found something, and what happened to findings afterwards |
| `yieldpoint backtest` | replay your own history and see what would have been flagged, before installing anything |
| `yieldpoint lsp` | diagnostics in your editor, compared against the committed file |

---

### 🔄 Where It Fits in the Loop

```text
      Task
       │
       ▼
  ╔═══════════════════════╗  tier: human
  ║ 1. BEFORE THE MODEL   ║──────────────────────▶ Ask a person
  ║    yieldpoint.harness ║
  ║    how much model     ║──────────────────────┐ tier: none
  ║    does this need?    ║                      │ (no model call)
  ╚═══════════════════════╝                      │
       │ tier: small | capable                   │
       ▼                                         │
  ┌───────────────────┐                          │
  │  Call the model   │◀──────────┐              │
  └───────────────────┘           │              │
       │                          │ deny +       │
       ▼                          │ prescription │
  ╔═══════════════════════╗       │              │
  ║ 2. BEFORE THE TOOL    ║───────┘              │
  ║    hook / middleware  ║                      │
  ║    does this edit     ║                      │
  ║    weaken the suite?  ║                      │
  ╚═══════════════════════╝                      │
       │ allow                                   │
       ▼                                         │
  ┌───────────────────┐                          │
  │  Apply the edit   │◀─────────────────────────┘
  └───────────────────┘
       │
       ▼
  ╔═══════════════════════╗  findings
  ║ 3. BEFORE THE MERGE   ║────────────▶ back to the model
  ║    CI / pre-commit    ║
  ║    the whole change   ║────────────▶ Merge
  ╚═══════════════════════╝  clean
```

---

### 🌍 Languages

Generated by `python scripts/language_matrix.py`, not written by hand. A language counts as read only if a real weakening is reported **and** a real legitimate edit is not.

| Language | Frameworks read | Weakening caught | Quiet on legitimate | Can gate a commit |
|---|---|---|---|---|
| Python | pytest / unittest | yes | yes | **yes** |
| JavaScript | Jest / Vitest / Mocha | yes | yes | no — warns only |
| TypeScript | Jest / Vitest | yes | yes | no — warns only |
| Java | JUnit / AssertJ | yes | yes | no — warns only |
| Kotlin | JUnit | yes | yes | no — warns only |
| Go | stdlib guards / testify | yes | yes | no — warns only |
| Rust | built-in test harness | yes | yes | no — warns only |
| C# | xUnit / NUnit / MSTest | yes | yes | no — warns only |
| Ruby | Minitest | yes | yes | no — warns only |
| PHP | PHPUnit | yes | yes | no — warns only |
| Swift | XCTest | yes | yes | no — warns only |
| Elixir | ExUnit | yes | yes | no — warns only |

**Exact** analysis parses the file and may block a commit. **Lexical** analysis matches shapes with regular expressions &mdash; it can tell an agent it weakened a test, but never stops a build. That is the honest ceiling for a package with zero runtime dependencies: a parser per language is a dependency per language.

**One exception, opt-in.** `pip install yieldpoint[typescript]` adds a real parser for TypeScript and JavaScript, which brings the structural rules &mdash; file length, function length, parameters, nesting, complexity &mdash; to those files and lets them gate. Assertion analysis there stays lexical. Without the extra installed, those files are reported as *not evaluated*, never as clean.

Run `yieldpoint languages` to see what share of your own tree any of this reaches.

---

### 🛡️ Core Guarantees

1. **Deterministic & Mechanical:** the same change yields the same verdict on every machine, every time. No rule reads a clock. An acknowledgement may carry an expiry, and that expiry never changes a verdict &mdash; only `yieldpoint allows --expired`, which says it is reading the clock, acts on it.
2. **No Models in the Verification Core:** we never ask an LLM to evaluate an LLM. Verification must be unforgeable and instantaneous.
3. **Nothing Unexamined Reports as Clean:** a file no analyser claims is reported *not evaluated*. A change with no baseline to compare against says so rather than guessing. A protected test file in a language whose assertions are not read is never recorded as checked.
4. **Auditable Ledger, Honestly Labelled:** `yieldpoint stats` separates what was *measured* from what is *architectural* from what is *estimated*, and names what it does not claim.
5. **The Rules Are Themselves Under Review:** raising a limit, disabling a rule or widening an exclusion is reported as `policy_weakened`. It never blocks &mdash; a project must be able to change its own standards &mdash; but it is never silent either.

---

### 🌐 Ecosystem & Resources

- 📖 **Documentation Site:** [tensilestream.github.io/yieldpoint](https://tensilestream.github.io/yieldpoint/)
- ⚙️ **Rules Reference:** [Engine Rules & Invariants](https://tensilestream.github.io/yieldpoint/rules.html)
- 🤝 **Contributing:** [Contribution Guidelines](https://github.com/tensilestream/yieldpoint/blob/main/CONTRIBUTING.md)
- 🔒 **Security Disclosures:** [Security Advisories](https://github.com/tensilestream/yieldpoint/security)

---

<div align="center">
  <sub>TensileStream &copy; 2026. Built with precision for the agentic engineering era.</sub>
</div>
