---
name: code-slimming-cleanup
description: Code slimming and cleanup guidelines. Use when reducing code size, removing dead code, simplifying abstractions, consolidating duplicate logic, or cleaning up after refactors while avoiding risky drive-by changes.
license: MIT
---

# Code Slimming and Cleanup

Guidelines for safely making code smaller, simpler, and cleaner without turning cleanup into an uncontrolled refactor.

**Tradeoff:** This skill favors safe, scoped cleanup over aggressive rewrites. If broader cleanup is needed, get explicit approval first.

## 1. Define the Cleanup Scope

**Name what kind of cleanup is being performed before editing.**

Classify the task as one of:

- **Local cleanup** — remove unused imports, variables, branches, helpers, or files created by the current change.
- **Simplification** — reduce unnecessary abstractions, indirection, configuration, or defensive code.
- **Dead-code removal** — delete code that is provably unreachable or unused.
- **Duplication reduction** — merge duplicate logic only when the shared behavior is truly identical.
- **Aggressive cleanup** — broad deletion, reorganization, or public API changes.

If the task may affect behavior, public APIs, data formats, migrations, or external contracts, stop and confirm the scope.

## 2. Prefer Deletion Over Abstraction

**Slimming means less code, not more structure.**

When simplifying:

- Delete unnecessary code before introducing new helpers.
- Inline single-use abstractions when doing so improves readability.
- Remove unused options, flags, parameters, and configuration only when they are not part of a public contract.
- Do not add a framework, registry, strategy pattern, factory, or generic abstraction just to make cleanup look organized.
- Do not replace simple duplication with an abstraction unless the shared behavior is stable and obvious.

Ask: "Did this change reduce the amount of code a future reader must understand?"

## 3. Prove Code Is Dead Before Removing It

**Do not delete code based on vibes.**

Before removing existing code, verify at least one of:

- Type checker, compiler, linter, or IDE reports it as unused.
- Search shows no references across the relevant repository scope.
- Tests cover the path and still pass after removal.
- The code is behind a removed feature path or obsolete configuration explicitly identified in the task.
- The user explicitly asked to remove that code.

If unsure, mention the suspected dead code instead of deleting it.

## 4. Keep Cleanup Surgical

**Do not turn cleanup into a drive-by refactor.**

Allowed:

- Remove code made unused by your current change.
- Simplify code directly involved in the requested task.
- Rename only when the old name becomes misleading because of the cleanup.
- Update tests directly affected by removed or simplified behavior.

Avoid unless explicitly requested:

- Reformatting unrelated files.
- Moving files for aesthetics.
- Changing public APIs.
- Rewriting working code in a preferred style.
- Mixing unrelated cleanup items in one diff.
- Deleting comments or docs you do not fully understand.

The test: every deleted or changed line should have a clear reason tied to the cleanup goal.

## 5. Preserve Behavior Unless Asked Otherwise

**A slimming PR should usually be behavior-preserving.**

Before and after cleanup:

- Identify expected behavior that must stay the same.
- Run existing tests when available.
- Add or update tests only when needed to lock down behavior that cleanup could accidentally change.
- If tests are missing, use type checks, builds, search, or targeted manual checks as fallback verification.

If the cleanup intentionally changes behavior, state that clearly and treat it as a feature or bug-fix change, not pure cleanup.

## 6. Report What Was Removed

When finishing, summarize cleanup by category:

- **Deleted:** files, functions, branches, imports, configuration, or dependencies removed.
- **Simplified:** abstractions, conditionals, parameters, or call paths reduced.
- **Kept intentionally:** suspicious code that looked removable but was not proven safe to delete.
- **Verified by:** tests, build, type check, lint, search, or manual inspection.

Good cleanup leaves reviewers confident that less code now does the same job.
