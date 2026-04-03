# TermMax V2 - Vault Accounting Inflation Exploit

## Vulnerability Overview
A critical accounting flaw exists in `OrderManagerV2.afterSwap` where the **principal** change of a swap is incorrectly added to the vault's **annualized interest** accounting. Near maturity, this principal is multiplied by a time-based factor `(365 days / time_to_maturity)`, causing the vault's `totalAssets()` and share price to inflate exponentially. An attacker can trigger this inflation via a small swap and then redeem their shares for a significant profit, draining other users' funds.

## Affected Contract
- `OrderManagerV2.sol` (specifically the `afterSwap` function)
- `TermMaxVaultV2.sol` (inherits the flawed accounting logic)

## Exploit Proof of Concept
The PoC demonstrates:
1. Victim deposits 100,000 DAI.
2. Attacker deposits 50,000 DAI.
3. Attacker waits until 10 seconds before maturity.
4. Attacker performs a large swap (selling FT) to trigger the `afterSwap` callback.
5. The vault's annualized interest spikes by millions of percent.
6. After waiting 10 seconds (until maturity), the attacker's shares are worth more than the entire vault's initial balance.

## Reproduction Command
To run the local PoC:
```bash
# Navigate to the PoC directory
cd termmax-inflation-poc

# Run the test
forge test --match-contract TermMaxInflationExploit -vv
```

## Remediation
Ensure that only the **interest** portion of a swap (not the entire principal) is added to `_annualizedInterest`, or implement strict bounds on how much the annualized interest can change per swap relative to the remaining time to maturity.

# please update the mappings to your need, thank you (-:
