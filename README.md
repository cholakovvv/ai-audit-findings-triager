# ai-audit-findings-triager

`[BETA]` A [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) skill that triages AI-generated Solidity audit findings against the actual code, sorting real bugs from false positives and assigning Immunefi V2.3 severity, so you review a triaged report instead of raw tool output.

## What It Does

- **Dedupes and clusters** the findings, then validates each against the source (and on-chain state where it matters)
- **Investigates the repo, deployed contracts, and docs** to resolve a finding itself, flagging for human review only when a fact genuinely can't be pinned down
- **Verifies each verdict from both sides** (exploit it / break it) before committing to valid or false positive
- **Assigns Immunefi V2.3 severity** and writes `triager-issues.md`: a summary dashboard plus one evidence-backed entry per finding

## Requirements

- Claude Code (Opus or Sonnet)
- The codebase in context

## Installation

```bash
cd ~/.claude/skills
git clone https://github.com/cholakovvv/ai-audit-findings-triager.git
```

Restart Claude Code, then verify with `/skills`.

## Usage

```
Using the ai-audit-findings-triager skill, triage these findings against ./src.
Findings: ./ai-report.md
```

For more context on findings that turn on real config, add the deployment so it can check on-chain state:

```
Deployed at 0x... on ethereum.
```

It dedupes, validates, self-verifies, and writes `triager-issues.md`. Confirm the verdicts; it's a starting point, not a replacement for your judgment.

## What It Won't Do

- **Rubber-stamp the tool.** Re-derives every claim from code; a confident writeup proves nothing.
- **Reject a bug it can't explain.** "Couldn't confirm it works" is `NEEDS_HUMAN_REVIEW`, never a false `INVALID`.
- **Guess external behavior.** Unknown token / oracle / config → finds out, or flags what's missing.
- **Write PoCs.** Triage only. For that, see [foundry-poc-mainnet-fork](https://github.com/cholakovvv/foundry-poc-mainnet-fork).

## License

TBD.

## Credits

Built by [Simeon Cholakov](https://github.com/cholakovvv/), blockchain security researcher.

If this skill saves you a triage pass, consider:

- Starring the repo
- Tagging me on [X](https://x.com/cholakovvv) when it catches something on a real audit
