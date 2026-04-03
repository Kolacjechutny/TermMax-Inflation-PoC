# TermMax V2 - Vault Accounting Inflation Exploit

## Vulnerability Overview
A critical accounting flaw exists in `OrderManagerV2.afterSwap` where the **principal** change of a swap is incorrectly added to the vault's **annualized interest**. Near maturity, a time-based multiplier is applied to this principal, causing an exponential and artificial spike in the vault's `totalAssets()` and share price.

## Impact
**Severity: Critical**
An attacker can drain 100% of the vault's assets (USDC/DAI/etc.) by wash-trading with a vault-owned order just before maturity. The attacker redeems a small number of shares for a massive portion of the vault's real liquidity, stealing from other depositors.

## Root Cause
In `contracts/v2/vault/OrderManagerV2.sol:afterSwap`, the code adds `deltaFt` directly to the `_annualizedInterest`:
```solidity
uint256 deltaAnnualizedInterest = (ftChanges * 365 days) / (maturity - block.timestamp);
_annualizedInterest += deltaAnnualizedInterest;
```
If `maturity - block.timestamp` is 1 hour, the principal is multiplied by **8,760**. This "phantom interest" is then accrued into `totalAssets()`, which determines the share redemption value.

## Reproduction Steps
1. Fork Arbitrum Mainnet.
2. Setup a target vault and order.
3. Attacker deposits a small amount to get shares.
4. Warp time to 1 hour before maturity.
5. Attacker performs a swap in the order to trigger the inflation.
6. Attacker redeems shares for a profit, draining other users' funds.

## Reproduction Command
```bash
export ARB_RPC_URL=<your_arbitrum_rpc_url>
forge test --match-contract VaultInflationArbForkPoC -vv
```
