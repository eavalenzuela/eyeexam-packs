# Planned Improvements & Feature Roadmap

Engineering roadmap for the **eyeexam-packs** catalog: 43 sandboxed,
self-cleaning Linux adversary-emulation test packs consumed by the
`eyeexam` breach-and-attack-simulation runner. Each pack is a
detection-validation artifact (`execute` → `verify_cleanup` →
`expected_detections`), not an attack tool — every technique is
simulated at file scope under `/tmp/eyeexam-<id>` with no external
traffic and no real destruction. This document is a scoping roadmap
only; nothing here authorizes shipping offensive code, live payloads,
or evasion tradecraft. All items keep the catalog firmly in the
"safe, sandboxed, authorized-use-only detection test" lane.

## Improvements (10)

1. **Sign the catalog (ed25519 `MANIFEST` + `MANIFEST.sig`).** The
   repo ships no signature, so operators must load it with
   `unsigned: true` and take an audited trust downgrade on every run;
   a signed manifest lets `eyeexam` verify integrity at each load and
   closes the catalog's biggest authorization/supply-chain gap.

2. **Add a JSON Schema for the pack format plus a validator.** Codify
   required fields, enum-constrain `destructiveness`/`tactic`/
   `platforms`, and enforce `id` ↔ filename agreement so malformed or
   half-written packs fail fast at author time instead of at run time.

3. **Stand up CI (GitHub Actions) as a merge gate.** Run schema
   validation, `shellcheck` over every `execute`/`cleanup`/
   `verify_cleanup` block, and duplicate-`sigma_id` detection on each
   PR so quality and safety are enforced mechanically, not by review
   memory.

4. **Add a containerized end-to-end smoke harness.** Execute every
   pack in a disposable VM/container asserting `execute` succeeds,
   `verify_cleanup` passes, and no residue is left outside the
   sandbox directory — the only way to prove the self-cleaning
   contract actually holds across all 43 packs.

5. **Standardize and lint the sandbox/cleanup contract.** Enforce a
   single derived work directory per pack, idempotent cleanup, and
   safe re-run when a prior sandbox already exists, then lint that no
   `execute` writes outside its own sandbox path — a core safety
   guardrail against a pack touching the host.

6. **Uniform pre-flight dependency guards and graceful skips.** Only
   some packs check tool availability (the `command -v dig` pattern);
   generalize this so every pack that needs `dig`/`nc`/`rsync`/etc.
   degrades to a clean, recorded skip instead of a spurious failure
   that pollutes coverage scoring.

7. **Parameterize and bound resource-heavy tests.** Fixed 10 MiB `dd`
   writes, large-file splits, and busy loops should read tunable,
   capped sizes/iterations so runs stay fast, quiet, and predictable
   on constrained or shared hosts — a performance and blast-radius win.

8. **Normalize `expected_detections` and add a detection-map index.**
   Replace placeholder sigma UUIDs with real, unique rule IDs and
   maintain a cross-reference file linking each test to the detection
   content it validates, so "missed" results are actionable and
   reporting is trustworthy.

9. **Enrich per-pack metadata for selection and provenance.** Add
   `tags`, ATT&CK sub-technique references/URLs, and added/updated
   dates and owner fields so the runner's `--tag` selectors work and
   every run's audit record carries meaningful, traceable test
   provenance.

10. **Author catalog documentation and a responsible-use notice.**
    Ship a repo README, a `CONTRIBUTING` guide encoding the pack
    authoring contract (sandbox-only, verify_cleanup required,
    low-by-default destructiveness), and an explicit
    authorized-use-only / scope statement — currently the catalog has
    no self-describing docs at all.

## New Features (5)

1. **Multi-platform pack trees.** Add parallel `windows/` and
   `macos/` catalogs mapping the same techniques the Linux packs
   cover, so detection validation is not limited to one OS while
   preserving the sandboxed, non-destructive contract per platform.

2. **Cross-backend detection expectations + coverage report.** Extend
   `expected_detections` beyond the single `slither` backend to the
   runner's `loki`/`wazuh`/`elastic`/`splunk` detectors and generate a
   matrix showing which SIEM each test exercises — turning the catalog
   into a portable, multi-SIEM coverage baseline.

3. **ATT&CK coverage expansion tracked against a manifest.** Broaden
   the catalog into the currently thin/empty tactic areas at a high
   level (additional discovery, defense-evasion, collection, and
   initial-access coverage) against a target sub-technique manifest,
   keeping every new entry sandboxed and detection-focused.

4. **Named pack profiles / bundles.** Curated selectable subsets
   (e.g. "quick smoke", "credential-focused", "destructiveness=low
   only") that map to scenario-based emulation runs, so operators pick
   an intent instead of hand-listing test IDs.

5. **Versioned, signed catalog releases.** Tagged releases with a
   changelog and a checksum manifest so operators can pin and verify a
   known-good catalog version, giving the detection baseline
   reproducibility and a clean supply-chain provenance trail.
