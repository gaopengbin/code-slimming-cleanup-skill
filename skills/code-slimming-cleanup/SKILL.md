---
name: code-slimming-cleanup
description: Code slimming and cleanup guidelines. Use when reducing code size, removing dead code, simplifying abstractions, consolidating duplicate logic, or cleaning up after refactors while avoiding risky drive-by changes.
license: MIT
---

# Code Slimming and Cleanup

Guidelines for safely making code smaller, simpler, and cleaner without turning cleanup into an uncontrolled refactor.

**Tradeoff:** This skill favors safe, scoped cleanup over aggressive rewrites. If broader cleanup is needed, get explicit approval first.

## Scan Procedure

Follow this order to avoid wasted work and ensure nothing is missed:

1. **Exclude** — Identify build artifacts, generated files, and `node_modules/`. Skip them from all subsequent steps.
2. **Hygiene** — Check for missing `.gitignore`, tracked build output, misplaced dependencies.
3. **File sizes** — Count lines per source file. Flag files exceeding thresholds.
4. **Duplication** — Grep for same-name function declarations across files. Compare bodies.
5. **Dead code** — Search for unexported/unreferenced functions, unused imports, unreachable branches.
6. **Dependencies** — Cross-reference `package.json` entries against actual imports in source.
7. **Report** — Output findings using the severity levels and format defined in this document.

For small projects (< 500 total source lines), skip steps 3-4 and only perform hygiene, dead code, and dependency checks. In the report, omit the Duplicates and Oversized files categories entirely rather than including empty sections.

## Severity Levels

Every finding must be tagged with a severity:

| Level | Meaning | Examples |
|-------|---------|----------|
| **Critical** | Wastes significant resources or blocks correct operation | Missing `.gitignore` with 300MB+ tracked artifacts; committed secrets |
| **Warning** | Maintenance burden or code smell that should be fixed soon | 100+ lines of exact duplication; file > 800 lines; unused dependency |
| **Info** | Worth noting but low urgency | File approaching 400 lines; 3-line utility duplicated twice; dependency that could move to devDependencies |

Order findings by severity in the report. Critical items first.

## 0. Exclude Build Artifacts and Generated Files

**Never analyze or report generated files as cleanup candidates.**

Before scanning, identify and skip:

- Build output directories (`dist/`, `build/`, `dist-electron/`, `.next/`, `.output/`, `out/`)
- Bundled files (e.g., webpack/esbuild/rollup output like `assets/bundle.js`)
- Compiled artifacts (`.exe`, `.dll`, `.asar`, `.pak`, `.bin`)
- Auto-generated code (protobuf output, GraphQL codegen, OpenAPI clients)
- `node_modules/`, `vendor/`, lock files

Instead, flag missing `.gitignore` rules if these artifacts are tracked in version control. The fix is a `.gitignore` entry, not deleting the files manually.

## 1. Define the Cleanup Scope

**Name what kind of cleanup is being performed before editing.**

Classify the task as one of:

- **Local cleanup** — remove unused imports, variables, branches, helpers, or files created by the current change. (Applies: sections 2, 3, 8, 9)
- **Simplification** — reduce unnecessary abstractions, indirection, configuration, or defensive code. (Applies: sections 2, 3, 5, 8, 9)
- **Dead-code removal** — delete code that is provably unreachable or unused. (Applies: sections 2, 3, 6, 8, 9)
- **Duplication reduction** — merge duplicate logic only when the shared behavior is truly identical. (Applies: sections 2, 4, 8, 9)
- **Aggressive cleanup** — broad deletion, reorganization, or public API changes. Requires explicit user approval. All sections apply; section 8's surgical constraint is relaxed but section 9 still applies.

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

## 4. Detect and Reduce Duplication

**Identical logic repeated across files is a maintenance liability, not just a style issue.**

Detection strategy:

1. Grep for function declarations with the same name across all source files.
2. For each match, compare function bodies — classify as "exact" (identical) or "near" (identical AST structure with only identifier renames or argument-vs-closure differences; anything involving different control flow, different literals, or different called functions is NOT a near-match).
3. If bodies differ in meaningful ways (different edge-case handling, different return types), they are NOT duplicates — leave them alone.

When scanning for duplication:

- Flag functions with identical or near-identical bodies appearing in 2+ files.
- Threshold: 10+ lines of duplicated logic warrants extraction to a shared module.
- Only merge when the behavior is truly identical — similar-looking code with subtle differences should stay separate.
- Prefer a flat utility module over deep abstraction hierarchies.

**Runtime boundary awareness:** If duplication exists because of browser/Node isolation (e.g., browser code cannot import a Node module that uses `fs` or `zlib`), do NOT recommend "just import the Node version." Instead:

