# Compare changes without fooling yourself

An audit produces this plan; it does not automatically run it. Match experiment
cost and scope to existing user authorization. Prefer normal candidate CI over
extra full-suite reruns when it answers the question. Do not benchmark by
flushing shared caches, taking over another run, or exposing release credentials.

## Experiment record

Record:

- **Hypothesis:** the measured phase and why this change should affect it.
- **Baseline/candidate:** immutable revisions, run links and attempts; exact
  command, test discovery/counts, race/coverage flags, toolchain, runner image,
  CPU/memory, services, worker limits, and cache keys/states.
- **Invariants:** checks and assertions retained, input closure, failure
  propagation, test isolation, branch trust, and deploy/artifact identity.
- **Comparison:** one meaningful variable when possible; name bundled changes
  when they cannot be separated. Do not attribute the whole bundle to one lever.
- **Budget and success bar:** number of runs, elapsed/compute cost, required
  improvement, acceptable variance, and stopping/rollback conditions.
- **Result:** targeted phase, entire job and required-check latency, runner
  usage, failures, cache/artifact bytes and transfer cost, and remaining bottleneck.

Use comparable CI hardware rather than a personal laptop. Prefer interleaved or
reversed A/B order to reduce drift. Include a fresh/new-key case, at least three
warm samples per candidate/baseline when noise matters, and representative
changed-source/dependency inputs. Explicitly label a smaller sample as tentative.
An unchanged warm rerun is not a normal changing-code workload. A “new key” with
fallback restored is not fully cold.

For caches, prove miss fallback, exact restoration, invalidation, error behavior,
and recovery after fixing an error. An expected invalid-input failure helps
prove the check still executes; require the intended diagnostic, not any failure.
For generation/builds, compare meaningful outputs or semantics, allowing known
nonreproducible metadata. For fixture/database reuse, verify repeated execution,
failure cleanup, identity reset, and concurrent isolation.

Compare median and range, not the fastest run. Report end-to-end effects even
when they are zero. If a package drops from 27s to 6s but another package gates
the job, report the package win without claiming a pipeline win. A cheaper
runner can be slower; show both dimensions. Warm gains that lose on frequent
cold runs may be a net regression.

## Cost and attribution

Use the actual provider's billing units and current rates:

`estimated compute cost = sum(billable units per job × applicable rate)`

Include retries, cancellations already billed, recurring warmers, minimums,
cache/storage/network charges, licenses, idle hosts, and maintenance where
relevant. State credits/discounts separately. Monthly projections need observed
run frequency and workload mix; model cold and warm proportions explicitly.

Deleting a 14-minute duplicate job and adding one minute of cache publication
saves roughly 13 runner-minutes in that observation, not necessarily 13 minutes
of developer wait. These are illustrative numbers, not a benchmark guarantee.

Classify the conclusion as controlled comparison, observational before/after,
local microbenchmark, projection, or unmeasured. Keep a rejected experiment's
reason and conditions for revisiting it. Do not retain complexity for a noisy
one-second gain, repeatedly rerun until green, or extrapolate vendor speedups.
