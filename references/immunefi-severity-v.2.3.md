# Immunefi V2.3 — Smart Contract Severity Reference

Consult this when assigning an **impact category** (Phase D step 4) and when considering an **out-of-scope INVALID**. The severity _matrix_ (impact × likelihood) lives in SKILL.md; this file is the impact taxonomy and the out-of-scope list.

## Impact categories

Pick the **highest applicable** category the finding's realized impact maps to. The category answers "what is the worst that happens IF this triggers," independent of how likely triggering is.

| Level        | Impacts                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Critical** | Manipulation of governance voting result. Direct theft of any user funds (at-rest or in-motion), other than unclaimed yield. Direct theft of user NFTs (at-rest or in-motion), other than unclaimed royalties. Permanent freezing of funds. Permanent freezing of NFTs. Unauthorized minting of NFTs. Predictable/manipulable RNG resulting in abuse of principal or NFT. Unintended alteration of what an NFT represents. Protocol insolvency. |
| **High**     | Theft of unclaimed yield. Theft of unclaimed royalties. Permanent freezing of unclaimed yield. Permanent freezing of unclaimed royalties. Temporary freezing of funds. Temporary freezing of NFTs.                                                                                                                                                                                                                                              |
| **Medium**   | Smart contract unable to operate due to lack of token funds. Block stuffing. Griefing (no direct profit motive, but damage to users or protocol). Theft of gas. Unbounded gas consumption.                                                                                                                                                                                                                                                      |
| **Low**      | Contract fails to deliver promised returns, but does not lose value.                                                                                                                                                                                                                                                                                                                                                                            |

### Notes on applying the table

- **"Theft of funds" vs "theft of unclaimed yield"** is the most common mis-map. Principal/at-rest user funds → Critical. Yield not yet claimed → High. Be precise about _what_ is stolen.
- **Permanent vs temporary freezing**: permanent freezing of principal → Critical; temporary freezing of principal → High; permanent freezing of _unclaimed yield_ → High. The reversibility axis you assessed in impact maps directly here.
- **Griefing** (damage with no attacker profit) tops out at Medium on impact alone — but if the same bug also enables a profitable theft or a permanent freeze, map to that higher category instead.
- **Insolvency** (protocol can't cover its liabilities) is Critical even if no single user is "stolen from" in one tx.
- The privilege/interaction downgrade clause: _"If the exploit requires elevated privileges or uncommon user interaction, the level may be downgraded or the finding rejected."_ Apply only when the requirement is **genuinely uncommon**. A bug reachable through a normal protocol entry point is not downgraded just because it's clever.

## Default out-of-scope (safe INVALID with cited category)

These are the _explicit_ cases where INVALID is safe — cite the specific category. Everything else needs a code-level contradiction to be INVALID.

- Impacts requiring attacks the reporter has _already executed_ on mainnet.
- Impacts requiring leaked keys or credentials.
- Impacts requiring privileged-address access **without** additional modification to those privileges (i.e. "the admin can rug" is centralization risk, out of scope — unless the finding shows an _unprivileged_ path to gain the privilege).
- External stablecoin depeg **not** directly caused by a protocol bug.
- Best-practice recommendations.
- Feature requests.
- Test files and configuration files — **unless the program states otherwise**. (Check scope; some programs include them.)
- Phishing / social engineering.
- Incorrect data from third-party oracles. **This does NOT exclude oracle _manipulation_ / flash-loan attacks** — manipulating a manipulable oracle is in scope; a feed simply returning bad data on its own is not.
- 51% attacks and similar basic economic/governance attacks.
- Lack-of-liquidity impacts.
- Sybil attacks.
- Centralization risks (trusted-role misbehavior with no unprivileged escalation path).

### Out-of-scope edge cautions

- **Centralization**: the line is whether an _unprivileged_ actor can reach the harmful state. "Owner can set fee to 100%" = centralization (OOS). "Anyone can call `setFee` because the modifier is missing" = a real access-control bug (in scope). Trace the actual guard before calling centralization.
- **Oracle**: distinguish "the feed reported a wrong price" (OOS) from "an attacker moved a spot price / manipulated a TWAP the protocol trusts" (in scope). The presence of a manipulable price source the protocol reads is the in-scope signal.
- **Privileged access**: if the finding demonstrates a path for a _non-privileged_ caller to obtain the privilege (e.g. an unprotected `initialize`, a role-grant with a missing check), it is **not** the OOS "requires privileged access" case — it's a real bug.

Always confirm against the specific program's published scope if you have it; a program can override these defaults in either direction.
