# Investigation Playbook

Concrete tactics for the **Investigation Ladder** (SKILL.md Phase D). The rule — _investigate before you escalate_ — is non-negotiable and lives in the skill body. This file is the how-to: what to actually search for at each tier, per dependency type. The goal is to convert "unclear → NEEDS_HUMAN_REVIEW" into "unclear → looked → now I know."

Climb tiers in order; stop the moment the **load-bearing fact** is pinned. A fact is load-bearing only if the verdict flips depending on its value — if the verdict is the same either way, note the uncertainty and move on, don't spend the ladder on it.

---

## Tier 0 — Re-read (most "unclear" is unread)

Before deciding anything is missing, confirm it actually is:

- Read the whole vulnerable function, not the snippet the finding quoted.
- Follow every internal callee one level deep; follow modifiers to their definitions.
- Read the storage variables it touches — where else are they written?
- Walk the inheritance chain (`is X, Y`) and check for overrides that change behavior.
- Re-check the finding's _premise_: AI findings frequently misquote a `require`, invert a comparison, or claim a function is external when it's internal. Verify the premise is even true before investigating its consequences.

A large share of NEEDS_HUMAN_REVIEW verdicts evaporate here.

---

## Tier 1 — The repository (grep beyond the named file)

The named file rarely contains the whole answer. Search the repo:

**Interfaces & implementations**

- The finding references `IWhatever` — find the _concrete_ implementation: `grep -rn "contract .* is .*IWhatever" .` or search for the function selector's body. Interface behavior is meaningless; implementation behavior is what matters.
- Libraries (`using X for`) and base contracts — read the actual math/logic they inject.

**Configuration & constants**

- `constructor`s, `initialize()`, and setter functions — these set the values a finding may be assuming. Grep for the state variable's name to find every writer.
- Constant declarations, `immutable`s, hard-coded addresses.

**Deployment reality**

- `script/`, `deploy/`, `*.s.sol`, hardhat deploy scripts, `foundry.toml` remappings — these reveal which tokens/oracles/params the team actually wires up.

**Tests are the intended-behavior oracle**

- `test/`, `*.t.sol`, `*.spec.ts` — tests encode what the developers _expect_ to happen and, crucially, which concrete tokens, oracles, decimals, and amounts they use. A test that sets up USDC (6 decimals) tells you the decimals question for free.
- **Mocks** (`Mock*.sol`, `test/mocks/`) reveal the assumed behavior of external contracts the team is designing against — often the exact behavioral question a finding hinges on.

**Cross-references**

- Grep the function/event/error name across the whole tree to find every caller and every guard that might sit upstream of the vulnerable entry point.

---

## Tier 2 — On-chain (verified source + real state)

If the protocol or its dependencies are **deployed**, the chain has ground truth the repo can't give you: actual config, actual proxy targets, actual token/oracle behavior. Use the block explorer / any available on-chain tooling.

**Find the deployment**

- Check the repo's README, `deployments/`, `broadcast/` (Foundry records real addresses here), or docs for addresses. If the user gave a target address, start there.

**Read verified source & real values**

- Pull the _verified_ implementation source from the explorer — it's the code actually running, which may differ from the repo branch.
- Read constructor args / current state: call view functions or read storage for the real decimals, heartbeat, owner, fee, cap, paused state, role assignments.

**Resolve proxies**

- If it's a proxy, find the implementation (EIP-1967 impl slot, or the explorer's "Read as Proxy"). Validate the finding against the _implementation's_ logic, and check the upgrade admin if the finding involves upgradeability.

**Concrete dependency behavior**

- The real token at the wired address: is it fee-on-transfer, rebasing, blocklisted, pausable, USDT-style approve? Read its verified source.
- The real oracle feed: its decimals, heartbeat, and whether it's the aggregator the protocol actually reads.

Real deployment config resolves the large class of findings that are only valid "if configured a certain way" — check how it's _actually_ configured.

If on-chain access isn't available in your environment, say so briefly and lean on Tier 1 + Tier 3; don't silently skip and then escalate.

---

## Tier 3 — The web (authoritative docs for standard deps)

For well-known dependencies, the behavior is documented — look it up rather than assuming.

- **Chainlink feeds**: the feed's page gives decimals, heartbeat/deviation, and deprecation status. Distinguish a _manipulable_ source (in scope) from a feed merely returning stale/bad data (OOS per the severity reference).
- **Tokens with quirks**: confirm fee-on-transfer / rebasing / blocklist / non-standard-return / decimals for the specific token (docs, the token's own explorer page).
- **External protocols**: Uniswap V2/V3/V4 pool mechanics, Aave/Compound interest and liquidation math, ERC-4337 semantics, LST/LRT redemption behavior — consult the protocol's own docs for the precise mechanic the finding depends on.
- **Standards**: EIP text for the exact required behavior when a finding claims a standards violation.

Cite what you found and where, so the human can re-check.

---

## When the ladder genuinely fails

Only after Tiers 0–3 have all failed to resolve a load-bearing fact does the finding become `NEEDS_HUMAN_REVIEW`. When that happens, be precise:

- State the **one** unresolved fact (not "it's complex").
- State **why it's load-bearing** (the verdict is VALID if the fact is A, INVALID if it's B).
- State **where you already looked** (repo: X, on-chain: Y, web: Z) so the human doesn't repeat your search.

This turns an escalation from a punt into a targeted hand-off — the human answers one crisp question instead of re-triaging from scratch. That precision is the difference between a triager that saves time and one that just reshuffles the pile.

---

## Efficiency notes for batch runs

- Resolve shared external facts **once** into the Phase C context ledger. If twenty findings touch the same price feed, investigate that feed a single time and reference the ledger entry.
- Investigate at the **cluster** level where possible: a root-cause cluster usually turns on one or two shared facts — pin those once for the whole cluster.
- Time-box per fact: if Tier 0–1 resolves it, don't go on-chain for sport. Reserve Tier 2–3 for facts the repo genuinely can't answer.
