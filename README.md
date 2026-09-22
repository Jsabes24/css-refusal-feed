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
| `feed/YYYY/MM/DD/not-run-<UTC-stamp>.json` + `.pub.pem` | a signed record that a suite **did not run**, and why — see below |
| `latest/refusal-digest.json` + `.pub.pem` | a convenience pointer to the newest **governance** run. Its meaning is fixed: the probe reads it to chain the next seed, and the site re-verifies it |
| `latest/<suite>/refusal-digest.json` + `.pub.pem` | the same pointer for every other suite |
| `latest/badge.json` + `latest/feed.xml` | non-normative legibility artifacts regenerated per run from the index — a [shields endpoint](https://shields.io/badges/endpoint-badge) badge and an RSS feed of recent runs (see below) |
| `index.ndjson` | one JSON line per run, append-only: `suite`, `completed_at`, `digest_hash`, `seed`, `spec_version`, `attempted`, `refused`, `all_refused`, `path`, `run_id`. A line that did not run carries `outcome: "not_run"` and `why` instead of the counts. **A line with no `suite` is `css-governance`** — every line written before there was a second suite was that one, and rewriting them would break the append-only discipline |

Only the files under `latest/` ever change; everything else is append-only.

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

**Each suite chains its own seeds.** More than one engine publishes here, and
a seed derived from another suite's digest would fork both chains silently —
visible only to somebody walking one backwards later. The governance suite
chains from `latest/refusal-digest.json`, which is why that pointer means what
it has always meant and never moves to another suite's run.

## Embed the record

The streak is embeddable anywhere markdown or a feed reader goes. Both
artifacts are derived from `index.ndjson` on every publish — they are
conveniences, not the record; the record is the digests, keys, and index.

Badge (live, and honest in four states — green when every suite ran recently
and refused everything, **amber when a suite has gone quiet**, red when a
guard did not fire or a suite published a *did not run*, grey when nothing
has been published at all). The message carries the date of the newest run,
so a cached badge still says when it was true:

```markdown
![standing refusals](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Fjsabes24%2Fcss-refusal-feed%2Fmain%2Flatest%2Fbadge.json)
```

RSS (newest 20 runs, one item per published digest):

```
https://raw.githubusercontent.com/jsabes24/css-refusal-feed/main/latest/feed.xml
```

## When a run does not happen

A digest says what the guards refused. It has no way to say *nothing was asked of
them*, and for twenty-eight days in 2026 that gap was visible here: the badge read
**"41 runs · every attack refused"** in bright green while the probe had not run
since 25 August. The workflow lived in a private repository, its Actions minutes
ran out, and the newest signed fact stayed true forever. A dead probe and a perfect
record rendered identically.

So a suite that cannot run publishes `css-refusal-notrun` — a separate format, not
a digest with the probes left out. That distinction is load-bearing: a v0.2 digest
verifier handed an empty digest reports `0 attempted, 0 refused`, and *all refused*
renders that **true**. An absence is a different `spec` string, which a digest
verifier refuses on its first structural check, and it carries:

| field | |
|---|---|
| `suite` | which suite did not run |
| `at` | when it should have |
| `why` | **required.** An absence that does not name its cause is the silence this exists to end, signed |
| `previous_digest_hash` | the last digest that did run, so the record cannot be quietly dropped |

**Its honest scope is narrow.** A verified absence proves somebody signed a
statement that the probe did not run. It says nothing about the guards — none were
exercised — and it cannot reach a producer who simply publishes nothing at all.
What it removes is the absence that *looks* like health.

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

Commits here are made automatically, one per run, by whichever publisher ran:

* the scheduled `refusal-probes` workflow of the (private) engine repository,
  committing as `github-actions[bot]`, for the governance suite;
* a daily timer on the operator's own box, committing as `css-refusal-probe`, for
  the Echonet suite. It runs there because Actions are metered on private
  repositories and free on public ones, and because a schedule on hardware that is
  already up does not depend on GitHub disabling cron in a quiet repository.

Neither is a person, and the author line says so.

This README and LICENSE are maintained in the engine repository under
`.github/refusal-feed/` and synced from there by whichever publisher runs —
corrections land there, never as hand edits here.

Published digests are provided under the [Apache License 2.0](./LICENSE).
© 2026 Continuity Laboratories.
