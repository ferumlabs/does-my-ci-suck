# Match the remedy to the measured bottleneck

Read the sections relevant to the observed constraint. These are candidate
experiments, not a checklist of changes every repository should make. Lessons
include retained, rejected, and superseded implementations. No historical
runner size, worker count, cache mode, or speedup is a universal default.

## Scheduling and unnecessary work

| Symptom | Candidate improvement | What must remain true / how it can fail |
| --- | --- | --- |
| Two pipelines verify the same PR revision | Deduplicate topic-branch push and PR triggers | Preserve main, merge-queue, fork, manual, and release behavior that the repo needs; equivalent SHAs alone do not prove equivalent event inputs |
| Unrelated changes run every component | Dependency-aware affected selection; skip unaffected external deployments before creating them | Include shared libraries, locks, config, generators, workflows, and unknown diffs; preserve stable required gates; test affected and unaffected cases |
| Obsolete PR revisions consume capacity | Cancel superseded verification in appropriately scoped concurrency groups | Avoid cross-branch cancellation; active releases/migrations generally need serialization that preserves completion, not cancellation |
| Independent work waits unnecessarily | Remove false dependencies; overlap independent jobs or supported background steps | Preserve exit-status propagation and explicit waits; account for duplicate setup, CPU/I/O contention, and runner capacity |
| A warmer repeats the complete validation suite | Save useful caches from existing successful trusted-main jobs | Ensure cache publication happens after input changes, separate incompatible consumers, and preserve full miss fallback; maintenance schedules may still have another purpose |
| Jobs wait after prerequisites finish | Inspect provisioning, concurrency limits, runner availability, and capacity | A start offset is not pure queue time; faster code cannot fix an approval wait; additional capacity has an idle/operational cost |
| Huge matrices repeat equivalent work | Review which platform/version dimensions detect distinct regressions | Preserve supported compatibility coverage; changing coverage frequency is an explicit policy decision, not a free optimization |

Treat a required aggregate gate carefully: an intentionally unaffected component
can be successful, but a failed or unexpectedly skipped dependency must not be
reported as passed. Inspect effective includes and reusable workflows rather than
assuming the root YAML contains every trigger/check.

## Dependencies and caches

| Symptom | Candidate improvement | Correctness and measurement checks |
| --- | --- | --- |
| Repeated downloads | Package archive/module cache | Confirm the consumer uses that path; a download cache does not skip linking, extraction, native builds, or lifecycle scripts |
| Install remains slow on archive hits | Exact compatible installed-tree cache | Key OS/arch/ABI, runtime, manifests/lock/config, patches, install mode, and setup logic; prove workspace/local-source and lifecycle closure; keep current-source generation; fall back to full install when unproven |
| Extraction/linking dominates | Supported archive compression settings and project-local hardlinks | Less compression means more transfer; hardlinks can share writes from patch/build scripts; never share mutable trees between concurrent projects |
| Root install includes irrelevant packages | Supported focused/workspace installation | Preserve tooling, patches, code generation, test discovery, and transitive runtime dependencies; production-only installs may omit required checks |
| Late cache-key calculation scans expanded trees | Compute compatibility identity once before generated dependencies exist | Include the full input closure; reuse an explicit output rather than accidentally changing key semantics |
| Many jobs publish the same archive | One eligible writer and restore-only consumers; optional upload overlap | Losing writers can still pay compression cost; cache creation must not gate unrelated validation or accept untrusted input |
| Cache download can stall longer than rebuilding | Supported bounded download/segment timeout with full fallback | A reported key match is not completed restoration; preserve real install/build failures |
| Cache is huge with little reusable work | Split by actual consumer, prune regenerable output, compare no-cache baseline | Separate actual producer directories, not just key names; distinguish race/non-race, toolchain and target flags |
| Keys never refresh or always miss | Correct hit outputs, key inputs, trusted writers, compatible fallback and eviction policy | Immutable keys may preserve incomplete work; invalid fallback can silently restore wrong artifacts; separate tool install, compilation, and test-result reuse |
| Frontend rebuild repeats valid work | Supported incremental/framework filesystem build cache | Directory restore alone may not enable caching; validate pinned-version flags, completed outputs, transitive imports, env/config, changed inputs, errors and fixes |

