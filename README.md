
# Alchemix contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Q&A

### Q: On what chains are the smart contracts going to be deployed?
For xalAssets 
xalAssets are Layer 2 alAssets that are compatible with Connext's xERC20 system, also referred to as xTokens. Optimism and Arbitrum are currently live. Future plans include Layer 2 chains such as Metis, Linea, and Base. There are two xalAssets - xalUSD and xalETH (when already in the context of Layer 2, they are simply referred to as alUSD and alETH).

Reward router/collector
The reward router/collector is only currently intended to be deployed on Optimism, Arbitrum, and Ethereum. 

___

### Q: If you are integrating tokens, are you allowing only whitelisted tokens to work with the codebase or any complying with the standard? Are they assumed to have certain properties, e.g. be non-reentrant? Are there any types of <a href="https://github.com/d-xo/weird-erc20" target="_blank" rel="noopener noreferrer">weird tokens</a> you want to integrate?
xalAsset
xalAssets are tokens. Connext has mint/burn control via the xERC20 standards and whitelisting of the Connext bridge. 

Reward router/collector
The reward router/collector will only interact with ERC20 tokens sent to the vault and queued by the multisig for distribution. Distributed tokens will be sold on an integrated dex for alUSD or alETH.

___

### Q: Are the admins of the protocols your contracts integrate with (if any) TRUSTED or RESTRICTED? If these integrations are trusted, should auditors also assume they are always responsive, for example, are oracles trusted to provide non-stale information, or VRF providers to respond within a designated timeframe?
Gelato - Gelato is trusted. Gelato can be unresponsive - if gelato is unresponsive, the harvest will fail. This is acceptable as Alchemist admins can manually trigger harvests, and users will still accumulate credit/yield while waiting for a harvest.

Connext - Connext is trusted to operate properly, be responsive, and not steal funds or mint unbacked assets.

___

### Q: Are there any protocol roles? Please list them and provide whether they are TRUSTED or RESTRICTED, or provide a more comprehensive description of what a role can and can't do/impact.
xalAssets
Admin role - trusted, can update the whitelist and map of bridges
Whitelist - allows contracts to mint the alAsset, including Alchemist contracts and bridges. It is assumed the Alchemist is safe/trusted, as all security functions to ensure backed minting of alAssets are covered by the Alchemist itself (which is already audited and live, and not in scope of this audit). Bridges are assumed safe/trusted, as security checks for minting/burning are done by the bridge itself, which is not in the scope of this audit.
Map of Bridges - Each whitelisted bridge is mapped to its rate limits

Reward router/collector
The router contract has an admin/owner.  The admin/owner is restricted to sweeping tokens and updating the list of collectors. It should be assumed that this is always a trusted admin (multisig). The router calls the collector, which the gelato keeper calls as part of yield harvests. The gelato keeper is the only contract that can call harvests. 

All Contracts
Contracts are upgradeable, so the audit scope would not include any economic damage a malicious administrator could cause from upgrades. 

___

### Q: For permissioned functions, please list all checks and requirements that will be made before calling the function.
xalAsset - Admins need to verify that addresses and rate limits are correct when updating the whitelist and map of bridges.

Router Contract - Admins need to verify that addresses are correct prior to adding connector contracts to the list of connectors.

___

### Q: Is the codebase expected to comply with any EIPs? Can there be/are there any deviations from the specification?
xalAssets
The xalAsset codebase is expected to comply with Connext's xERC20 standard.

The only deviation is that the old alAsset contracts have their own mint/burn functions that separate mint()/burn() and mintFrom()/burnFrom(). mint()/burn() is how a user mints/burns alAssets to/from their account. mintFrom()/burnFrom() is how a user gives another contract the right to mint/burn alAssets on their behalf. 

Connext xERC20 merges all of these functions into mint() and burn(). Therefore, the canonical alAsset contract was modified as follows:

1. The burn() function in the alAsset contract was renamed to burnSelf(), with no other changes. The transmuter references this function, so it is essentially a legacy function only used for the Transmuter. 
2. burnFrom() was renamed to burn(), and functionality was added so that the new burn() function can handle burns from yourself and someone else, including connext. 

Mint() and mintFrom() functions did not require any changes.

Reward router/collector
Not designed to comply with any EIPs.

___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, arbitrage bots, etc.)?
xalAssets
Connext monitors bridging off-chain and can pause bridging 

Reward router/collector
The gelato keeper will call the harvester contract, which calls the harvest function on the Alchemist for regular yield AND calls the reward router. The router holds the reward tokens that the DAO intends to distribute and sends those tokens to the collector. The collector receives those funds and checks if there are any other funds to claim from a 3rd party protocol (ie, 3rd party rewards), then the collector swaps all the funds into alAssets and donates to the depositors.

___

### Q: Are there any hardcoded values that you intend to change before (some) deployments?
No.
___

