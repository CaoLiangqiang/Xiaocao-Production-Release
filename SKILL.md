---
name: production-release
description: Execute production releases end to end across repositories and package ecosystems. Use when the user asks to release, publish, ship, cut a version, create a GitHub Release, publish npm/PyPI/Cargo/Docker artifacts, or says "版本发布", "发版", or equivalent; includes versioning, changelog and docs, validation, commits and PRs, CI, merge, tags, registry publication, verification, and cleanup with minimal manual intervention.
---

# Production Release

## Overview

Own the release outcome the user requested instead of returning a checklist. Match execution to the requested terminal state: planning is read-only, local preparation stops before remote changes, preparing a release PR stops before merge and publication, and an explicit request to release, publish, or ship authorizes the repository's normal end-to-end release operations. Continue until that terminal state is reached and independently verified.

## Operating contract

- Execute every safe, in-scope step available through local tools, repository automation, hosting providers, and package registries.
- Stop at the requested terminal state. A smaller request such as a release plan, local preparation, draft PR, or draft hosting release does not inherit authority for later publication steps.
- Minimize user work. Do not ask the user to run routine commands that Codex can run.
- Prefer existing repository release policy and automation over inventing a parallel process.
- Preserve the user's checkout, unrelated worktree changes, and external state. Never silently include unrelated files or move the user's current branch to satisfy release cleanliness.
- Keep the user informed during long-running checks and publication, but do not stop for non-blocking preferences.
- Complete all possible preparation before requesting an unavoidable human gate.

An explicit release request does not authorize work beyond its stated terminal state, unrelated repositories, paid services, destructive history rewrites, package unpublishing, or new legal/public-disclosure decisions.

## 1. Set scope and establish a checkpoint

Before changing local or external state:

1. Classify the requested terminal state:
   - **Inspect or plan**: analyze and report without mutation.
   - **Prepare locally**: update and validate files, but do not push, open PRs, merge, tag, or publish.
   - **Prepare for review**: create the requested branch, commit, push, draft PR, or draft release, then stop before merge or publication.
   - **Release or publish**: execute the repository's normal end-to-end flow through consumer-side verification.
2. State the interpreted terminal state before the first mutation. Ask only when the wording supports materially different outcomes and the next action would cross an external or irreversible boundary.
3. Establish the release identity from evidence: repository, target branch and commit, package or artifact, version, channel, registry, and intended visibility.
4. Track completed external checkpoints such as pushed commit, PR, merge commit, tag, workflow run, published artifact and digest, and release URL. On resume, reconstruct this state from Git, the hosting provider, and the registry rather than relying on memory.
5. Before every irreversible action, refresh the relevant external state. If a command times out or returns an ambiguous result, query the remote system before retrying; a failed response does not prove the operation failed.
6. Treat an existing tag, release, or artifact as complete only when its commit, version, channel, metadata, and digest match the intended release. Stop on a conflict instead of overwriting or creating a look-alike release.

## 2. Discover the release system

Before editing or publishing:

1. Read repository instructions such as `AGENTS.md`, contribution guides, release docs, and CI workflows.
2. Inspect Git status, current branch, remotes, tags, default branch, and hosting authentication without switching or cleaning the user's checkout.
3. Detect the project layout, monorepo boundaries, package managers, lockfiles, language versions, and generated artifacts.
4. Locate every authoritative version source: manifests, lockfiles, source constants, schemas, docs, examples, and changelog.
5. Identify configured registries and delivery surfaces: GitHub/GitLab releases, npm, PyPI, crates.io, container registries, binaries, installers, or app stores.
6. Inspect existing release automation, trusted publishers, signing, branch protection, and required checks.
7. Confirm which worktree changes belong to the release. If mixed or ambiguous, isolate known release changes and ask only when ownership cannot be determined safely.
8. Choose a safe execution workspace. When the user's checkout is dirty, on another task, or not dedicated to the release, prefer a temporary worktree or clean clone at the verified target commit. Do not reset, clean, or switch the original checkout merely to obtain a clean release state.

Use platform-specific publishing skills and connectors when available. For GitHub branch, commit, push, and PR work, use the applicable GitHub publishing workflow rather than recreating it.

## 3. Determine the version

Choose the version in this order:

1. Use an explicit user-provided version.
2. Follow repository-managed versioning such as Changesets, release-please, semantic-release, Cargo workspace policy, or conventional commits.
3. Infer SemVer only when repository policy and the complete diff since the last published version make the impact unambiguous:
   - `patch`: backward-compatible fixes, packaging fixes, and small maintenance changes.
   - `minor`: backward-compatible user-visible features.
   - `major`: incompatible public behavior, API, schema, CLI, or migration changes.

Ask before an inferred major release, an unclear pre-release/stable-channel transition, a version that conflicts with repository policy, or a diff that mixes changes with unclear public impact. Do not infer from the working tree alone when it may omit other unreleased changes, and do not ask merely to restate a patch or minor version already established by clear repository evidence.

Never reuse, overwrite, move, or force-update a published version or release tag. Query the registry and remote tags immediately before publication.

## 4. Prepare release metadata

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

## 5. Run proportional, applicable quality gates

Use a clean or deterministic dependency installation where supported. Run repository-required checks and the release-specific checks that apply to the artifact, ecosystem, supported platforms, and risk of the change. Before running them, distinguish:

