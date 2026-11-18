# Datasploit recovery plan

This plan describes a practical route from the current, partially modernized
repository to a tool that a new maintainer can install, understand, and use
reliably today. It is a sequence of deliverables, not a promise that every
historical integration will be restored. Keep this file current: check off
work only when its acceptance criteria are met, and add findings or blockers
as repository issues rather than silently changing the scope.

## What "usable again" means

A first supported release should let a new user install Datasploit in an
isolated Python environment, configure it without exposing secrets, run a
small set of explicitly supported collectors against targets they are
authorized to investigate, understand failures and skipped integrations, and
get predictable output. The command-line interface is the initial product;
the historical web UI, database, alerting, and every old data source are not
release prerequisites.

The current entry point and collector layout support domain, email, IP, and
username inputs. Do not promise phone-number support unless it is implemented
and tested. The README and setup guide advertise Python 3.12, while some docs
and collector integrations describe older services or installation flows;
verify those claims before presenting them as supported.

## Work sequence

### 1. Agree on support and safety boundaries

- [ ] Maintainers: decide the initial supported Python version(s), target
  types, supported collectors, output formats, and maintenance owner.
- [ ] Maintainers: define acceptable-use guidance. State that users must have
  authorization, respect source terms and rate limits, and handle personal or
  credential-related data responsibly.
- [ ] Human or coding agent: inventory each collector's upstream service,
  authentication needs, current availability, terms/rate limits, data
  sensitivity, and whether it is passive or sends active requests.
- [ ] Maintainers: label each collector **supported**, **needs validation**,
  **disabled**, or **remove**. Keep integrations disabled until their behavior,
  legality, and current service/API are confirmed.

**Exit criteria:** a reviewed support matrix exists, default runs exclude
unverified or active collectors, and the project's acceptable-use boundaries
are documented.

### 2. Make a clean installation reproducible

- [ ] Test installation from a fresh clone in a clean Python 3.12 environment;
  record the exact commands and fix dependency or import failures.
- [ ] Choose one canonical dependency/install path. Keep `requirements.txt`,
  `Pipfile`, and their lock data consistent, or document which one is
  authoritative.
- [ ] Add a minimal, non-network smoke check that imports the package and
  entry point and displays `--help` without requiring API credentials.
- [ ] Confirm config creation/loading works from a fresh checkout and when
  launched outside the repository directory. Never commit user config or keys.
- [ ] Remove or clearly mark obsolete installation instructions (including
  Python 2-era/package-library claims) only after verifying the replacement.

**Exit criteria:** a new contributor can follow one documented install path;
the smoke check passes without credentials or network access; generated config
and reports stay out of version control.

### 3. Establish behavior checks for the core CLI

- [ ] Add focused automated tests for target classification, private-IP
  skipping, file input parsing, collector discovery, disabled collectors,
  missing prerequisites, and report selection.
- [ ] Test invalid and empty input, an unreadable target file, a collector
  import error, and an individual collector failure. Errors must be visible
  and must not prevent unrelated collectors from continuing where appropriate.
- [ ] Verify `-a` and `-q` flag behavior against their help text; correct either
  the implementation or documentation so the interface is unambiguous.
- [ ] Decide and test stable exit codes and output-directory behavior.

**Exit criteria:** core tests run offline and do not call third-party services;
the command's documented options and failure behavior match actual behavior.

### 4. Restore a small, reliable collector set

- [ ] For each candidate in the support matrix, validate current endpoints,
  authentication, response parsing, timeouts, rate limits, and failure
  messages using controlled test fixtures or authorized test targets.
- [ ] Add request timeouts and robust HTTP/status/JSON handling to supported
  collectors; ensure a broken upstream does not crash the whole scan.
- [ ] Keep TLS verification enabled. Review any existing certificate
  verification bypasses and remove them or document a justified exception
  before a collector is supported.
- [ ] Check that credentials are not placed in URLs, logs, reports, or
  exceptions; use supported authentication headers and redact sensitive data.
- [ ] Make collector enablement and required configuration accurate, and
  explain which features need accounts, paid plans, or API keys.
- [ ] Test only against targets and services where the tester has permission;
  avoid embedding real targets or credentials in fixtures and examples.

**Exit criteria:** each enabled-by-default collector has a current support
record, bounded network behavior, safe secret handling, and repeatable tests.
Unverified collectors remain explicitly disabled or are removed.

### 5. Make results useful and predictable

- [ ] Define a consistent result shape and report contract for collector
  output, including empty results, partial failures, and provenance/source.
- [ ] Test text reports for safe filenames, target-specific separation,
  predictable encoding, and no accidental overwrite of unrelated files.
- [ ] Decide whether JSON is part of the initial supported interface. If yes,
  implement and test it as structured output; do not imply HTML/JSON support
  that the current runner does not provide.
- [ ] Ensure console output distinguishes completed, empty, skipped, and
  failed collectors, and summarizes where reports were written.

**Exit criteria:** a user can tell what ran, what did not, why, and where
results are stored; documented output formats match working behavior.

### 6. Replace stale guidance with one current user path

- [ ] Update `README.md`, `docs/Usage.md`, `docs/setupGuide.md`, and API
  configuration docs to use the supported install, config, and CLI workflow.
- [ ] Remove or clearly archive conflicting legacy claims (including phone
  support, obsolete domains, old API setup links, and library entry points)
  after confirming they are no longer supported.
- [ ] Add a minimal example using a reserved/example target, explain that
  results depend on upstream services and credentials, and show how to run
  without optional integrations.
- [ ] Document configuration keys, report location, privacy implications,
  rate-limit expectations, and how to disable a collector.

**Exit criteria:** a first-time user can go from clone to a safe, successful
smoke run by following one path in the docs.

### 7. Release and maintain

- [ ] Add continuous integration for the supported Python version(s), offline
  tests, and basic style/static checks selected by maintainers.
- [ ] Define dependency update and vulnerability triage ownership; pin or
  constrain dependencies consistently and review updates before release.
- [ ] Tag a recovery release only after the clean-install, core-test, and
  documented smoke-run criteria above pass.
- [ ] Create a small issue list for deferred work: additional collectors,
  packaging/library support, richer reports, persistence, visualization, and
  alerting. Reassess each item before implementation rather than treating the
  historical roadmap as a commitment.

**Exit criteria:** a release can be reproduced from a clean environment and
has a maintainer, support policy, and a clear process for reporting breakage.

## How to work this plan

- Humans should make product, policy, and support decisions; agents should not
  infer that an old integration is safe or operational just because its code
  exists.
- Agents should take one bounded checkbox or closely related set per change,
  inspect current behavior first, avoid unrelated rewrites, add or update
  focused tests, and report files changed, commands run, results, and remaining
  uncertainty.
- Never use real API secrets or unapproved third-party targets in tests.
  Redact credentials and personal data from logs, sample output, and issues.
- Update this plan as work lands. A checked box means its stated exit criterion
  is verified, not merely that code was edited.
