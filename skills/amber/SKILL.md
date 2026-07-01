---
name: amber
description: >-
  Use this skill whenever you are asked to build, design, or generate a data-processing tool (classifier, extractor, redactor, scorer, validator) that must run on regulated, sensitive, or private data WITHOUT that data ever reaching a frontier/cloud model — banking records, patient files, PII, claims, legal docs, anything under GDPR/HIPAA/NIS2. Trigger it EVEN for a direct "build me the tool" imperative you could code yourself: consult it first, because a naive build silently omits the seal it exists to provide — the deterministic tier-1/tier-2 split, the timestamp-free manifest, and the no-frontier-model-at-runtime guarantee. Also trigger for framings like "the AI designs the tool but never sees the data," "no model at runtime," "air-gapped / on-prem / inside our boundary," "auditable / deterministic," the Amber method by name (Forge, Enclave, Machine, Trusted AI, sealed, frozen, manifest), or a spec plus synthetic fixtures to build from. This IS the Forge procedure for constructing an Amber Machine.
---

# Amber — building a Machine in the Forge

You are the frontier model, working in **the Forge**. Your job is to construct a
**Machine**: a finished, deterministic, auditable data-processing tool that the
customer will run in their own **Enclave**, on real data, with you absent by
construction.

You never see the customer's real data. You work only from a **spec** (the shape
of the problem) and **synthetic fixtures** (safe, manufactured stand-in data).
From those, and only those, you produce the Machine — then you are gone. The
capability is preserved in the Machine; you are not in its runtime.

This skill is the build procedure. It is prose-driven on purpose: there is no
prefab engine to fill in. You construct the Machine from the method each time.
That freedom is also the risk — two builds that drift apart quietly break the
guarantee below. The gates in this skill exist to hold the method steady across
builds.

The vocabulary (Forge, Enclave, Machine, Engine, Policy, Trusted AI, Judge,
Capability, spec, synthetic fixtures, manifest, receipt) is defined precisely in
[references/glossary.md](references/glossary.md). Read it if any term feels
underspecified — the method reads as vaguer than it is when the terms are guessed
at rather than used as defined.

## The one guarantee everything serves

**The customer's real data never reaches the model that designed the Machine.**
Capability and data are separated in time and never meet. Every decision you make
in the Forge serves this. It expresses as three properties a regulated buyer
actually asks for:

- **Frozen** — *does it drift?* The Machine's behavior is fixed at build time. No
  retraining, no self-update, no call-home. Same input, same output, on the day
  it ships and a year later. What was tested is what runs.
- **Transparent** — *can I audit it?* The Machine is readable code, not an opaque
  model. An auditor can hold it to the light and see every branch and case. Under
  regimes like NIS2 and the Cyber Resilience Act, the person who signs must be
  able to see and show what the system does.
- **Sealed** — *where does my data go?* The frontier model is simply not on the
  runtime data path. There is nothing to trust because there is nothing there.
  Not a "we don't train on your data" promise — a property of the architecture.

If a choice would weaken one of these, it is the wrong choice. Say so and pick
again.

## The execution ladder

Inside the Machine, work is resolved in this order, and never out of it:

1. **Deterministic code first.** Most of any real pipeline is fixable logic —
   rules, transforms, decisions you can write down once and freeze. This is the
   elegant half. Push everything you can down to this rung.
2. **A Trusted AI only where genuinely needed.** Where a task needs judgment that
   truly cannot be reduced to fixed logic, the Machine may call a self-hosted,
   open-weights **Trusted AI** inside the Enclave, on data that never leaves. This
   is **optional and off by default** — a Machine with no Trusted AI is the normal
   case, not a degraded one.
3. **The frontier model never.** You are not given an address to call at runtime.
   The Machine cannot reach you. This is what makes Sealed structural.

Each rung keeps Sealed true. Climb only when the rung below genuinely cannot do
the job, and be able to prove it can't (see the complementarity check).

## The build procedure

Work these gates in order. Each is a checkpoint, not a suggestion — a later gate
assumes the earlier ones passed. If a gate fails, stop and resolve it before
continuing; don't paper over it downstream.

### Gate 1 — Is this an Amber problem?

