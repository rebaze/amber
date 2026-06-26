# Amber

**The model that designs your data processing never sees your data.**

Amber is a method for putting frontier-model capability to work on regulated data
without ever disclosing that data to the frontier model. The model works only at
build time, on the structure of your problem and on synthetic stand-in data. What
it produces is a finished, deterministic tool that runs inside your own trusted
environment, against your real data, with no model in the loop.

---

## The bind

A frontier model can design how your data should be processed better than most
teams can do it by hand. Give it the shape of a problem — records to classify,
fields to extract, text to redact, anomalies to flag — and it will design
processing that is more accurate, more complete, and more robust than the rules a
team would write under deadline.

For regulated data, you cannot use that capability the obvious way. Banking
records, patient files, anything under GDPR — you cannot hand it to a US or
Chinese cloud model. The constraint is not about encryption. A TLS connection to
a foreign cloud is still disclosure to a foreign cloud; the data has left your
trust boundary the moment it is processed there. The trust boundary is about
*where processing happens*, not whether the link in transit is encrypted.

So regulated teams are left with a bad choice. Either forgo frontier-quality
processing entirely and build weaker logic in-house, by hand, at the level of
capability your own team can muster — or stand up and serve a capable model
yourself, paying for the infrastructure and the operational risk, and still
getting weaker results than the frontier. Neither option gives you frontier
capability on data you are not allowed to disclose.

Amber removes the choice.

## The inversion

Amber forms when resin from a living tree flows around something and then hardens.
The organism is gone, but its form is preserved with perfect fidelity — frozen,
permanent, and clear enough to study millions of years later. The insect is
absent from the amber that holds its shape.

Amber, the method, inverts the usual arrangement between a model and your data.
The usual arrangement sends your data to the model at the moment of processing.
Amber never does. The model works *only at build time* — on the structure of your
problem, and on synthetic data manufactured to stand in for the real thing. From
that, and only that, it produces a finished tool.

Then it sets.

What you deploy is a hard, deterministic artifact. It runs inside your own trusted
environment, against your real data, with no model in the loop. The capability is
preserved in the tool. The model that produced it is absent by construction — the
same way the insect is absent from the amber that holds its shape. The thing with
the capability and the thing with the data are separated in time, and they never
meet.

## Three properties

The metaphor is not decoration. Three properties come straight out of it, and each
answers a question a regulated-industry buyer actually asks.

### Frozen — *does it drift?*

The artifact's behavior is fixed at build time. It does not retrain, does not
update itself, does not call home. The same input yields the same output, every
time, on the day you deploy it and a year later. There is no model in the runtime
to drift, no silent version bump from a vendor, no behavior that changes
underneath you. What you tested is what runs. What you audited stays audited.

### Transparent — *can I audit it?*

Amber is clear. What Amber produces is readable, auditable code — not an opaque
model whose behavior you can only sample and hope. You can hold the artifact to
the light and see exactly what it does: the logic, the branches, the handling of
each case. This is what an auditor needs, and increasingly what regulation
demands. Under regimes like NIS2 and the Cyber Resilience Act, accountability is
personal and specific; the person who signs needs to be able to see, and show,
what the system does. You cannot do that with a model. You can do it with the tool
Amber leaves behind.

### Sealed — *where does my data go?*

Your real data flows through the tool and never to the model that designed it. The
capability and the data are separated in time and never meet. This is the
load-bearing guarantee, and it does not rest on a promise. It rests on the fact
that the frontier model is simply not present on the runtime data path. There is
nothing to trust, because there is nothing there. A "we don't train on your data"
assurance is a policy; Sealed is a property of the architecture.

## How it works

Amber has two phases, and they never overlap.

**At build time,** the frontier model receives the specification of your problem
and a set of synthetic fixtures — data manufactured to stand in for the real
thing, sharing its structure and its hard cases but none of its actual content.
The model designs the processing and emits a deployable harness: the tool. This is
where the capability is captured. It happens entirely outside your trust boundary,
because nothing real is present — there is nothing to protect.

**At runtime,** the harness runs inside *your* trusted environment, against your
real data. The frontier model is not in this loop. It was never given an address
to call, because the architecture gives it none.

Most processing the harness needs is fixable logic — rules, transforms, decisions
that can be written down once and frozen. That runs as plain deterministic code.
Where a task genuinely needs judgment that cannot be reduced to fixed logic, the
harness calls a **self-hosted local model, inside the same trusted boundary**. That
model runs on your infrastructure, under your control, on data that never leaves.
The execution ladder is: deterministic code first; a local model only where it is
needed; the frontier model never. Each rung keeps Sealed true. The frontier model
designed the whole arrangement and appears nowhere in it.

## What "a spec" is

You do not need to write code to use Amber. What you bring is a *spec*: the
structure of your problem, and synthetic fixtures that stand in for your real data.

The structure is the shape of the work — what comes in, what must come out, which
cases are hard, which mistakes are unacceptable. The fixtures are stand-ins:
records that look and behave like yours, manufactured for the purpose, carrying
the structure and the edge cases that matter without carrying anything real. The
frontier model designs against these. Because the fixtures are synthetic, the
build phase discloses nothing — and because they are faithful to the structure,
the tool that results is faithful to your real work.

## Where it breaks

Amber is not a universal solvent, and the honest line matters more than the
pitch.

**The boundary.** Amber fits problems whose *judgment can be fixed at build time*.
If a task requires open-ended frontier reasoning on live data — reasoning that
cannot be reduced to deterministic logic and is beyond what a self-hosted local
model can do — then it is out of scope for Amber. There is no trick that lets you
have frontier reasoning on the real data at runtime without disclosing the real
data at runtime. Amber does not pretend otherwise. That is the line.

**The texture.** In practice the line cuts through the middle of most real
pipelines, not around them. A pipeline is rarely one monolithic act of judgment;
it is mostly fixable logic with a few genuinely hard spots. Amber freezes
everything it can into deterministic code and lets the local model cover the
residue. The residue shrinks; it does not always vanish.

**The move.** So part of the method is decomposition — designing the problem so
that the frozen artifact plus a local model covers it, isolating the parts that
truly need judgment from the parts that only looked like they did. A problem that
cannot be decomposed this way is not an Amber problem, and you will know early.

**Verifying the seal.** Because the deployable is transparent code running in your
own environment, you do not have to take the seal on faith. You can inspect the
artifact and confirm what it talks to. The absence of the frontier model on the
data path is a thing you can check, not a thing you are told.

## Why now, and what it changes

The pressure is from both sides at once. Frontier capability is pulling further
ahead of what in-house teams can build by hand, so the cost of *not* using it
rises every quarter. At the same time, regulation is making disclosure to foreign
cloud models harder and the personal accountability for systems sharper. The gap
between "the best way to process this" and "the way you are allowed to process
this" keeps widening.

Amber closes it by separating the two things that the usual arrangement
conflates: the capability that designs the processing, and the data the processing
runs on. Keep them apart in time and you can have frontier-designed quality on data
you are never allowed to disclose — with a guarantee that is structural rather than
promised, and at a cost closer to running your own code than to serving your own
model.

The model that designs your data processing never sees your data. That is the
whole idea, and everything else follows from it.
