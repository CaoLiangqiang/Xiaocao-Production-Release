---
name: production-release
description: Execute production releases end to end across repositories and package ecosystems. Use when the user asks to release, publish, ship, cut a version, create a GitHub Release, publish npm/PyPI/Cargo/Docker artifacts, or says "版本发布", "发版", or equivalent; includes versioning, changelog and docs, validation, commits and PRs, CI, merge, tags, registry publication, verification, and cleanup with minimal manual intervention. Do not use for ordinary commit, push, or PR work unrelated to a release, or for Git explanations.
---

# Production Release

## Overview

Own the release outcome instead of returning a checklist. Treat an explicit request to release, publish, or ship as authorization for the repository's normal versioning, branch, commit, PR, merge, tag, registry, and release operations. A request only to inspect, prepare locally, create a release PR, or create a draft release authorizes work through that named stage and stops before later merge, tag, or publication steps. Continue until the requested outcome is reached and independently verified.

## Operating contract

- Execute every safe, in-scope step available through local tools, repository automation, hosting providers, and package registries.
- Keep this workflow self-contained for the user: do not ask them to invoke a separate Git publishing skill to finish a release.
- Minimize user work. Do not ask the user to run routine commands that Codex can run.
- Prefer existing repository release policy and automation over inventing a parallel process.
- Preserve unrelated worktree changes and external state. Never silently include unrelated files.
- Keep the user informed during long-running checks and publication, and report durable identifiers or URLs as external state is created, but do not stop for non-blocking preferences.
- Complete all possible preparation before requesting an unavoidable human gate.

An explicit release request does not authorize unrelated repositories, paid services, destructive history rewrites, package unpublishing, or new legal/public-disclosure decisions.

## 1. Discover the release system

Before editing or publishing:

1. Read repository instructions such as `AGENTS.md`, contribution guides, release docs, and CI workflows.
2. Inspect Git status, current branch, remotes, tags, default branch, hosting authentication, and the available connector or CLI capabilities.
3. Detect the project layout, monorepo boundaries, package managers, lockfiles, language versions, and generated artifacts.
4. Locate every authoritative version source: manifests, lockfiles, source constants, schemas, docs, examples, and changelog.
5. Identify configured registries and delivery surfaces: GitHub/GitLab releases, npm, PyPI, crates.io, container registries, binaries, installers, or app stores.
6. Inspect existing release automation, trusted publishers, signing, branch protection, and required checks.
7. Inspect staged, unstaged, untracked, and already-committed unreleased changes. Confirm that the scope being reviewed is exactly the scope that will be staged, packaged, and published. If ownership is mixed or ambiguous, isolate known release changes and ask only when it cannot be determined safely.

Use available platform connectors and repository-native tools without transferring ownership of the release to another user-invoked workflow. For GitHub, use local Git for branch creation, staging, commits, and pushes; prefer the GitHub connector for PR lookup, creation, and status; use `gh` only when connector coverage is insufficient, such as authentication diagnostics or fork and cross-repository PR semantics. Do not require both paths when one authenticated path can complete the task.

Before the first mutation, briefly report the resolved release plan: release scope, intended version or inference, source branch and commit, delivery channels, Git review path, planned checks, and requested stopping point. Continue without a separate confirmation when the plan follows repository evidence and the user's explicit request; ask only when Section 8 identifies a genuine human gate.

## 2. Determine the version

Choose the version in this order:

1. Use an explicit user-provided version.
2. Follow repository-managed versioning such as Changesets, release-please, semantic-release, Cargo workspace policy, or conventional commits.
3. Infer SemVer from the release diff:
   - `patch`: backward-compatible fixes, packaging fixes, and small maintenance changes.
   - `minor`: backward-compatible user-visible features.
   - `major`: incompatible public behavior, API, schema, CLI, or migration changes.

Ask before an inferred major release, an unclear pre-release/stable-channel transition, or a version that conflicts with repository policy. Do not ask for a clear patch or minor version merely because the user omitted the number.

Never reuse, overwrite, move, or force-update a published version or release tag. Query the registry and remote tags immediately before publication.

## 3. Prepare release metadata

Update all version sources atomically. Do not assume a manifest-only version command covers source constants or docs.

Maintain as applicable:

- package/workspace manifests and lockfiles;
- source/runtime version constants;
- changelog with the actual release date;
- compatibility, migration, upgrade, and deprecation notes;
- README installation examples and supported-version statements;
- package repository, issue tracker, license, executable, file-list, and publish metadata;
- release notes written from the actual diff and user impact.

Do not select a new license, make a private repository public, or expose previously private source without explicit informed consent. Do not add credentials or registry tokens to files or hosting secrets when tokenless trusted publishing is available.

## 4. Run proportional quality gates

Use a clean or deterministic dependency installation where supported. Derive validation commands from repository instructions, manifests, task runners, and CI configuration. If no trustworthy command exists for a relevant check, report it as unverified with the reason instead of inventing one. Run all repository-defined checks plus release-specific checks:

- formatting and whitespace checks;
- type checking, linting, unit, integration, and end-to-end tests;
- builds for supported runtime versions or platforms;
- dependency and supply-chain audit;
- package dry-run and exact file-list inspection;
- secret scan of the release diff and packaged artifact;
- isolated installation from the built artifact;
- executable/version/help and representative product smoke tests;
- workflow/configuration syntax validation.

Ensure tests use temporary homes, registries, projects, or credentials and do not mutate real user data unless the user explicitly selected a live integration test.