Measure restore + extraction + save + downstream work against the no-cache
alternative. Distinguish summed phase work from elapsed time when uploads or
other phases overlap; use the observed job/gate endpoint for wall-clock impact.
Cache identity must include all output-affecting inputs; secrets
must not be stored in caches. Untrusted PRs must not poison caches consumed by
privileged jobs. Do not cache approval, vulnerability-feed results, or externally
mutable state as if source identity made them current. Task-result caches need a
stronger deterministic-input contract than dependency download caches.

Past experiments rejected an extra cache because warm savings were about one
second and cold runs regressed. That is a reason to measure transfer and hit
frequency, not to forbid installed caches everywhere.

## Test infrastructure, fixtures, and waits

| Symptom | Candidate improvement | Correctness boundary |
| --- | --- | --- |
| Replaying migrations dominates ordinary tests | Generated flat schema for ordinary setup | Verify freshness/schema equivalence; retain real incremental migration and upgrade-compatibility tests |
| Recreating schema for every case | Preserve initialized schema; reset mutable data and identities | Include sequences, empty tables with advanced identities, sessions, failed/interrupted cleanup, and isolation; do not assume rollback covers application commits or multiple connections |
| Repeated real-migration setup for independent cases | Migrate once, clone isolated template databases | Preserve per-case state and cleanup ordering; prove template equivalence and keep tests of migration execution itself |
| Database/container startup repeats across packages | Bounded worker/package resource pool | Lock across processes when needed; keep per-case data isolation and cleanup; tune pool and package concurrency separately |
| Parsing/seeding immutable data repeats | Memoize parsed catalogs or immutable fixture corpora | Keep mocks, clocks, clients, writable objects and stores fresh; prove copies/immutability; constructor and mutation tests must still exercise fresh inputs |
| Sleeps/backoff dominate | Event-driven synchronization or supported virtual time | Preserve ordering, cancellation, deadline and race semantics; a timeout ceiling is not necessarily time actually spent |
| Test-specific ticks are too slow | Inject shorter intervals where behavior is equivalent | Do not shorten arbitrary waits until flakes disappear or alter production defaults; prefer deterministic synchronization |
| HTTP server sockets dominate setup | Supported in-memory transports/test servers | Preserve transport/proxy routing; keep real-socket coverage where TLS, network behavior, streaming or integration semantics require it |
| Pure tests initialize DOM/browser environments | Lightweight default plus explicit environment selection | Inspect transitive imports, not filename extensions; retain actual DOM/browser coverage |
| CPU idle while tests queue on a pool | Tune package workers, in-test concurrency and service slots independently | Measure CPU/memory/I/O and contention; global clocks/schemas/mocks can make naive parallelism unsafe |
| Test bodies remain the critical path | Native parallelism, then balanced sharding if useful | Include each shard's setup and scheduling costs; prove complete/disjoint test discovery, reporting/coverage merge, and isolation |
| Intermittent failures create retries | Diagnose seed/order, state leaks, timing assumptions, resource pressure and external dependencies | Keep first-attempt outcomes; retries are containment, not a fix; quarantine requires explicit policy, ownership and a restoration condition |

An immutable fixture improvement can speed up several packages while leaving
the full suite unchanged because another package is the bottleneck. Likewise,
thread pools can lose to process pools through startup, contention or isolation
overhead. Retain the mode supported by repeated complete-job measurements.

## Generation, linting and tool replacement

- Measure generator execution separately from repeated package loading or
  compilation. Prefer supported source-input modes when they preserve interface
  closure, embedded types, imports and generated output. Keep package mode for
  cases the source parser cannot represent. Remove proven-unused generated
  artifacts and their generation directives, not the underlying tested contract.
- Compare native batching/worker controls before custom orchestration. Package
  loading, plugins and configuration may repeat in every worker. Extra lint
  workers have regressed real pipelines; a CPU count is not a speedup guarantee.
- Inspect resolved lint configuration after presets/overrides. Remove duplicate
  checks only after rule/options/resolver and failure-case equivalence is proven.
  Typed lint caches must invalidate for transitive type/import changes.
