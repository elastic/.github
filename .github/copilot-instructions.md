# High-signal code review policy

When reviewing a pull request, default to leaving no comment. Comment only when the changed code introduces a concrete correctness, security, data-integrity, compatibility, reliability, rollout, or operability defect.

When multiple issues compete for attention, prioritize in this order: (1) concrete correctness, security, or data-integrity defects; (2) architectural or design problems that compound over time; (3) missing test coverage for changed behavior. Leave the highest-priority comments first.

Do not comment on wording, spelling, formatting, naming preferences, syntax, style, import ordering, or other concerns that a compiler, formatter, linter, schema validator, test, or CI check should report. Do not restate a failing automated check. Do not leave praise, summaries, optional refactors, generic best practices, or speculative suggestions.

Before leaving a comment, verify all of the following:

- The issue is introduced or made materially worse by this pull request.
- You can describe a specific failure scenario and its impact.
- The claim is grounded in the changed lines and their direct imports — do not search or grep the broader codebase to construct a finding not visible in the diff.
- The repository's current code and configuration support the claim.
- The comment identifies the affected path and gives an actionable correction or asks for concrete missing evidence.
- The expected value of the comment is high enough to interrupt a human reviewer.

If any condition is missing, remain silent. Express uncertainty only when the uncertainty itself is a material risk that the author can resolve with specific evidence. On a re-review after new commits, do not repeat a comment on a line that has not changed since the prior run.

Use the repository's `.github/skills/code-review/SKILL.md` for domain-specific review checks when it exists. Copilot review is advisory and does not replace required human approval.

## Trust-boundary and supply-chain changes

Always inspect a diff that changes CI, build, release, dependency, credential,
network, or security configuration. Leave a review comment when the change
introduces or materially broadens a capability without a concrete, repository-
supported need and an appropriate control. In particular, flag:

- broader token, role, cloud, repository, or secret permissions than the changed
  operation requires;
- new collection, copying, logging, caching, artifact upload, or persistence of
  credentials or secret-bearing worker state;
- privileged workflows that check out, download, or execute untrusted pull-
  request code, artifacts, build scripts, hooks, or dependency lifecycle scripts;
- new outbound destinations, telemetry, uploads, remote shells, or network
  services unrelated to the repository's documented responsibility;
- downloaded or remotely referenced code, actions, reusable workflows, images,
  plugins, tools, or binaries that are executed without immutable identity and
  integrity verification;
- new dependencies or build inputs that are not resolved immutably;
- weakened or bypassed tests, approval gates, secret scanning, signature or
  checksum verification, TLS verification, sandboxing, or least-privilege
  defaults;
- shared caches, workspaces, or non-ephemeral runners that allow an untrusted or
  earlier build to influence a privileged or later build;
- unexplained executable binaries, encoded or obfuscated payloads, generated
  code without a documented generator, or functionality unrelated to the
  repository's stated purpose.

For GitHub Actions, require third-party actions and reusable workflows to be
pinned to a full commit SHA. Require container images by digest when they execute
in CI, build, release, or production paths. Require downloaded executables or
scripts to be pinned to an immutable version and verified with an expected
cryptographic digest, signature, or attestation before execution; HTTPS or a
checksum downloaded from the same mutable source is not sufficient integrity
evidence. Reject pipe-to-shell execution of unverified remote content.

For package-manager dependencies, require the repository's lockfile or equivalent
immutable resolution to be committed and enforced. Do not comment on a version
range merely because it is a range when an enforced lockfile pins the resolved
artifact. Do comment when a new executable build or runtime dependency can float
without reviewed immutable resolution.

Describe the exact new trust path, the realistic abuse or compromise scenario,
and the smallest corrective control. Do not allege malicious intent; when code
appears unrelated or concealed, ask for concrete purpose and provenance and
recommend removal when neither is established.

## Missing tests for material behavior

Check whether changed behavior has meaningful deterministic coverage. Comment
when the diff adds or changes a material correctness, security, compatibility,
failure-handling, state-transition, permission, parsing, retry, concurrency,
build, or release behavior and no existing or new test would fail if that
behavior regressed. A bug fix should normally include a regression test that
fails for the original defect.

Before commenting, inspect the relevant existing tests, including higher-level
and generated-output tests. Do not request tests for documentation-only changes,
mechanical refactors with unchanged behavior, generated files whose source and
generation are already tested, or behavior already exercised at an appropriate
layer. Do not demand a unit test when an existing integration or end-to-end test
more directly covers the contract.

A missing-test comment must name the unverified behavior and a concrete scenario
and assertion that would catch a realistic regression. Do not leave a generic
"add tests" comment or prescribe test-file structure. When deterministic testing
is genuinely impractical, ask for specific reproducible verification evidence
and the reason it cannot be automated.