Never bypass, suppress, or reinterpret a failed required gate as success. Diagnose and fix safe release defects, rerun the affected checks, and continue. Ask only when a failure requires a product or compatibility decision.

## 5. Publish through reviewed Git history

Unless the repository defines another workflow:

1. Resolve the base and head branches. If the current non-default branch contains only the intended release work, keep it and reconcile it with the freshly fetched default branch according to repository policy. Otherwise create a focused release branch from the latest remote default-branch commit. Use a user-provided branch name when given; otherwise follow repository convention or derive a concise name from the release scope.
2. Stage only intended files and inspect the full staged diff.
3. Commit concise release-preparation changes using a user-provided message or repository convention; otherwise derive it from the staged diff.
4. Push to the primary configured release remote and required mirrors, establish upstream tracking when absent, and verify the remote head resolves to the exact locally validated commit.
5. Query for an existing PR from the resolved head to base before creating one. Resume or update a matching PR instead of creating a duplicate; otherwise create a draft PR.
6. Wait for required CI. Fix failures and push follow-up commits.
7. Mark ready and merge only after required checks pass.
8. Synchronize a clean local default branch to the exact merged commit.

Treat an explicit end-to-end release request as authorization for this normal PR and merge flow. Do not merge when review or branch policy explicitly requires another human approval.

For GitHub PRs, derive the repository from the push remote, the head from the current branch, and the base from an explicit user choice or the remote default branch. When a fork or cross-repository target is involved, choose a tool that can represent the qualified head repository and branch instead of guessing. Prefer the GitHub connector after the branch exists remotely; fall back to `gh pr create` or `gh pr view` only when necessary. If CLI fallback needs a multiline PR body, pass a temporary Markdown file so headings and newlines are preserved.

Use a concise PR title that summarizes the complete release diff. The body should explain what changed, why it changed, user or developer impact, the root cause when the release fixes a defect, release and safety boundaries, validation evidence, and known limitations.

Before pushing, recheck status and diff after validation. If a formatter, generator, build, or test changed files, include only intended changes and rerun affected checks so the exact validated commit is the one pushed. Use broad staging such as `git add -A` only when the entire worktree is confirmed in scope. If branch updates or release changes conflict, resolve only when the intended result is unambiguous from repository evidence; otherwise preserve recoverability, list the conflicted files, and request one concrete decision.

## 6. Publish artifacts

Create the tag only after the release commit is on the default branch. Prefer annotated or repository-standard signed tags. Require the tag version to match all manifests and runtime version output.

Treat a timeout or ambiguous response from a remote, hosting provider, or registry as unknown state. Query the external system before retrying so an operation that actually succeeded is not repeated.

Use the repository's configured delivery mechanism:

- **npm**: run pack/publish dry-runs, inspect included files, require a new version, and prefer npm Trusted Publishing with OIDC. After trust is configured, never publish locally or maintain long-lived write tokens. Verify `dist-tags`, owner, license, repository metadata, and a clean registry install.
- **PyPI**: build sdist/wheel, inspect metadata, run package checks and isolated install, and prefer PyPI Trusted Publishing.
- **Cargo**: run tests, `cargo package`, inspect the crate, and publish only after registry/version availability checks.
- **Containers**: build from the release commit, test the image, publish immutable version tags plus the intended moving tag, and record the digest. Sign or attest when configured.
- **Git hosting releases**: create release notes from the verified diff, attach required artifacts and checksums, and distinguish stable from pre-release status.
- **Other ecosystems**: follow repository-native automation and apply the same immutable-version, least-privilege, artifact-inspection, and post-publication verification rules.

For a first publication, complete repository, package-name, license, visibility, owner, 2FA, and trusted-publisher setup. Human authentication may be necessary, but finish all non-authenticated preparation first and resume immediately afterward.

## 7. Verify the delivered release

Do not stop at a successful upload response. Verify from the consumer side:

1. Registry metadata shows the expected name, version, channel/tag, license, owner, and source repository.
2. The immutable tag points to the merged release commit.
3. The hosting release is public/internal as intended, non-draft, and has accurate notes.
4. A fresh environment can install or pull from the real registry without local paths or warm project caches.
5. The installed artifact reports the expected version and passes representative smoke tests.
6. Trusted publishing, provenance, signatures, checksums, and image digests are present when expected.
7. The default branch and all required release workflows are green.

Clean generated dependency directories, build output, local tarballs, temporary credentials, and release test directories when they are reproducible and not intended deliverables. Leave the repository on a clean, synchronized default branch.

## 8. Human gates and blockers

Request user action only for:

- login, browser approval, 2FA, CAPTCHA, hardware keys, or inaccessible signing keys;
- first remote-repository creation or selection when no release remote is configured, including the owner, repository name, and visibility;
- first-time public visibility or license decisions after explaining irreversible exposure;
- paid operations or new external accounts;
- destructive unpublish, package transfer, force-push, tag replacement, or history rewrite;
- ambiguous major/breaking version decisions;
- mandatory human review enforced by policy;
- product choices that cannot be inferred from repository evidence.

When blocked, state the exact completed work, one concrete action the user must take, and what will resume afterward. Never ask the user to send passwords, tokens, recovery codes, or one-time codes in chat.

## 9. Final handoff

Lead with whether the release is complete. Report:

- released version and distribution channel;
- release branch, PR target and URL, merged commit, tag, and release/registry URLs;
- artifacts and supported platforms;
- validation and security results;
- publication automation and credential model;
- any remaining owner decision or known limitation;
- repository cleanliness and synchronization state.

Do not call a release complete while a required registry, tag, release page, workflow, or consumer-side verification remains pending.
