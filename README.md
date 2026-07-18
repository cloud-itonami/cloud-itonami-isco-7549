# cloud-itonami-isco-7549

Open Occupation Blueprint for **ISCO-08 7549**: Craft and Related Workers Not Elsewhere Classified.

This repository designs a forkable OSS business for a craft workshop scheduling and logistics coordination practice: a workshop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so a craft workshop crew keeps its own operating records instead of renting a closed workforce-management SaaS.

ISCO-08 7549 is a broad, generic residual "Not Elsewhere Classified" category covering diverse craft work (varied hand-tools, varied materials) not captured by a more specific ISCO craft code. Standard workshop safety hazards apply generically (hand-tool injury, varied-material handling) without a single dominant hazard type, so this actor's scope and hazard reporting stay deliberately generic rather than technique-specific. This batch completes the ISCO-08 75xx craft-trades range.

**Maturity: `:implemented`.** `src/craftnec/` implements the
`CraftNecActor` as a `langgraph.graph/state-graph`
(`craftnec.actor`) wired to a `Craft Trades Coordination Advisor`
(`craftnec.advisor`) and an independent `CraftNecGovernor`
(`craftnec.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt)
+-> :hold (:hard?)`. 24 tests / 52 assertions green (`clojure -M:test`).
HARD invariants (always hold, never
overridable): worker provenance, workshop provenance, no-actuation
(`:effect` must be `:propose`), a closed op-allowlist
(`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize a craft-execution decision
(e.g. deciding a craft item is finished) or a
product-quality/safety-clearance decision (e.g. declaring an item
quality- or safety-cleared), or override a shop safety officer's
judgment. Always-escalate paths (human sign-off regardless of
confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics coordination robot performs crew scheduling, job/commission/progress-record logging and craft-materials supply-order coordination for a craft workshop crew, under an actor that proposes actions and an independent **Craft Trades Coordination Governor** that gates them. The governor never
dispatches hardware itself, never performs craft work on the shop floor, and never finalizes a craft-execution decision or a product-quality/safety-clearance decision, nor overrides a shop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged hand-tool-hazard/material-handling/equipment-condition concern, or an above-threshold supply order) require human sign-off. **This actor coordinates workshop scheduling/logistics only — it never performs craft work or makes quality/safety-clearance decisions itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Craft Trades Coordination Advisor -> CraftNecGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
a craft-execution decision, declare a product quality- or safety-clearance,
override a shop safety officer's judgment, suppress an operating record, or
disclose sensitive data without governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7549`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
