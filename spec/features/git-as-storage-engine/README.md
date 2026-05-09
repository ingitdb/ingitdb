# Feature: Git as Storage Engine

**Status:** Draft

## Summary

In inGitDB, Git is not a backup or transport layer — it is the storage engine. Every write produces a commit, every isolated change set is a branch, and the full history of the database is preserved. Standard Git operations — branch, merge, bisect, revert, blame, cherry-pick — apply directly to data records.

## Problem

Conventional databases offer transactions but not history; you can roll back the last operation but not see what a row looked like a year ago, who changed it, or why. Conventional version control offers history but not data semantics. inGitDB collapses the two: by storing records as files in a Git repository, history, branching, and review come for free, and the team workflows already used for code apply unchanged to data.

## Behavior

### Commits as the unit of change

Every committed write to the database is a Git commit. There is no "uncommitted" persistent state; the working tree before a commit is a draft, exactly as in any Git repository.

#### REQ: commit-per-change

Every persistent change to records or schema MUST be expressed as a Git commit. The commit graph is the authoritative log of database changes.

#### REQ: standard-git-operations

Standard Git operations MUST work on the database without translation. `git log` shows record history, `git blame` attributes record changes, `git revert` undoes them, and `git bisect` searches them.

### Branches as isolated change sets

Branches isolate work on data the same way they isolate work on code.

#### REQ: branches-native

Branches in the underlying Git repository MUST be first-class branches of the database. A branch can be reviewed, merged, or discarded using standard Git mechanics.

### Distribution

The database is whatever a Git clone of the repository contains. There is no separate replication channel.

#### REQ: clone-equals-database

A `git clone` of the repository at a given ref MUST yield a complete, self-contained snapshot of the database at that ref. No additional download, restore, or hydration step is required.

### Server-optional reads

Reading from the database is a file-system operation on a clone. There is no server in the read path.

#### REQ: no-server-for-reads

A consumer with a local clone MUST be able to read any record without running an inGitDB daemon, server, or background process.

## Dependencies

- storage-format

## Acceptance Criteria

### AC: history-is-git-history

**Requirements:** git-as-storage-engine#req:commit-per-change, git-as-storage-engine#req:standard-git-operations

The history of any record is recoverable through standard Git tools (`git log -p <file>`, `git blame <file>`). Bisecting a database for the introduction of a bad value works using `git bisect` without any inGitDB-specific tooling.

### AC: branch-merge-roundtrip

**Requirements:** git-as-storage-engine#req:branches-native, git-as-storage-engine#req:clone-equals-database

A user creates a branch, modifies records on it, and merges it back to the main branch using standard Git commands. The resulting state is a valid inGitDB database with merged data.

## Outstanding Questions

- How are very large databases (millions of records) handled given Git's per-file overhead — is there guidance on shard boundaries or partial clones?
- Are non-Git VCS backends (e.g. Mercurial, Jujutsu) explicitly out of scope, or are they a future consideration?

---
*This document follows the https://specscore.md/feature-specification*
