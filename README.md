# TermMax V2 - Vault Accounting Inflation Exploit (Fork PoC)

## Vulnerability Overview
A critical accounting flaw exists in `OrderManagerV2.afterSwap` where the **principal** change of a swap is incorrectly added to the vault's **annualized interest** accounting. Near maturity, this principal is multiplied by a time-based factor `(365 days / time_to_maturity)`, causing the vault's `totalAssets()` and share price to inflate exponentially. An attacker can trigger this inflation via a swap and then redeem their shares for a significant profit, draining other users' funds.

## Impact
- **Critical**: Theft of all funds in any TermMax V2 Vault.
- An attacker can inflate the share price by millions of percent near maturity.
- This PoC demonstrates an attacker stealing 100,000 USDC from a victim on an Arbitrum fork.

## Affected Contract
- `OrderManagerV2.sol` (specifically the `afterSwap` function)
- `TermMaxVaultV2.sol` (inherits the flawed accounting logic)

## Reproduction
The PoC forks the Arbitrum mainnet and uses real protocol code and real USDC.

### Prerequisites
- [Foundry](https://getfoundry.sh/) installed.
- A valid Arbitrum RPC URL.

### Command
```bash
# Navigate to the PoC directory
cd termmax-inflation-poc

# Set your RPC URL
export FORK_URL=https://arb1.arbitrum.io/rpc

# Run the test
forge test --match-contract TermMaxArbForkExploit -vv
```
<img width="841" height="362" alt="Screenshot_2026-04-03_20_25_13" src="https://github.com/user-attachments/assets/2cc2d668-eaf1-4344-9b7a-245efbbec346" />


## PoC walkthrough
1. **Fork State**: The test forks Arbitrum at the current block.
2. **Setup**: Deploys a TermMax Market and Vault using the real protocol bytecode.
3. **Victim Deposit**: A victim deposits 100,000 USDC into the vault.
4. **Attacker Entry**: The attacker deposits 50,000 USDC.
5. **Vulnerability Trigger**: 10 seconds before maturity, the attacker performs a swap (selling 500,000 FT).
6. **Accounting Spike**: The vault incorrectly adds the 500,000 FT principal to the annualized interest.
7. **Theft Verification**: At maturity (10 seconds later), the attacker's shares have inflated from 50,000 USDC to ~216,000 USDC, effectively stealing the victim's 100,000 USDC.

## Remediation
Ensure that only the **interest** portion of a swap (not the entire principal) is added to `_annualizedInterest`, or implement strict bounds on how much the annualized interest can change per swap relative to the remaining time to maturity.

# please update the mappings to your need, thank you (-:
