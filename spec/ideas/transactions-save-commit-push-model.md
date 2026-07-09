---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: Transactions and the Save / Commit / Push Durability Model

**Status:** Draft
**Date:** 2026-07-09
**Owner:** alexander.trakhimenok@gmail.com
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How should inGitDB define transactions when its storage medium naturally has
three distinct durability levels — file saved to the work tree, change
committed to git, commit pushed to a remote — and what atomicity, isolation,
rollback and publication guarantees should each level give to applications
(dalgo drivers, OpenVaultDB, the CLI, sync tooling)?

## Context

Observed while building the OpenVaultDB MVP (2026-07-09), which drives
dalgo2ingitdb through a server:

- `RunReadwriteTransaction` today writes record files immediately, one
  exclusive flock per file; there is no rollback: a worker failing after
  writing some files leaves those writes on disk.
- The git commit is opt-in (non-empty `dal.TxWithMessage`) and covers exactly
  the files tracked during the transaction — at most one commit per
  transaction. On a plain (non-git) directory the commit step is silently
  skipped.
- Push is entirely out of scope of the driver. OpenVaultDB added a
  per-database push policy (`none | sync | async`, coalescing background
  pusher for async) as a server-level after-write hook — a pragmatic MVP
  answer that belongs in a deliberate model instead.
- Consumers had to bolt on compensations: OpenVaultDB pre-validates whole
  batches before letting any file write happen (poor man's atomicity) and
  synthesizes default commit messages so history stays meaningful.
- The honest concurrency contract is single-writer per tree (advisory flocks,
  per-file granularity, cross-platform caveats).

The durability ladder, side by side:

| Level | Mechanism | Survives | Visible to | Reversible via |
|-------|-----------|----------|------------|----------------|
| Saved | file write in work tree | process crash | local readers | manual cleanup (today: nothing) |
| Committed | git commit | branch resets, tooling | local readers + git tooling | `git revert` / `reset` |
| Published | git push | local disk loss | other clones / collaborators | force-push or revert commit |

Key insight: git gives inGitDB a rollback and snapshot mechanism for free —
the current implementation just doesn't use it. `HEAD` is a consistent
snapshot; the work tree diff is the "dirty" transaction state.

## Recommended Direction

Make the three-level durability ladder a **normative part of the inGitDB
specification**, expressed as named acknowledgement levels a caller chooses
per transaction (`saved` | `committed` | `published`), with `committed` as
the default for readwrite transactions. Implementations map the levels to
their storage: the Go driver via work-tree writes, git commits and pushes.
Add git-based rollback to the Go driver (restore tracked files from `HEAD`,
delete files created by the failed transaction — the driver already tracks
every written path), documented as best-effort on plain directories.
Auto-generated commit messages become the driver default so history never
degrades to dirty trees swept up by the next committer.

## Alternatives Considered

1. **Keep transactions driver-specific (status quo).** No spec changes; each
   consumer invents its own compensations (as OpenVaultDB did). Rejected:
   guarantees diverge across implementations (Go vs TS), and applications
   cannot reason about durability portably.
2. **Git-index-staged transactions.** Write transaction state into the git
   index (or a temp tree) via plumbing, materializing the work tree only on
   commit. Strongest isolation; considered too invasive for the current
   file-oriented driver and unusable on plain directories. Revisit if MVCC
   reads become a requirement.
3. **Sidecar journal / WAL.** A database-style undo log alongside the tree.
   Rejected: duplicates what git already provides, adds a second source of
   truth inside a repo whose whole point is that files ARE the truth.
4. **Branch-per-writer with merges as the only multi-writer story.** Not an
   alternative to the ladder but a complement; kept as a future direction
   (see Open Questions).

## MVP Scope

- Spec: define the durability ladder and the three ack levels with their
  guarantees and failure semantics (partial save, commit failure, push
  rejection).
- Go driver: rollback-on-error from `HEAD` for git-backed trees; commit by
  default with an auto-generated message when none is supplied; expose ack
  level via `dal.TransactionOptions`.
- OpenVaultDB: replace its bespoke push hook with the spec's ack levels
  (manifest default + per-request override).
- `ingitdb doctor` / `ovdb status`: surface "N commits unpublished" as a
  health signal.

## Not Doing (and Why)

- **Distributed consensus / multi-master replication** — out of scope;
  git-based publication plus single-writer trees is the model.
- **CRDT-style merge of record files** — conflict semantics belong to a
  future sync layer, not the core transaction model.
- **Server-side multi-tenant git hosting** — an OpenVaultDB hosting concern,
  not an inGitDB spec concern.
- **Encryption at rest** — orthogonal; tracked separately.
- **MVCC snapshot reads** — deferred until a consumer demonstrates need;
  read-committed via `HEAD` blobs is the documented ceiling for now.

## Key Assumptions to Validate

1. Restoring tracked files from `HEAD` plus deleting newly created files is a
   sufficient and safe rollback for every record-file layout (SingleRecord,
   MapOfRecords) — including partially written multi-record files.
2. Committing by default does not unacceptably slow write-heavy workloads
   (one `git add` + `git commit` per transaction); measure and, if needed,
   allow `saved`-level acks with periodic auto-commit.
3. Async publication with single-flight coalescing loses no acknowledged
   data on process crash (the commits are local; only the push is pending).
4. Non-fast-forward push rejections can be handled with fetch+rebase for
   append-mostly record workloads without corrupting definitions.
5. The ack-level API can be expressed through `dal.TransactionOptions`
   without breaking existing dalgo consumers.

## SpecScore Integration

- Promote to a Feature under `spec/features/` (e.g. `transactions/`) defining
  REQ ids for: ladder levels and their guarantees, rollback behavior,
  default-commit behavior, push policies, unpublished-commits health signal.
- Conformance vectors under `conformance/`: failing-transaction rollback
  scenarios, ack-level acknowledgement matrices, push-rejection handling —
  executable by both the Go CLI and future TS implementation.
- Cross-reference from the dalgo2ingitdb repo spec and from OpenVaultDB's
  architecture doc (which currently documents the pragmatic interim model).

## Open Questions

1. Should `committed` be the default ack level, or is opt-in commit (today's
   behavior) preferable for CLI-heavy local workflows?
2. What exactly does a `published` ack mean when the remote rejects a
   non-fast-forward push — fail the transaction (data is already committed
   locally) or return a distinct "committed-not-published" outcome?
3. Is rollback safe to attempt on plain (non-git) directories via a sidecar
   undo buffer, or should plain directories simply document weaker
   guarantees?
4. Where does branch-per-writer multi-writer support live — driver, ovdb, or
   a sync layer above — and how do merge conflicts surface to applications?
5. Which parts are normative spec versus Go-driver implementation guidance
   (e.g. flock strategy is clearly implementation; is rollback mechanics)?
6. Should isolation be strengthened to read-committed (`HEAD` blob reads)
   for readonly transactions, and at what plumbing cost?
