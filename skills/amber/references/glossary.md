# Glossary

Amber has a small number of load-bearing terms. They recur, and if their meaning
has to be reconstructed each time, the method reads as vaguer than it is. This is
the canonical reference. The [SKILL.md](../SKILL.md) uses these terms as defined
here.

> Naming still settling: **the Forge** / **the Enclave** (the two environments)
> and **Trusted AI** (formerly "local model") are working labels — swap them
> freely, the definitions are what matter.

---

## The two AIs

The method uses AI in exactly two places, and they could not be more different in
trust or in role.

### Frontier model
The strong, general model that **constructs the Machine** at build time, working
in [the Forge](#the-forge-build-environment) from a [spec](#spec) and
[synthetic fixtures](#synthetic-fixtures) alone. It is *strong but untrusted*: it
is used precisely because it can design processing no in-house team could match,
but it is a foreign cloud model that must never see real data. It works only at
build time and is **absent at runtime by construction** — the Machine is never
given an address to call it.

### Trusted AI
An **open-weights model, self-hosted inside [the Enclave](#the-enclave-runtime-environment)**,
that may run *at runtime*, under the operator's control, on real data that never
leaves. It is the only model present when real data is being processed. Where the
frontier model is strong-but-untrusted, the Trusted AI is the inverse:
weaker, but trusted because you host it. It has two possible roles — **Judge** and
**Capability** — and both are **optional and off by default.** A Machine with no
Trusted AI at all is the normal case, not a degraded one.

> *Formerly "the local model." "Trusted" names the property that matters
> (you host it, inside the boundary); "local" only named where it sits.*

#### Judge *(optional role of the Trusted AI)*
A Trusted AI that, at runtime, sees **both the Machine's deterministic output and
the real data**, and issues an **advisory, [receipted](#receipt) verdict** on a
case the deterministic rules cannot settle. A Judge never silently overrides a
deterministic outcome; its verdict is advice with a paper trail, not a gate.

**A Judge is optional and must be explicitly defined.** If a Machine's spec does
not define a Judge, the Machine has none. *Implementer's default: no Judge unless
one is explicitly specified.* A Judge is worth adding only when it earns its place
— it must resolve at least one case that passes every deterministic check (see the
complementarity requirement in the README).

#### Capability *(optional role of the Trusted AI)*
An inference call the frontier model **designs into the Machine** to provide a
*safe capability that is genuinely hard to express as deterministic code* (fuzzy
matching, free-text classification, and the like). Unlike the Judge — which sits
outside and comments — a Capability is wired into the Machine's own logic. For it
to work, the **Machine must expose a pluggable model endpoint**: at runtime it
connects to a Trusted AI inference endpoint reachable inside the Enclave. Also
optional; a Machine uses one only where the frontier model determined deterministic
code could not do the job.

---

## The two environments

There are exactly two environments, and no model crosses between them.

### The Forge *(build environment)*
Where the **frontier model works and constructs the Machine.** It is inherently
*unsafe and untrusted*, and that is fine, because **no customer data is present** —
only the [spec](#spec) (the blueprint) and [synthetic fixtures](#synthetic-fixtures)
(safe, constructed examples). Nothing real is here, so there is nothing to protect.
Outside the [trust boundary](#trust-boundary).

### The Enclave *(runtime environment)*
Where the **Machine runs, on real data.** A *constrained, trusted* environment:
**no frontier model is available**, and even **internet access cannot be assumed.**
The Machine must run within these limits. Any Trusted AI runs here too, self-hosted.
Inside the [trust boundary](#trust-boundary).

---

## The constructed thing

### The Machine
The **finished, deterministic tool the frontier model constructs** — the thing you
deploy and run in the Enclave. It can be arbitrarily complex; that complexity is
exactly why a frontier model builds it rather than a team by hand. Yet it must be
**trusted and auditable**, because it runs on real data with no frontier model
watching. The Machine is made of an [Engine](#engine) and a [Policy](#policy).

> *You may see "the artifact," "the tool," or "the harness" used loosely elsewhere
> for the same thing; this documentation says **Machine**.*

### Engine
The **stable, generic machinery** of the Machine. Rarely changes; **audited once**
by engineers. The Engine is where arbitrary complexity lives, and it must be
trusted despite that complexity.

### Policy
The **volatile, domain-specific ruleset** of the Machine — the rules and thresholds
particular to your problem. Changes often; **audited every cycle** by the domain
experts who own it. Separating Policy from Engine is what lets "auditable" mean
something operational: a reviewer re-reads the small Policy each cycle, never the
whole Engine.

---

## Inputs

### Spec
The **structure of the problem** the customer brings: what comes in, what must come
out, which cases are hard, which mistakes are unacceptable. The blueprint the
frontier model builds against. Bringing a spec does not require writing code.

### Synthetic fixtures
**Manufactured stand-in data** that shares the structure and hard cases of the real
data but contains **nothing real.** The safe, constructive examples the frontier
model designs against in the Forge. Because they are synthetic, the build discloses
nothing; because they are faithful to the structure, the resulting Machine is
faithful to the real work.

---

## Phases, guarantees, and receipts

### Build time / Runtime
The two **non-overlapping phases.** Build time happens in the Forge (frontier model
constructs the Machine); runtime happens in the Enclave (the Machine processes real
data). They never overlap.

### The ladder
The execution order inside the Machine: **deterministic code first; a Trusted AI
only where genuinely needed; the frontier model never.** Each rung keeps
[Sealed](#sealed) true.

### Trust boundary
The line real data must not cross — defined by **where processing happens**, not by
whether a link is encrypted. The Enclave is inside it; the Forge is outside it.

### Frozen / Transparent / Sealed
The three properties a regulated buyer asks for, defined in full in the README:
**Frozen** (it does not drift), **Transparent** (you can audit it), **Sealed**
(your data never reaches the frontier model).

### Manifest
The **timestamp-free receipt that proves what a Machine is**: content hashes of the
Engine, the Policy, and the originating Spec, plus pinned dependency versions, and
deliberately no timestamp so two builds of the same inputs are byte-identical. It
turns "Frozen" from a claim into something an auditor checks with `diff`.

### Receipt
The **durable runtime output of a Trusted AI call**: the verdict, a one-line
reason, and which model and version decided it. Transparency for the Trusted AI is
a runtime output obligation — without a receipt, an embedded model is a black box
inside otherwise-readable code.