Amber fits problems whose **judgment can be fixed at build time**. Before building
anything, decompose the spec: separate the parts that are fixable logic from the
residue that genuinely needs judgment on live data.

- If the whole problem reduces to deterministic logic (plus, at most, a bounded
  Trusted AI for the residue) — it is an Amber problem. Proceed.
- If the core of the problem requires open-ended frontier reasoning on the real
  data at runtime — reasoning beyond what a self-hosted Trusted AI can do and
  irreducible to fixed logic — **it is not an Amber problem.** There is no trick
  that gives frontier reasoning on real data at runtime without disclosing real
  data at runtime. Say this plainly and stop. Do not pretend otherwise; the honest
  line matters more than the pitch.

The line usually cuts *through the middle* of a pipeline, not around it. A
pipeline is rarely one monolithic act of judgment — it is mostly fixable logic
with a few hard spots. Your job is decomposition: freeze everything you can into
deterministic code, isolate the genuine residue, and cover only that with a
Trusted AI. State the decomposition explicitly as the first output — what goes to
tier 1, what (if anything) goes to tier 2, and why the tier-2 parts cannot be
frozen.

### Gate 2 — Confirm the inputs, and refuse real data

You build from exactly two things:

- **The spec** — what comes in, what must come out, which cases are hard, which
  mistakes are unacceptable. The customer does not need to write code to bring a
  spec.
- **Synthetic fixtures** — manufactured data that carries the structure and edge
  cases of the real thing but nothing real.

If the fixtures you are handed look like they might be real data — real names,
real account numbers, anything that reads as genuine PII rather than constructed —
**stop and flag it.** The Forge is untrusted by design; real data must never enter
it. This is the build-time half of the seal. In a naive build it rests on
discipline, so *your* discipline is load-bearing: name the risk, don't process
suspected real data, ask for synthetic fixtures instead.

If the fixtures don't cover the hard cases the spec names, ask for more before
building — a Machine is only as faithful as the structure its fixtures carry.

### Gate 3 — Choose the policy language (a declared parameter)

The Machine is made of two separable parts:

- **Engine** — the stable, generic machinery. Arbitrarily complex; **audited once**
  by engineers. This is where hard complexity is allowed to live.
- **Policy** — the volatile, domain-specific ruleset: the thresholds and rules
  particular to this problem. Changes often; **re-audited every cycle** by the
  domain experts who own it.

This seam is what makes "auditable" operational: a reviewer re-reads the small
Policy each cycle, never the whole Engine. So the Policy must be readable *by the
people who audit it*, who are often not engineers. Decide, and state, how Policy
is expressed:

- **Constrained, inspectable DSL / declarative rules** (recommended default when
  domain experts audit). A small rule language a non-engineer can read is worth
  more than raw code they can't. Prefer this unless told otherwise.
- **Sandboxed expression evaluation** (e.g. a restricted set of Python
  expressions). More expressive, but sits in tension with "auditable by a
  non-engineer." Choose it only when the auditors are themselves technical, and
  say so.

Choose for the people who must *audit* the Policy, not only those who write it.
This is a design choice — state it; don't leave it implicit.

### Gate 4 — Build the Machine, deterministic first

Construct the Engine and Policy so that everything the ladder can push to tier 1
*is* tier 1. The deterministic path must be **bit-reproducible**: same input, same
output, no reliance on wall-clock, ordering of unordered inputs, locale, or any
ambient nondeterminism. Frozen depends on this.

Keep the two parts genuinely separate. If domain logic leaks into the Engine, the
"audit the Policy every cycle, the Engine once" promise stops being true.

### Gate 5 — Wire a Trusted AI only if the spec calls for it, under contract

Only if Gate 1 identified a genuine residue does the Machine talk to a model at
runtime — a **Trusted AI**, self-hosted in the Enclave. This is the one tier that
can break the method's guarantees if left informal, so it is the one tier with a
contract. **Before wiring anything, read
[references/tier-2-contract.md](references/tier-2-contract.md) in full** and make
the call conform to it. In short, a conforming call:

1. **Cannot silently change a deterministic outcome.** Its verdict is advisory and
   receipted; the tier-1 output is invariant to whether the model is present at
   all.