- Consider faster formatters, linters, package managers or compilers only when
  the measured phase matters. Check custom plugins, language coverage, ignores,
  formatting churn, lifecycle scripts, lockfiles, diagnostics and exit behavior.
  Run a bounded comparison with intentional failures; document every semantic
  gap. Do not silently remove rules to make a migration green.
- Prefer official supported versions/options over maintaining dependency patches.
  A framework diagnostic-cache patch can add upgrade/correctness burden even
  when it once helped. Installation/toolchain pinning and eliminating downloads
  may be cheaper than replacing the toolchain.

## Docker and release paths

- Break image jobs into context transfer, dependency resolution, compilation,
  image output/registry push, and cache import/export. Preserve the full source,
  embedded/generated input, build-argument and target-platform closure.
- Use supported layer caching where reuse exceeds its cost. Compare `mode=min`
  and `mode=max` when intermediate-layer export dominates. Measure cold,
  unchanged warm, changed source, changed dependency and embedded-input cases.
  Minimal export has helped some pipelines and can hurt others.
- A Docker layer cache does not imply mutable `RUN --mount=type=cache` contents
  persist between ephemeral runners. Verify actual consumers and persistence
  before adding host-cache restores or cache-mount transfer machinery.
- Prebuilt dependency/toolchain images can reduce setup but require input
  invalidation, trusted publication, vulnerability freshness and maintenance.
  Prefer simple cache policies when the base-image lifecycle costs more.
- Build one release candidate and promote its immutable digest through stages
  instead of rebuilding identical inputs. Preserve fresh required validation,
  provenance, architecture/config correctness and release gates; verify the
  deployed digest. A mutable tag is not proof of artifact identity.
- Batch repeated remote infrastructure-output queries into validated structured
  reads. Fetch only necessary fields; don't print secrets. Skip redundant system
  tool installation only when the runner's installed version meets requirements.
- Native/mobile runtime fingerprints can avoid unnecessary binary rebuilds, but
  must include native modules, patches, build properties and compatibility inputs.
  Source-count reduction alone is not evidence of fewer build minutes.
- Memory-oriented compiler settings may make a build fit a smaller machine.
  Measure memory, speed and output/diagnostic tradeoffs; disabling sourcemaps
  loses debugging information and is not a free default optimization.

## Runner, artifact and repository costs

- Right-size each job from measured effective CPUs, peak memory and runtime.
  Light orchestration rarely benefits from the largest test runner. Smaller
  runners can increase gate latency; larger ones can lower cost if speed scales
  enough. Include queue/provisioning, architecture compatibility and rate changes.
- Self-hosted, managed and spot runners are alternatives, not automatic upgrades.
  Account for patching, isolation, untrusted PRs, credentials, idle capacity,
  interruptions/retries, data locality and operation burden. A vendor benchmark
  cannot establish this repository's economics.
- Cap artifact/log sizes and retention. Prefer compact success summaries and
  fuller failure diagnostics where appropriate; retain required release,
  compliance and debugging artifacts. A faster upload can reduce post-job time.
- Remove unused current-tree assets and generated output; consider appropriate
  checkout depth/LFS behavior only if the build/versioning/diff logic permits it.
  Deleting a file does not remove it from Git history. History rewriting is a
  separate disruptive operation, not an incidental CI audit recommendation.

## Primary documentation to check against pinned versions

- [Cache behavior and trust restrictions](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [Concurrency controls](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency)
- [Docker cache backends](https://docs.docker.com/build/cache/backends/)
- [Docker GitHub Actions caching and cache mounts](https://docs.docker.com/build/ci/github-actions/cache/)
- [Yarn compression and linking configuration](https://yarnpkg.com/configuration/yarnrc)
- [Node setup caching](https://github.com/actions/setup-node)
- [Go test flags](https://pkg.go.dev/cmd/go#hdr-Testing_flags)
- [Go virtual-time testing](https://pkg.go.dev/testing/synctest)
- [Vitest environments](https://vitest.dev/config/environment)
- [Next.js CI build caching](https://nextjs.org/docs/app/guides/ci-build-caching)
- [Typed lint performance](https://typescript-eslint.io/troubleshooting/typed-linting/performance/)
- [ESLint worker tradeoffs](https://eslint.org/blog/2025/08/multithread-linting/)
