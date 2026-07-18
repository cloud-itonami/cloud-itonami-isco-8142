# cloud-itonami-isco-8142

Open Occupation Blueprint for **ISCO-08 8142**: Plastic Products Machine
Operators.

This repository designs a forkable OSS business for an injection-molding/
extrusion plant scheduling and logistics coordination practice: a plant
scheduling and supply-coordination robot manages crew/task records under a
governor-gated actor, so a plastic products crew keeps its own operating
records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/plasticcoord/` implements the
`PlasticCoordActor` as a `langgraph.graph/state-graph`
(`plasticcoord.actor`) wired to a `Plastic Plant Scheduling Coordination
Advisor` (`plasticcoord.advisor`) and an independent `PlasticCoordGovernor`
(`plasticcoord.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok? true) +-> :request-approval (:escalate? true, human-in-the-loop
interrupt) +-> :hold (:hard? true)`. HARD invariants (always hold, never
overridable): molder provenance, facility provenance, no-actuation
(`:effect` must be `:propose`), a closed op-allowlist (`:log-work-record`,
`:schedule-crew-operation`, `:flag-safety-concern`,
`:coordinate-supply-order` — nothing else may ever be proposed), and a
permanent, unconditional block on any proposal that would directly finalize
a molding-operation-execution decision (e.g. deciding to proceed with a
specific injection-molding or extrusion run) or a plant-safety-clearance
decision (e.g. declaring a plant or a molded batch safe for handling), or
that would override a plant safety officer's judgment. Always-escalate
paths (human sign-off regardless of confidence, mapping this repo's Trust
Controls in [`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above the
registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a plant scheduling/logistics coordination
robot performs crew scheduling, production-run/inventory/progress-record
logging and raw-plastic/resin supply-order coordination for a plastic
products (injection-molding/extrusion) crew, under an actor that proposes
actions and an independent **Plastic Plant Scheduling Coordination
Governor** that gates them. The governor never dispatches hardware itself,
never operates injection-molding or extrusion equipment on the plant floor,
and never finalizes a molding-operation-execution decision or a
plant-safety-clearance decision, and never overrides a plant safety
officer's judgment; `:high`/`:safety-critical` actions (such as a flagged
machinery-hazard/heat-exposure/fume-exposure/equipment-condition concern, or
an above-threshold supply order) require human sign-off. **This actor
coordinates PLANT SCHEDULING/LOGISTICS ONLY — it never operates
injection-molding or extrusion equipment itself, and it never makes a
plant-safety-clearance decision itself.**

Plastic Products Machine Operators run injection-molding and extrusion
equipment — a heavy-machinery hazard domain (crush/entanglement risk from
presses), alongside heat exposure and fume exposure (plastic off-gassing
during processing). This is a real industrial-injury and exposure domain;
this actor never operates that equipment and never clears it as safe — it
only schedules and logs around it, and always routes machinery/heat/fume
safety concerns to a human plant safety officer.

## Core Contract

```text
crew roster + facility registration + safety-reporting policy
        |
        v
Plastic Plant Scheduling Coordination Advisor -> PlasticCoordGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a molding-operation-execution decision, finalize a
plant-safety-clearance decision, override a plant safety officer's
judgment, suppress an operating record, or disclose sensitive data without
governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `8142`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