### Q: If the codebase is to be deployed on an L2, what should be the behavior of the protocol in case of sequencer issues (if applicable)? Should Sherlock assume that the Sequencer won't misbehave, including going offline?
It is acceptable to assume the sequencer won't misbehave or go offline.
___

### Q: Should potential issues, like broken assumptions about function behavior, be reported if they could pose risks in future integrations, even if they might not be an issue in the context of the scope? If yes, can you elaborate on properties/invariants that should hold?
These issues should be reported for xalAssets. They need not be reported for the reward router/collector contracts. 
___

### Q: Please discuss any design choices you made.
xerc20
The past deployment of the bridging portion of alAssets on L2 is a fork of Frax's original bridge contracts. The contracts are being upgraded to be compatible with xERC20. See the “deviations from specifications” section above to describe the deviation in burn() functions. 

The state variables of the old contracts cannot be deleted when upgraded, so the state variables of the old contracts related to the old bridging system could not be removed. The old functions have not been removed in case any oversights need to be undone related to the old state variables. Therefore, auditors should:
1. Make sure the combined modifications to the burn-related functions do not create any issues
2. Determine if old state variables can create any issues
3. Ignore any issues caused by using the old system as intended and/or disabling it. Auditors can assume the old bridge system will not be used, and only xERC20 is used moving forward. 

There is a crossChainCanonicalBase and an alchemicalTokenBase that are combined to create the xalAssets. The crossChainCaonicalBase has all of the old bridging functions and state variables that are not intended to be used, while alchemicalTokenBase has the new xERC20 functions and the standard alAsset functions.

Reward router/collector
1, One could in theory merge some collectors, but there is no large benefit in doing so
2. Originally there was one reward collector, but as more became necessary the router was added as a hub contract so that the Alchemist can always call the same contract but new reward tokens can be added indefinitely.

___

### Q: Please list any known issues/acceptable risks that should not result in a valid finding.
1. It should be assumed that the old system is not intentionally used anymore. Therefore, any report noting issues with using the old system to bridge assets would be out of scope. However, if the functions/state variables of the old system can affect or create issues with the intended functionality of the new xERC20 bridge system including minting unbacked alAssets, OR if the old system can be accessed and used by anyone other than the owner/admin, the report is in scope.
2. Operational risks of using the old bridging contracts should not result in a valid finding
3. It should be assumed the dex’s used for reward router contracts do not have vulnerabilities themselves
4. The router contract has a max slippage, each collect has getExpectedExchange() to understand the expected amount out using this slippage. It is a known risk that due to the nature of public keeper transactions, MEV sandwich attacks could result in the maximum slippage being realized.

___

### Q: We will report issues where the core protocol functionality is inaccessible for at least 7 days. Would you like to override this value?
No - 7 days is acceptable.
___

### Q: Please provide links to previous audits (if any).
https://alchemix-finance.gitbook.io/user-docs/audits
___

### Q: Please list any relevant protocol resources.
https://alchemix-finance.gitbook.io/user-docs/resources
___

### Q: Additional audit information.
The primary difference between the battle-tested alAssets on Mainnet and the L2 xalAssets is the retired bridge system and the new xERC20 bridge system. This is the purpose of this audit - to ensure the upgrade to xERC20 is not creating any issues with the alAsset. 

The reward router/collector has a very specific purpose—to receive a token from the DAO or 3rd party incentives, convert it to an alAsset, and then donate that alAsset to increase depositors' yield.

___



# Audit scope


[v2-foundry @ 846b83257b5e64af2885dca7f82d5ff5ed47ecb7](https://github.com/alchemix-finance/v2-foundry/tree/846b83257b5e64af2885dca7f82d5ff5ed47ecb7)
- [v2-foundry/src/AlchemicTokenV2Base.sol](v2-foundry/src/AlchemicTokenV2Base.sol)
- [v2-foundry/src/CrossChainCanonicalAlchemicTokenV2.sol](v2-foundry/src/CrossChainCanonicalAlchemicTokenV2.sol)
- [v2-foundry/src/CrossChainCanonicalBase.sol](v2-foundry/src/CrossChainCanonicalBase.sol)
- [v2-foundry/src/interfaces/IRewardCollector.sol](v2-foundry/src/interfaces/IRewardCollector.sol)
- [v2-foundry/src/interfaces/IRewardRouter.sol](v2-foundry/src/interfaces/IRewardRouter.sol)
- [v2-foundry/src/libraries/TokenUtils.sol](v2-foundry/src/libraries/TokenUtils.sol)
- [v2-foundry/src/utils/RewardRouter.sol](v2-foundry/src/utils/RewardRouter.sol)
- [v2-foundry/src/utils/collectors/OptimismRewardCollector.sol](v2-foundry/src/utils/collectors/OptimismRewardCollector.sol)


