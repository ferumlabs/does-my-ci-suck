# Tools and providers: choose by the gap

Reference links checked November 5, 2026. Capabilities, versions, pricing,
quotas and product names change: verify the current official source before
recommending a specific plan or migration. This list is illustrative, not a
ranking, endorsement, or claim that these services were used in the experiments.
None is required to run the skill.

Start with the existing CI provider's logs, API, summaries, test reports and
billing export. A workflow edit or a structured reporter may solve the problem
without another subscription. Recommend an external service only when it fits
an observed constraint and the benefits justify migration and ongoing costs.

| Need | Options and official links | Evaluate before recommending |
| --- | --- | --- |
| Inspect existing runs locally | [GitHub CLI](https://cli.github.com/manual/gh_run_view); [GitLab pipeline efficiency](https://docs.gitlab.com/ci/pipelines/pipeline_efficiency/); [CircleCI Insights](https://circleci.com/docs/guides/insights/insights/) | Available permissions, retained history, step/test granularity, event/attempt semantics; prefer these before buying observability |
| Catch workflow configuration errors | [actionlint](https://github.com/rhysd/actionlint) for GitHub Actions; the selected provider's native validation | Static validation cannot prove test coverage, runtime correctness, or speed; use existing pinned tooling rather than downloading arbitrary latest binaries |
| Pipeline traces and cross-run bottlenecks | [Datadog CI Visibility](https://docs.datadoghq.com/getting_started/ci_visibility/) | Provider support, instrumentation/export overhead, retention, data access and ingest pricing; local bounded spans are the smaller alternative |
| Per-test reliability, duration and splitting | [Buildkite Test Engine](https://buildkite.com/docs/pipelines/configure/tests) | Framework/report integration, supported CI source, stable identities, ingestion cost; collect native JUnit/JSON first; do not enable test skipping/quarantine implicitly |
| Different managed GitHub Actions compute/cache | [Blacksmith](https://www.blacksmith.sh/) | Actual runner OS/architecture, isolation, cache locality, network, startup, compatibility, support and current total cost; compare with right-sized native runners |
| Ephemeral GitHub runners in your AWS account | [RunsOn](https://runs-on.com/) and [pricing model](https://runs-on.com/pricing/) | AWS access/operations, spot interruption and fallback, concurrency, regional capacity, storage/network and license costs; managed runners may be simpler |
| Remote container builds and persistent build caching | [Depot](https://depot.dev/) | Whether context upload, compilation or export is the constraint; architectures, secrets/provenance, cache behavior, egress, trust, cost and fallback; first compare native BuildKit policies |
| Repeated monorepo task execution | [Nx affected tasks](https://nx.dev/docs/features/ci-features/affected), [Nx remote caching](https://nx.dev/docs/features/ci-features/remote-cache), [Turborepo remote caching](https://turborepo.dev/docs/core-concepts/remote-caching) | Migration/configuration cost, deterministic inputs/outputs, transitive dependencies, environment values, remote cache trust and data contents; simple path selection may suffice |
| Measured lint/format bottleneck | [Oxlint](https://oxc.rs/docs/guide/usage/linter.html), [Biome migration](https://biomejs.dev/installation/migrate-from-other-tools/) | Rule/plugin/language coverage and formatting semantics against the installed tools; benchmark full jobs and keep uncovered required checks |
| Understand hosted-runner spending | [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions), or the current provider's billing documentation/export | Account plan, job-level rounding, OS/runner rates, included usage/credits, storage and retries; do not hardcode prices in this skill |

For a recommendation, return a short comparison:

1. Observed problem and why this option could address it.
2. Smallest native/local change to compare against, including doing nothing.
3. Compatibility and trust requirements; migration, operation and data costs.
4. A bounded benchmark using the same commands, revision, runner resources,
   cache conditions and output mode where possible.
5. Acceptance/rollback criteria and what remains unverified.

Do not reproduce vendor percentage savings as expected results. Do not sign up,
purchase capacity, install a GitHub App, upload private source/logs, or change a
runner pool merely because it appears in this reference. Those are implementation
steps governed by the user's requested scope. Bare infrastructure hosts and spot
VMs are raw capacity; they are not equivalent to a maintained CI runner service.
