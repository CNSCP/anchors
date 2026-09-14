# CP Registry — signed anchors

A mirror of the signed anchors of the [Connection Profile Registry](https://cp.cnscp.io)'s
audit chain, kept here so that the anchors do not live only on the host they exist to check.

This repository holds no code and never will. Its commit history is the point: each commit
adds one anchor, at a time this repository records and the Registry does not control.

## What an anchor is

Every act on the Registry — a name registered, a version published, a Prefix transferred — is
written to an append-only log, each entry carrying a hash of the one before it. That makes
editing the past detectable: alter anything and every entry after it stops matching.

What a hash chain cannot catch is a **fork**. A registry could keep two internally consistent
histories and serve one to each party; both would verify, and neither party could tell,
because the only thing either has to check against is the chain that same host handed them.

An anchor closes that gap. It is one small document committing to a single fact —

> at this moment, the head of my chain was exactly this

— signed with a key held outside the Registry's infrastructure. Two parties whose copies match
the same signed head are provably holding the same history. To fork them, the operator would
have to sign two different heads for one sequence, and both signatures are public, so the
contradiction is itself the evidence.

## Layout

```
keys.json                    the keys an anchor may be signed by
2026/09/000207-20260914T162308.json    one anchor: head sequence, then when it was signed
```

Anchor files are named by head sequence and signing time, so they sort in the order they were
made. Nothing here is ever rewritten: an anchor that turns out to be wrong is superseded by a
later one, never edited, because the history of what was claimed is part of what this records.

## Checking an anchor

Each file is exactly what the Registry serves at `/.well-known/cp-anchor`. The signature is
Ed25519 over a canonical serialization of the six fields, in this order and no other:

```
{"journal_format":…,"origin":…,"head_seq":…,"head_event_hash":…,"at":…,"key_id":…}
```

The `at` field is used **exactly as written**. It is part of what was signed, so a timestamp
re-formatted — even into the same instant — is a different document and will not verify.

With the Registry's own tooling, which walks the public journal and compares it to the signed
head:

```sh
npm run verify-journal -- https://cp.cnscp.io --anchor cp-anchor-2026-09
```

Or from scratch, in a few lines of any language with an Ed25519 implementation: fetch the
journal, recompute each entry's hash and confirm the links, then check the anchor's signature
and compare its `head_event_hash` against what you arrived at for that sequence. No account,
no credential, no relationship with the operator.

## The root key

`cp-anchor-2026-09` is vouched for by nothing here — that is what makes it the root. Its
fingerprint is published where a person can check it by eye:
[cnscp.io/anchor.html](https://cnscp.io/anchor.html), the
[README](https://github.com/CNSCP/registry) of the Registry, and §20.2 of the
[design document](https://github.com/CNSCP/registry/blob/main/REGISTRY-DESIGN.md).

```
463f 5b19 07b9 4d22 569a 5710 a1ae 2525 eaf4 a23e 2bf6 52fa 4c94 10ae a843 ca72
```

Later keys carry the previous key's signature over their own canonical form, in `keys.json`,
so a verifier can walk from a key it already trusts to the one in question.

## Cadence, and what an anchor does not say

An anchor is published after anything irreversible — a publication, a deprecation, an
allocation, a transfer — and weekly regardless. Everything up to the signed head is attested;
everything after it is not, which is why the weekly floor exists: it makes a quiet week an
*attested* quiet week rather than an absence.

An anchor says nothing about whether a Profile is correct. A wrong contract, correctly
published, is anchored like any other — permanence is the Registry's promise, not correctness.

## If two anchors disagree

If this mirror and the Registry ever serve different anchors for the same sequence, or two
signatures exist over one sequence with different heads, **keep both**. That is not a glitch
to be tidied away; it is exactly the evidence this whole arrangement exists to produce.
