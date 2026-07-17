# css-refusal-feed — the standing refusal record

Every day, an automated probe suite attacks a fresh instance of the
[Constitutional Stewardship](https://continuitylaboratories.com) governance
engine with the attacks it must refuse — false restoration of a suspended
steward, ratification of an invariant-violating amendment, a successor
claiming authority without inheriting its obligations, and the Continuity
Clock overreaching its own mandate. Each run emits a **signed refusal
digest**: the engine's verbatim refusal ground for every attack, plus the
signed event ledger of each attempt, in which the refused transition is
provably absent. This repository is where those digests are published.

Nothing here asks to be believed. Every file is independently verifiable
offline, and the latest digest is verified live, in your browser, at
<https://continuitylaboratories.com/refusal>.

## Layout

| Path | What it is |
|---|---|
| `feed/YYYY/MM/DD/refusal-digest-<UTC-stamp>.json` | one run's signed digest — **immutable once published** |
| `feed/YYYY/MM/DD/refusal-digest-<UTC-stamp>.pub.pem` | that run's Ed25519 public key (PKIX PEM) — each run signs with a fresh ephemeral keypair, and the public half is published beside the digest |
| `latest/refusal-digest.json` + `.pub.pem` | a convenience pointer to the newest run — the only files here that change |
| `index.ndjson` | one JSON line per run, append-only: `completed_at`, `digest_hash`, `seed`, `spec_version`, `attempted`, `refused`, `all_refused`, `path`, `run_id` |

**Append-only discipline:** published `feed/` entries and existing
`index.ndjson` lines are never rewritten. The git history of this repository
is itself part of the record — a mutated entry would be visible to anyone
with a clone.

## The seed chain

Digests are spec v0.2 ("Vigilance"): every identifier and ordering staged by
a probe's setup derives deterministically from the run's `run.seed`, so a
replayed transcript cannot match a fresh run. Seeds are **chained**: each
run's seed is `SHA-256` of the previous published digest's `digest_hash`
(ASCII bytes of the hex hash), read from this feed before the run. The
sequence of digests is therefore self-linking — tampering with any published
digest breaks every later seed. A run that cannot reach the feed starts a
new chain with a random seed; that restart is visible in `index.ndjson`.

## Verify a digest yourself

- **In your browser** — <https://continuitylaboratories.com/refusal> fetches
  `latest/` from this repository and re-verifies it in front of you: the
  canonical-bytes hash and Ed25519 proof, every probe ledger's event hashes
  and signatures, the provable absence of each refused transition, the seed
  derivations, and the recounted summary.
- **Independently** — the digest wire format, verification algorithm, and a
  conformance corpus (golden vectors + tamper matrix) are published under
  Apache-2.0 in [`jsabes24/css-succession-receipts`](https://github.com/jsabes24/css-succession-receipts)
  (`spec/refusal-transparency.md`, `corpus/refusal-v0.1`, `corpus/refusal-v0.2`).
  A verifier you write from that spec needs nothing from us but the digest
  and the `.pub.pem` beside it.

## Honest scope

A verified digest proves the record is intact and attributable to its key,
that the probe ledgers are genuine engine output, that the attacks named
were attempted, and that the refused transitions did not happen. It is
**self-attested** (`run.attestation: "self"`): the operator ran the probes
against its own engine and signed its own run. Independent countersigned
runs are the format's specified evolution. A digest also says nothing about
attacks it does not contain; the probe suite and cadence are published, and
a run in which a guard did NOT fire is published here exactly like any other
(`all_refused: false` in the index — the record is published, not hidden).

## Provenance

Commits here are made automatically by the scheduled `refusal-probes`
workflow of the (private) engine repository, one commit per run. This README
and LICENSE are maintained in that repository and synced by the same
workflow — corrections land there, never as hand edits here.

Published digests are provided under the [Apache License 2.0](./LICENSE).
© 2026 Continuity Laboratories.
