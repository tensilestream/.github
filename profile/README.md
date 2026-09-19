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

### 📦 Flagship Projects

#### 🔷 [yieldpoint](https://github.com/tensilestream/yieldpoint) &mdash; Deterministic verification for coding agents

- **Three-door defense:** Per-edit hooks (Claude Code / IDE), Stop gate (shell and batch edits), and Git pre-commit / CI backstop.
- **Zero runtime dependencies:** Never touches the network, never invokes an LLM, never conflicts with your project dependencies.
- **Fast:** Instant AST transition analysis; sub-10ms evaluation via a content-addressed index.
- **Ecosystem ready:** Native support for LangGraph, Model Context Protocol (MCP), Claude Code hooks, pre-commit, and CI workflows.

```bash
pip install yieldpoint
yieldpoint init
```

---

### 🔄 Where It Fits in the Loop

TensileStream verification sits directly inside the agent execution stream across three critical insertion points:

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

### 🛡️ Core Guarantees

1. **Deterministic & Mechanical:** The same change yields the exact same verdict on every machine, every time.
2. **No Models in the Verification Core:** We never ask an LLM to evaluate an LLM. Verification must be unforgeable and instantaneous.
3. **Auditable Savings Ledger:** `yp stats` tracks exactly how many unnecessary model calls, tokens, and regressions were prevented locally.

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
