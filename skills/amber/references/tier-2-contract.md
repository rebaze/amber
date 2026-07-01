# The Trusted AI contract

The [SKILL.md](../SKILL.md) is the build procedure. This document is the part
it defers: the contract the [Trusted AI](glossary.md#trusted-ai) tier must satisfy,
and a reference adapter so every adopter does not independently rediscover the same
handful of sharp edges. Terms used here (Trusted AI, Judge, Capability, Machine,
Enclave) are defined in the [glossary](glossary.md).

The execution ladder is *deterministic code first; a trusted model only where it
is needed; the frontier model never.* The deterministic tier is the easy, elegant
half. The Trusted AI — self-hosted inside the Enclave — is where the real
engineering friction lives, and it is the one tier that talks to a model at
runtime. It needs a contract precisely because it is the tier most able to break
the method's guarantees if left informal. It applies to both roles of the Trusted
AI, the [Judge](glossary.md#judge-optional-role-of-the-trusted-ai) and the
[Capability](glossary.md#capability-optional-role-of-the-trusted-ai).

## What a Trusted AI call must guarantee

A conforming tier-2 call obeys four rules. Together they keep the three
properties (Frozen, Transparent, Sealed) true through the model tier.

1. **It cannot silently change a deterministic outcome.** A model verdict is
   advisory and receipted, never a silent pass/fail gate on a tier-1 result. The
   deterministic output is *invariant to whether the model is present at all.*

2. **It degrades by skipping, not failing.** If the endpoint is unreachable, the
   model times out, the budget is exceeded, or the output cannot be parsed, the
   judgment rule skips cleanly and records that it skipped. A missing model
   produces a smaller audit, not a broken run.

3. **It emits a receipt.** Every call produces durable output alongside its
   result: the verdict, a one-line reason, and which model and version decided it.
   Transparency for tier 2 is a runtime output obligation, not a property of the
   source code — a model embedded in readable code is still a small black box
   unless it explains itself in what it emits.

4. **It talks only to the declared endpoint.** The model is reached over one
   declared local endpoint the machine is allowed to talk to and no other, on
   infrastructure the operator controls, adding no network egress beyond it. This
   is what makes "inside the trusted boundary" checkable rather than asserted.

## Decision vs. flag

Before wiring tier 2, decide what a verdict is *allowed to do*: act
autonomously, or produce a receipted flag a human adjudicates. For an audit or
compliance posture the flag-for-a-human option is usually correct — it keeps every
consequential decision with an accountable person while still capturing the
model's reasoning as durable output. Whichever you choose, state it; it is a
design choice, not a detail.

## Complementarity check

Tier 2 earns its place only if it catches something tier 1 cannot. If every case
the model flags could also be caught by a deterministic rule, the model is
decoration — remove it. Validating an machine therefore includes constructing at
least one case that passes every deterministic check and is resolved only by the
model's judgment. If you cannot construct that case, do not ship the tier.

## Reference adapter

The recurring yak-shave. A conforming adapter:

- **Disables reasoning/thinking mode** where the server supports it. Reasoning
  models otherwise ignore an "answer PASS/FAIL" instruction and emit chain-of-
  thought instead of the verdict.
- **Parses the verdict robustly.** The verdict may arrive after chain-of-thought
  in the main content, or in a separate `reasoning_content` field, or wrapped in
  prose. Extract it defensively; on any ambiguity, skip (rule 2), do not guess.
- **Bounds the token budget** on both the request and the response, so a verbose
  model cannot blow up cost or latency. A call that would exceed the budget skips.
- **Skips, never fails, on any error** — connection, auth, timeout, malformed
  output — and records the skip in the receipt.
- **Captures the receipt** (verdict + reason + model + version) as durable output
  in whatever medium the machine already uses for its audit trail.

Endpoint and auth vary (a `127.0.0.1` OpenAI-compatible server with a key from a
secret manager is one common shape); the adapter isolates that variety behind the
four guarantees above so the rest of the machine never sees it.

## The deterministic tier's own auditability

A note that belongs here because it follows from the engine/policy seam: the
deterministic rule language is itself an auditability surface. A sandboxed
`eval` of Python expressions works, but sits in tension with "auditable by a
non-engineer." If domain experts are the ones re-reading the policy every cycle,
the deterministic tier wants a constrained, inspectable DSL rather than
Turing-complete code. Choose the policy language for the people who must audit it,
not only for the people who write it.
