---
name: end-of-life
description: Audit projects for end-of-life (EOL) software and prevent unsupported version recommendations using current endoflife.date lifecycle data. Use when choosing or recommending software versions, creating installation instructions, generating runtime or container configuration, reviewing dependencies and CI, or checking whether a project's runtimes, frameworks, databases, and operating systems remain supported.
---

# End of Life

Verify support against the current date before recommending a software version
or declaring a project free of EOL software. Use the public
[endoflife.date API v1](https://endoflife.date/docs/api/v1/) and the product
maintainer's support policy. Training knowledge, an old example, a successful
build, and an absence of reported vulnerabilities do not establish support.

These instructions work with any assistant that can read Markdown and retrieve
HTTPS content. Use whatever HTTP client, JSON parser, repository search, and
read-only runtime inspection are available in the host environment. Read
[references/api.md](references/api.md) when making requests or interpreting
the response fields.

## Check before suggesting versions

Apply this check within the user's task whenever introducing or recommending a
version in prose, code, commands, manifests, containers, CI, or documentation.
A recommendation needs the same evidence as an audit finding.

1. Identify the product, proposed version, relevant release cycle, and support
   provider. Fetch current lifecycle data before writing the recommendation.
2. Select a released, supported stable cycle compatible with the project's
   platform and dependencies. Prefer an appropriate LTS cycle when available;
   verify its current LTS phase and support horizon. Use the latest applicable
   patch in that cycle, verifying availability through official release or
   package metadata when needed.
3. Check the components introduced or used by the requested change: a supported
   application runtime can still use an EOL container OS, database, framework,
   or build tool. Include bundled tools when the proposed workflow uses them;
   state other coverage gaps without expanding into unrelated packages.
   Validate examples and copyable snippets as well as the surrounding explanation.
4. Give the chosen version, lifecycle source, check date, and relevant support
   deadline. Correct an EOL version supplied by the user or found in an example
   and offer a verified replacement. If compatibility prevents replacement,
   explain that constraint and provide a migration path.

Do not present an EOL release as suitable for a new deployment. An explicitly
requested historical example, reproduction, or legacy migration may require
one; label its support status and isolate that use from the recommended target.
If live verification fails, state that support is unverified and avoid inventing
a supported version or a lifecycle date.

## Audit an existing project

1. **Inventory the relevant surfaces.** Inspect manifests and lockfiles, runtime
   pins and toolchain files, container stages and base images, CI matrices and
   runner OSs, deployment/IaC configuration, service versions, and maintained
   setup documentation. Include direct and transitive packages where lifecycle
   information is available. Record each product, version or constraint, role
   (build/test/runtime), and source file with line number or runtime evidence.
2. **Establish what is actually used.** Prefer resolved lockfile entries, image
   digests/metadata, and runtime observations over loose declarations. Inspect
   the project's own execution environment when accessible; the assistant's
   host runtime is not proof of the deployed version. Distinguish observed,
   pinned, permitted, and documented versions. Resolve floating image tags,
   aliases, and version ranges where possible. A range permitting EOL software
   is a configuration risk, not proof that an EOL version is running.
3. **Map products and cycles precisely.** Discover product slugs through the API
   catalog and verify names, aliases, or package identifiers. Match the product's
   release scheme: some lifecycles use a major version, others a minor version,
   edition, vendor build, or dated release. Preserve version strings. Match
   component boundaries, never a raw prefix (`3.1` must not match `3.10`). A CI
   action tag is not the version of the runtime the action installs.
4. **Fetch and classify.** Fetch each distinct product once per audit and reuse
   its release data for all occurrences. Record the check date, `generated_at`,
   and source URL. Apply the support rules below; validate conflicts or missing
   coverage against the official vendor lifecycle. Use a 90-day upcoming-EOL
   warning horizon unless the user supplies a different horizon.
5. **Report or fix within scope.** For an audit, report findings and supported
   targets. For an authorized fix, update the relevant pins, manifests,
   lockfiles, images, CI, and maintained examples consistently, run appropriate
   compatibility checks, and recheck lifecycle status. Configuration edits alone
   do not establish that a running deployment has been upgraded.

Do not install or execute project code just to inventory it. Query only public
product identifiers and release cycles; keep private project files and secrets
local. Remote inspection, upgrades, migrations, and deployments require the
authorization appropriate to those actions.

## Interpret support correctly

Use today's date, not a hardcoded date. Treat the EOL date as a boundary requiring
verification; if the vendor's effective time is unclear, flag it rather than
asserting support through that day. If dates and booleans disagree, consult the
official policy before clearing the finding.

| Classification | Evidence and action |
| --- | --- |
| EOL | `isEol` is true or the applicable support deadline has elapsed. Flag it and recommend a verified supported target. |
| Supported | The release has shipped, `isEol` is false, and the relevant vendor policy supports this edition/platform. Ended active support (`isEoas`) can still mean security support continues. |
| Approaching EOL | A supported cycle ends within the warning horizon. Report days remaining and plan an upgrade; choose a target with enough support for the project's intended lifetime. |
| Support exception | Upstream EOL with verified distribution backports or extended support for the exact package, edition, platform, and entitlement. Report coverage, its deadline, and the upstream EOL status separately. |
| Unknown | Product/cycle not found, ambiguous mapping, invalid or unavailable data, or unresolved deployment version. Identify the evidence gap; never count it as supported. |

`isMaintained: true` can include commercial or extended support after upstream
EOL. It cannot clear an EOL finding by itself. `isEoes: false` describes an
extended phase, not proof that the project has access to it; null or missing
extended-support fields do not establish eligibility. Require evidence of
entitlement or vendor backport coverage before using a support exception.

`eolFrom: null` means the deadline is unknown, not that support lasts forever.
A current `isEol: false` can establish the cycle's present status, but report
the missing deadline. A discontinued product needs its vendor policy checked;
discontinuation alone does not describe all software security support phases.

Exclude prereleases and future releases from ordinary upgrade targets. Do not
equate the highest version, the API's `latest` endpoint, or an image's `latest`
tag with a suitable stable/LTS recommendation. Lifecycle support does not prove
that every old patch in a supported cycle still receives fixes or is secure.

## Evidence and completion

Report the audit scope and check date first. Use a concise table:

| Product / role | Version / cycle | Evidence | Status / deadline | Verified target | Lifecycle source |
| --- | --- | --- | --- | --- | --- |

Prioritize EOL findings, then upcoming deadlines and unknowns. Include affected
paths/lines, deployed evidence when available, support exceptions, and data
timestamps. Separate present deployment findings from stale documentation and
constraints that merely allow an EOL version.

Declare only the scope actually checked: for example, "No EOL cycles found in
the inspected runtime pins and container definitions." List unresolved products,
missing deployment observations, and untracked packages. endoflife.date is not
a complete package registry; use official lifecycle policies for missing
products. A product with no published lifecycle remains unverified, not EOL by
assumption. Prefer an applicable current vendor policy when it contradicts the
aggregator and explain the discrepancy with sources.

A recommendation is complete when the suggested stack and copyable examples
have current support evidence. A project audit is complete when all discovered
in-scope versions have a classification or an explicit evidence gap. An upgrade
is complete only to the extent its changes and actual runtime were verified.