2. **Degrades by skipping, not failing.** Unreachable endpoint, timeout, blown
   budget, unparseable output → the rule skips cleanly and records that it
   skipped. A missing model produces a smaller audit, not a broken run.
3. **Emits a receipt** — verdict, one-line reason, model + version — as durable
   output. A model embedded in readable code is still a black box unless it
   explains itself in what it emits.
4. **Talks only to the one declared local endpoint**, adding no other network
   egress. This is what makes "inside the boundary" checkable rather than
   asserted.

Decide and state the **decision-vs-flag** fork: is the verdict allowed to act
autonomously, or does it only produce a receipted flag a human adjudicates? For an
audit/compliance posture, flag-for-a-human is usually right — it keeps every
consequential decision with an accountable person. Whichever you pick, state it.

**Complementarity check (required if tier 2 exists).** Tier 2 earns its place only
if it catches something tier 1 cannot. Construct and include at least one concrete
case that passes *every* deterministic check yet is resolved only by the model's
judgment. If you cannot construct that case, the Trusted AI is decoration — remove
it and resolve the problem in tier 1.

The two roles of a Trusted AI (**Judge** — sits outside the logic and comments;
**Capability** — wired into the Machine's own logic) are defined in the glossary.
Both are optional. Use the one the spec's residue actually calls for.

### Gate 6 — Emit the manifest

"Frozen" is not verifiable until you can prove what the Machine *is*. Every build
emits a **manifest** — the receipt that turns Frozen and Transparent from claims
into things an auditor checks with `diff`. Emit it with exactly these contents:

- `engine_hash` — content hash (e.g. SHA-256) of the Engine source.
- `policy_hash` — content hash of the Policy source.
- `spec_hash` — content hash of the originating spec.
- `dependencies` — every dependency pinned to an exact version.
- `tier2` — present only if a Trusted AI is wired: the declared endpoint contract
  and the model/version it expects. Absent otherwise.

**Deliberately no timestamp, no build host, no random seed, nothing ambient.** Two
builds of the same inputs must be *byte-identical* and diffable. A changed Policy
must show up as one changed hash and nothing else moving. If anything in the
manifest varies between two identical-input builds, the build is not Frozen — find
the leak and remove it.

### Gate 7 — Verify the seal before you hand it over

The seal is checkable, not taken on faith. Before declaring the Machine done,
confirm and report:

- **No path to the frontier model.** The Machine is never given an address to call
  you. Grep your own output for it — there must be nothing there.
- **No network egress** except, if tier 2 exists, the one declared local endpoint.
- **Bit-reproducible tier 1** on the synthetic fixtures.
- **Complementarity case present** if and only if tier 2 exists.
- **Manifest is timestamp-free** and reproduces identically on a rebuild.

Report each as a checked fact, not an assertion. "The absence of the frontier
model on the data path is a thing you can check, not a thing you are told" — so
show the check.

## What you deliver

A complete Machine the customer can drop into their Enclave:

1. **The decomposition** — tier-1 vs tier-2 split, with the reasoning (from Gate 1).
2. **The Engine** — stable machinery, readable, auditable.
3. **The Policy** — in the language chosen at Gate 3, readable by its auditors.
4. **The Trusted AI adapter** — only if the spec called for one, conforming to the
   contract, with the complementarity case (Gate 5).
5. **The manifest** — timestamp-free, per Gate 6.
6. **A seal report** — the Gate 7 checks, each shown rather than claimed.

Everything readable, everything frozen, nothing that can reach you at runtime.

## Where Amber breaks — refuse honestly

Amber is not a universal solvent. Do not stretch it to fit a problem it doesn't.

- **Irreducible live judgment.** If the problem needs frontier-level reasoning on
  real data at runtime, it is out of scope. Say so at Gate 1 and stop.
- **Undecomposable problems.** If the problem cannot be split into a frozen machine
  plus a bounded Trusted AI, it is not an Amber problem. You will usually know
  early.
- **Suspected real data in the Forge.** Stop and ask for synthetic fixtures (Gate
  2).
- **A Trusted AI that earns nothing.** If you cannot build the complementarity
  case, remove the tier (Gate 5).

The honest line is part of the method. A clear "this isn't an Amber problem, and
here's why" is a correct outcome, not a failure.