- **Required** gates defined by repository policy, branch protection, or publication automation.
- **Applicable** release checks selected because the corresponding artifact or risk exists.
- **Unavailable** checks that require inaccessible credentials, services, or supported platforms; report these as unverified rather than silently passing them.

Typical applicable checks include:

- formatting and whitespace checks;
- type checking, linting, unit, integration, and end-to-end tests;
- builds for supported runtime versions or platforms;
- dependency and supply-chain audit;
- package dry-run and exact file-list inspection;
- secret scan of the release diff and packaged artifact;
- isolated installation from the built artifact;
- executable/version/help and representative product smoke tests;
- workflow/configuration syntax validation.

Do not invent a new support matrix, scanner, or release toolchain solely to make the checklist longer. Use repository-provided tooling first and add infrastructure only when the release requirement actually needs it.

Ensure tests use temporary homes, registries, projects, or credentials and do not mutate real user data unless the user explicitly selected a live integration test.

Never bypass, suppress, or reinterpret a failed required gate as success. Diagnose and fix safe release defects, rerun the affected checks, and continue. Ask only when a failure requires a product or compatibility decision.

## 6. Publish through reviewed Git history

When the requested terminal state includes the corresponding remote Git operations, follow repository policy. Unless the repository defines another workflow:

1. In the isolated release workspace, create or reuse a focused release branch from the verified target branch commit.
2. Stage only intended files and inspect the full staged diff.
3. Commit concise release-preparation changes.
4. Push to the primary configured release remote and required mirrors.
5. Create a draft PR with scope, user impact, safety boundaries, validation evidence, and known limitations.
6. Wait for required CI. Fix failures and push follow-up commits.
7. Mark ready and merge only after required checks pass and the requested terminal state includes merge. A request to prepare a release PR stops before this step.
8. After merge, verify the remote default branch contains the exact merged commit. Update only the isolated release workspace when needed; do not switch, reset, or clean the user's original checkout.

Treat an explicit end-to-end release request as authorization for this normal PR and merge flow. Do not merge when review or branch policy explicitly requires another human approval.

## 7. Publish artifacts

Publish artifacts only when the requested terminal state includes publication. Create the tag only after the release commit is on the default branch. Prefer annotated or repository-standard signed tags. Require the tag version to match all manifests and runtime version output.

Immediately before tagging or publishing, re-query remote tags, hosting releases, registry versions, and channels. After each mutation, capture the resulting commit, URL, version, digest, or workflow run. On an ambiguous response, verify externally before deciding whether any retry is safe.

Use the repository's configured delivery mechanism:

- **npm**: run pack/publish dry-runs, inspect included files, require a new version, and prefer npm Trusted Publishing with OIDC. After trust is configured, never publish locally or maintain long-lived write tokens. Verify `dist-tags`, owner, license, repository metadata, and a clean registry install.
- **PyPI**: build sdist/wheel, inspect metadata, run package checks and isolated install, and prefer PyPI Trusted Publishing.
- **Cargo**: run tests, `cargo package`, inspect the crate, and publish only after registry/version availability checks.
- **Containers**: build from the release commit, test the image, publish immutable version tags plus the intended moving tag, and record the digest. Sign or attest when configured.
- **Git hosting releases**: create release notes from the verified diff, attach required artifacts and checksums, and distinguish stable from pre-release status.
- **Other ecosystems**: follow repository-native automation and apply the same immutable-version, least-privilege, artifact-inspection, and post-publication verification rules.

For a first publication, complete repository, package-name, license, visibility, owner, 2FA, and trusted-publisher setup. Human authentication may be necessary, but finish all non-authenticated preparation first and resume immediately afterward.

## 8. Verify the delivered release

Do not stop at a successful upload response. Verify from the consumer side:

1. Registry metadata shows the expected name, version, channel/tag, license, owner, and source repository.
2. The immutable tag points to the merged release commit.
3. The hosting release is public/internal as intended, non-draft, and has accurate notes.
4. A fresh environment can install or pull from the real registry without local paths or warm project caches.
5. The installed artifact reports the expected version and passes representative smoke tests.
6. Trusted publishing, provenance, signatures, checksums, and image digests are present when expected.
7. The default branch and all required release workflows are green.

Clean generated dependency directories, build output, local tarballs, temporary credentials, and release test directories created in the isolated release workspace when they are reproducible and not intended deliverables. Preserve the user's original checkout and branch. Report any pre-existing or release-related changes left there instead of deleting them.

## 9. Human gates and blockers

Request user action only for:

- login, browser approval, 2FA, CAPTCHA, hardware keys, or inaccessible signing keys;
- first-time public visibility or license decisions after explaining irreversible exposure;
- paid operations or new external accounts;
- destructive unpublish, package transfer, force-push, tag replacement, or history rewrite;
- ambiguous major/breaking version decisions;
- mandatory human review enforced by policy;
- product choices that cannot be inferred from repository evidence.

When blocked, state the exact completed work, one concrete action the user must take, and what will resume afterward. Never ask the user to send passwords, tokens, recovery codes, or one-time codes in chat.

## 10. Final handoff

Lead with whether the requested terminal state is complete. For preparation-only work, say explicitly that publication was outside scope rather than calling the release complete. Report as applicable:

- released version and distribution channel;
- merged commit, tag, and release/registry URLs;
- artifacts and supported platforms;
- validation and security results;
- publication automation and credential model;
- any remaining owner decision or known limitation;
- repository cleanliness and synchronization state.

Do not call a release complete while a required registry, tag, release page, workflow, or consumer-side verification remains pending.
