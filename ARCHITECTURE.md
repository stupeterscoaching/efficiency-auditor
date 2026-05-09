# Architecture

This document defines the complete system architecture for `efficiency-auditor`. Read this before touching any code.

---

## Overview

`efficiency-auditor` is a pluggable efficiency monitoring module for AI agent systems. Drop it into any agent hierarchy to get passive token usage tracking, cost reporting, model tier recommendations, and post-change validation.

It is designed to be model-agnostic and system-agnostic — it observes all agent traffic passively and never blocks the pipeline.

The module is particularly valuable in systems that use ephemeral worker context windows (inspired by [Late](https://github.com/mlhher/late)'s approach) — since each worker spawns fresh, the Auditor gets clean per-task token data with no cross-task pollution.

Part of the [Bessemer Agentic](https://github.com/usebessemer) open source ecosystem.

---

## Module Hierarchy

```
┌─────────────────────────────────────────────────────┐
│             EFFICIENCY DIRECTOR                     │
│            (mid-tier frontier model)                │
│                                                     │
│  Analyses auditor reports                           │
│  Recommends model tier changes                      │
│  Posts upgrade requests to #approvals               │
│  Monitors post-change quality over time             │
│  Reports findings to host system's Director         │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              EFFICIENCY AUDITOR                     │
│              (local model — passive)                │
│                                                     │
│  Observes all agent traffic                         │
│  Tracks tokens per agent per task                   │
│  Flags anomalies and waste                          │
│  Never blocks pipeline — observe only               │
└─────────────────────────────────────────────────────┘
```

---

## Trust Boundaries

| Action | Authority |
|---|---|
| Recommend any tier change | Efficiency Director |
| Auto-downgrade to cheaper/local model | Efficiency Director |
| Auto-revert failed downgrade to same/lower tier | Efficiency Director |
| Upgrade any agent to paid model | Requires executive approval |
| Revert to paid model | Requires executive approval |
| Org restructuring | Requires executive approval |

**Executive approval flow:** Efficiency Director posts recommendation to `#approvals` Discord channel. Executive types `approve` to confirm or `reject` to decline. System waits before acting.

---

## Post-Change Validation

After any tier change, the Auditor monitors the affected agent for a defined period. The host system's Tech Lead scores every task output during that period. Efficiency Director issues a verdict report — keep, revert, or escalate.

If quality degrades mid-period, escalation happens immediately rather than waiting for the period to end.

Any revert involving a paid model tier requires executive approval via `#approvals`.

---

## Instantiation

```javascript
const { EfficiencyAuditor } = require('@usebessemer/efficiency-auditor');

const auditor = new EfficiencyAuditor({
  currency: "CAD",           // USD | CAD | GBP | EUR
  monitoringPeriod: 7,       // days to monitor after a tier change
  qualityThreshold: 7,       // minimum acceptable quality score (1-10)
  reportInterval: "daily",   // session | daily | weekly
  discordChannel: "#efficiency"
});
```

---

## Contracts

### Efficiency Director <- Auditor (report)

```javascript
payload: {
  period: "session | daily | weekly",
  currency: "USD | CAD | GBP | EUR",
  byAgent: {
    "agent-name": {
      tokensIn: 0,
      tokensOut: 0,
      cost: 0,
      tasksCompleted: 0,
      tasksEscalated: 0,
      avgTokensPerTask: 0
    }
  },
  anomalies: [
    {
      agent: "string",
      type: "high-token-usage | excessive-escalations | low-completion-rate",
      description: "string",
      severity: "low | medium | high"
    }
  ],
  totalCost: 0,
  periodSummary: {
    totalTasks: 0,
    totalEscalations: 0,
    totalInsights: 0
  }
}
```

### Director <- Efficiency Director (recommendation)

```javascript
payload: {
  period: "session | daily | weekly",
  currency: "USD | CAD | GBP | EUR",
  summary: {
    totalCost: 0,
    projectedMonthlyCost: 0,
    potentialSavings: 0
  },
  recommendations: [
    {
      type: "downgrade | upgrade | restructure",
      agent: "string",
      currentModel: "string",
      proposedModel: "string",
      rationale: "string",
      estimatedSaving: 0,
      qualityRisk: "low | medium | high",
      requiresApproval: true
    }
  ],
  validatedChanges: [
    {
      agent: "string",
      change: "string",
      verdict: "correct | incorrect | inconclusive",
      qualityDelta: {
        before: 0,
        after: 0
      },
      action: "keep | revert | escalate"
    }
  ]
}
```

### Director <- Efficiency Director (change validation report)

```javascript
payload: {
  changeId: "unique id linking back to original recommendation",
  agent: "string",
  change: {
    fromModel: "string",
    toModel: "string",
    direction: "upgrade | downgrade",
    implementedAt: "ISO8601"
  },
  monitoringPeriod: {
    start: "ISO8601",
    end: "ISO8601",
    tasksSampled: 0
  },
  qualityTracking: {
    baseline: 0,
    scores: [],
    average: 0,
    trend: "improving | stable | degrading"
  },
  costTracking: {
    before: 0,
    after: 0,
    delta: 0,
    currency: "string"
  },
  verdict: "correct | incorrect | inconclusive",
  action: "keep | revert | escalate",
  requiresApproval: true
}
```

---

## Discord Integration

The module posts to two channels in the host system's Discord server:

- `#efficiency` — periodic reports and recommendations
- `#approvals` — upgrade requests requiring executive approval

---

## Status

**v0.1.0 — Architecture release.** Contracts, trust boundaries, and integration interface are defined. Implementation coming in v0.2.0.

This is an intentional design-first release. The architecture is stable and ready for community contribution.