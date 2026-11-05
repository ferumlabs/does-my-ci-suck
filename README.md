# Does My CI Suck?

An agent skill for reviewing CI: failures, slow jobs, wasted compute, missing
checks, and missing visibility. It reads your repository and available run
history, then suggests what to fix or measure next.

Built from CI work and experiments across Ferum v5, Ferum v6, Dibs, and other
projects at Ferum Labs. It includes what worked and what did not. The skill has
no company-specific dependencies and does not require a paid service.

## Install

Clone this repository into your agent's skill directory. For Codex:

```sh
git clone https://github.com/ferumlabs/does-my-ci-suck.git ~/.codex/skills/does-my-ci-suck
```

For other agents, use their supported skill directory. Keep `references/`
beside `SKILL.md`. This is an instruction skill, not a standalone executable.

## Use

Open the repository you want reviewed and ask:

> Use $does-my-ci-suck to review this repository's CI and suggest improvements.

Add a run URL, sanitized logs, or a cost/latency target if you have one. Without
run history, it reviews configuration and explains what evidence is missing.
The default audit is read-only. Ask explicitly if you want changes implemented.

[SKILL.md](SKILL.md) is the entrypoint. [Does My CI Suck.md](Does%20My%20CI%20Suck.md)
is an alias. The [tool and provider notes](references/tools-and-providers.md)
cover options for specific bottlenecks, with tradeoffs and official links.

## Contributing

Open an issue or pull request with a specific problem or improvement.

- For performance claims, include the command, tool versions, runner specs,
  cache conditions, sample count, and before/after job timings. Say when a result
  is an estimate or only a local benchmark. Failed experiments are useful too.
- Keep advice generic. Explain when a change helps, when it can hurt, and how
  to check that tests and required checks still run.
- Link official documentation for tool or provider claims. Remove secrets and
  private data from examples. Keep edits short; skip promotional copy.

Before submitting, check relative links and try the changed guidance on a
representative repository or a small, sanitized example. Describe what you tested.
