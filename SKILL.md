---
name: does-my-ci-suck
description: Audit and debug a repository's CI for slow feedback, failures, flakiness, wasted compute, missing checks, and visibility gaps. Use for a CI health review or evidence-backed improvement plan; implement changes only when requested.
---

# Does My CI Suck?

Find what prevents fast, trustworthy, reasonably priced CI feedback. Work from
the local repository, supplied logs, and available read-only provider access.
Support any CI provider and language; no particular hosting service, company
layout, paid tool, or credential store is required.

Default to diagnosis and suggestions. If the user also requests fixes, implement
the agreed scope under the repository's own rules. An audit does not authorize
workflow dispatches, cache deletion, billing changes, service enrollment, or
deployments. Existing authorization should be used without asking again.

## Establish the question and available evidence

Infer the repository and provider from the working directory, remotes, CI files,
and user-supplied run links. Read applicable repository instructions. Inspect
workflows, reusable actions/includes, called scripts, dependency locks, test
configuration, build files, and recent relevant changes. Do not execute repository
scripts or install dependencies merely to understand them.

Identify the actual objective: faster first failure, faster required checks,
lower compute cost, fewer flakes, reliable releases, or missing validation.
Use the user's target when supplied; otherwise state the observed baseline and
prioritize bottlenecks without inventing a universal acceptable duration.

Choose the strongest available mode and name its limits:

| Evidence available | What to do |
| --- | --- |
| Configuration only | Static audit. Explain what the pipeline is configured to do, coverage risks, and hypotheses. Timing, utilization, reliability, and savings remain unknown. |
| Existing run summaries/logs | Diagnose observed jobs/steps, failures, cache behavior, and the dependency path. Label incomplete or incomparable runs. |
| Repeated runs plus test/resource/cost data | Compare distributions and cohorts; identify recurring waste, flaky cases, and cost/latency tradeoffs. |
| Authorized experiments | Validate one scoped change on comparable CI runners; report retained and rejected results. |

Read [measurement.md](references/measurement.md) when collecting runs or when
visibility is insufficient. It includes local GitHub CLI examples and equivalent
evidence requirements for other providers. If authentication, logs, retention,
or network access is missing, continue the static audit and specify the smallest
missing evidence. Do not fabricate a healthy score or demand admin access.

## Trace correctness before speed

Map each required or advisory check to its real command and failure behavior.
Compare discovered suites/components to CI selection. Inspect tolerated failures,
shell pipelines, background commands, skipped jobs, merge-queue events, and
required-gate configuration where accessible. A green badge does not prove that
all tests ran or that a tool returning errors failed the gate.

For a failing run, find the first causal failure, then separate downstream skips
and cleanup errors. Classify it as application regression, test nondeterminism,
setup/dependency failure, resource exhaustion, or provider/infrastructure failure
only to the extent supported by evidence. A retry passing once is not proof of
a flake. Preserve first-attempt failures when evaluating reliability.

Keep validation scope, race/coverage checks, failure propagation, isolation,
security boundaries, and release gates intact. Surface existing coverage gaps
separately from optimization. Do not make CI faster by hiding failures, silently
quarantining tests, weakening assertions, disabling required checks, or replacing
fresh external-policy/security checks with a prior green result.

## Locate the constraint

Map the dependency graph and distinguish:

- Event-to-required-result latency and time to first actionable failure.
- Job execution and summed runner-minutes, including repeated jobs and attempts.
- Start offsets, actual queue waits if known, and manual approval waits.
- Checkout/toolchain, cache transfer/extraction, installation, generation/build,
  service setup, fixtures, test bodies, teardown, uploads, and post-job work.
- Reliability and verification completeness, independent of speed.

Rank work by contribution to the critical path or total cost for the stated goal.
Parallel job durations and aggregated test phases are not elapsed pipeline time.
Runner labels are not evidence of actual CPUs, memory, or utilization.

Read the relevant sections of [optimization-catalog.md](references/optimization-catalog.md)
to match the measured symptom to an experiment. The catalog includes successful,
failed, and superseded approaches as decision rules, not universal settings.
Check supported behavior against the repository's pinned tool versions.

When a tool/provider change could address the constraint, consult
[tools-and-providers.md](references/tools-and-providers.md). Explain the fit,
local/native alternative, migration cost, and uncertainty. Verify current official
documentation and pricing before a concrete recommendation. Vendor claims are
not measurements of this repository.

## Propose or run a bounded experiment

Use [experiments.md](references/experiments.md) for the comparison contract.
Prefer removing repeated work and measuring setup before adding shards, bigger
runners, more caches, or another service. This is a prioritization heuristic,
not a requirement to reject a clearly evidenced hardware constraint.

For each recommendation, give the observed fact, causal hypothesis, affected
phase, proposed change, correctness invariants, cost/risk, verification method,
and rollback condition. If data cannot establish a likely benefit, propose the
measurement first. Provide specific instrumentation changes for visibility gaps;
do not apply them during a read-only audit.

## Deliver an actionable report

Lead with an evidence-qualified verdict: healthy for the stated goal, specific
problems found, or insufficient visibility. No arbitrary numeric grade.

Include, at a depth proportional to the request:

1. **Scope and confidence:** repo/revision, provider, workflows, time range,
   attempts/sample size, evidence links or local file references, access gaps.
2. **Baseline:** required-check latency, compute usage, failures/retries, cache
   conditions, and cost basis where available; unknown fields stay unknown.
3. **Findings:** severity/priority, evidence, observed impact, suspected cause,
   and whether confirmed, inferred, or unmeasured. Keep coverage/reliability
   failures distinct from performance opportunities.
4. **Next actions:** a short ranked list of concrete changes or measurements,
   effort, tradeoffs, and an acceptance/rollback test. Identify the first useful
   next step even when no provider access exists.
5. **Experiment outcomes, if any:** baseline/candidate revisions and runs,
   complete-job versus phase results, cold/warm conditions, preserved checks,
   and rejected changes. State an unmet target plainly.

Estimate money only from explicit rates, billing units, frequency, and measured
usage. Separate gross usage, credits, invoice savings, and engineering/hosting
overhead. Never add overlapping improvements or extrapolate a faster package
into a faster pipeline without critical-path evidence.
