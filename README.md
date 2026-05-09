# efficiency-auditor

A pluggable token usage monitoring and cost optimisation module for AI agent systems.

Drop it into any tiered agent hierarchy to get passive token tracking, cost reporting, model tier recommendations, and post-change quality validation — without touching your pipeline logic.

Part of the [Bessemer Agentic](https://github.com/usebessemer) open source ecosystem.

---

## What it does

```
Your agent system runs normally
        ↓
Efficiency Auditor observes all traffic passively
        ↓
Efficiency Director analyses usage patterns
        ↓
Recommendations posted to #efficiency
        ↓
Downgrades execute automatically
Upgrades require executive approval via #approvals
        ↓
Post-change quality monitored for N days
        ↓
Verdict: keep, revert, or escalate
```

The module never blocks the pipeline. It observes only.

---

## Status

**v0.1.0 — Architecture release.**

The contracts, trust boundaries, and integration interface are fully defined. Implementation is coming in v0.2.0.

This is an intentional design-first release. If you want to contribute the implementation, read [ARCHITECTURE.md](./ARCHITECTURE.md) and open a PR.

---

## Architecture

See [ARCHITECTURE.md](./ARCHITECTURE.md) for:
- Full module hierarchy
- Trust boundaries and approval flows
- Post-change validation logic
- Complete message contracts
- Discord integration

---

## Designed for ephemeral agent systems

This module is particularly effective in systems using ephemeral worker context windows (inspired by [Late](https://github.com/mlhher/late)). Since each worker spawns fresh with no cross-task context, the Auditor gets clean per-task token data — no pollution, no ambiguity.

It pairs naturally with [agent-dev-team](https://github.com/usebessemer/agent-dev-team).

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). Implementation contributions for v0.2.0 are welcome.

---

## License

[MIT](./LICENSE) — Bessemer Agentic 2026