- Extract the pure-logic subset (no platform APIs) into a shared module that both environments can import.
- If the shared module would be trivial (< 5 lines), note the duplication as Info severity and move on.

Report duplicates with: function name, file locations, line counts, and whether they are exact or near-matches.

## 5. Enforce File Size Limits

**Large files are harder to review, test, and maintain.**

Flag files that exceed these thresholds:

| Metric | Warning | Action required |
|--------|---------|-----------------|
| Single file lines | > 400 | Flag for review |
| Single file lines | > 800 | Recommend splitting |
| Single function lines | > 80 | Flag for extraction |
| Single function lines | > 150 | Recommend splitting |

When recommending a split:

- Identify natural seams (e.g., separate concerns, independent state, distinct API surface).
- Prefer splitting by responsibility, not by arbitrary line count.
- Do not split if the file is a single cohesive unit (e.g., a binary format encoder/decoder).

## 6. Audit Dependencies

**Unused or redundant dependencies add install time, attack surface, and bundle size.**

Check for:

- `dependencies` or `devDependencies` not imported anywhere in source.
- Dependencies duplicating built-in functionality (e.g., `lodash.get` when optional chaining suffices).
- Dependencies only used in one place that could be trivially inlined.
- `dependencies` that should be `devDependencies` (build tools, test frameworks, bundlers).

**Classifying dependencies vs devDependencies:**

- If a package is `import`ed or `require`d at runtime → `dependencies`.
- If a package is only used by build scripts, test runners, or bundlers → `devDependencies`.
- If a package is not imported at runtime but its static files (e.g., `dist/`) are referenced by a bundler config (`pkg.assets`, `build.files`, `copy-webpack-plugin`) → it must stay in `dependencies` for the bundler to resolve it. Do not move it to devDependencies.

Tools: `npx depcheck`, `npx knip`, or manual grep for package names in `src/`.

For non-Node projects, substitute equivalent tooling: Python → `pip-check`/`vulture`; Go → `go mod tidy`/`staticcheck`; Rust → `cargo-udeps`. The dependency rules apply to the equivalent manifest file (e.g., `requirements.txt`, `go.mod`, `Cargo.toml`).

## 7. Check Version Control Hygiene

**Tracked files that should be ignored waste repo size and cause noise.**

Flag if any of these are committed:

- Build output (`dist/`, `build/`, `dist-electron/`, `.next/`, `.output/`, `out/`)
- Bundled assets that have a build script (e.g., `assets/bundle.js` when `npm run build` generates it)
- OS/editor files (`.DS_Store`, `Thumbs.db`, `.idea/`, `.vscode/` with user-specific settings)
- Secrets or credentials (`.env`, `*.pem`, `credentials.json`)

Recommend adding a `.gitignore` if one is missing. If artifacts are already tracked, note that `git rm --cached` is needed to stop tracking without deleting local files.

## 8. Keep Cleanup Surgical

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

## 9. Preserve Behavior Unless Asked Otherwise

**A slimming PR should usually be behavior-preserving.**

Before and after cleanup:

- Identify expected behavior that must stay the same.
- Run existing tests when available.
- Add or update tests only when needed to lock down behavior that cleanup could accidentally change.
- If tests are missing, use type checks, builds, search, or targeted manual checks as fallback verification.

If the cleanup intentionally changes behavior, state that clearly and treat it as a feature or bug-fix change, not pure cleanup.

## 10. Report Format

When finishing, output findings in this structure:

```
### [Severity] Category heading

| Location | Lines | Issue | Suggestion |
|----------|-------|-------|------------|
| file:line | N | description | action |
```

Group by severity (Critical → Warning → Info), then by category within each severity.

Required categories in the report (omit any category with no findings rather than including an empty section):

- **Hygiene issues:** missing `.gitignore`, tracked build artifacts, misplaced dependencies.
- **Deletable code:** files, functions, branches, imports proven dead or redundant.
- **Duplicates:** functions to consolidate, with file locations, line counts, match type.
- **Oversized files:** files/functions exceeding thresholds, with split suggestions.
- **Dependency issues:** unused, redundant, or misclassified packages.
- **Kept intentionally:** suspicious code that looked removable but was not proven safe to delete.
- **Verification needed:** what tests, builds, or manual checks are required before acting.

**End with a one-line summary** quantifying the total impact, e.g.:

> Summary: 3 critical, 5 warning, 2 info. Estimated reduction: ~320 source lines + 380 MB tracked artifacts.

Good cleanup leaves reviewers confident that less code now does the same job.
