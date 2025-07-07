# Notional Exponent contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the **Issues** page in your private contest repo (label issues as **Medium** or **High**)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Q&A

### Q: On what chains are the smart contracts going to be deployed?
Ethereum, in the future we will consider Base or Arbitrum
___

### Q: If you are integrating tokens, are you allowing only whitelisted tokens to work with the codebase or any complying with the standard? Are they assumed to have certain properties, e.g. be non-reentrant? Are there any types of [weird tokens](https://github.com/d-xo/weird-erc20) you want to integrate?
The system is expected to work with standard ERC20 tokens as well as non-standard tokens like USDT.
___

### Q: Are there any limitations on values set by admins (or other roles) in the codebase, including restrictions on array lengths?
There are three privileged roles in the system which are whitelisted in AddressRegistry.sol

- upgradeAdmin: allowed to upgrade contracts with a 7 day timelock (see: TimelockUpgradeableProxy) and list withdraw request managers and allowed vaults for withdraw managers, lending routers, and update the reward tokens via the RewardManagerMixin. 
- pauseAdmin: allowed to pause the system and selectively whitelist selectors
- feeReceiver: receives accrued fees from vaults

upgradeAdmin and pauseAdmin should not be able to bypass the restrictions set out in TimelockUpgradeableProxy.
___

### Q: Are there any limitations on values set by admins (or other roles) in protocols you integrate with, including restrictions on array lengths?
No
___

### Q: Is the codebase expected to comply with any specific EIPs?
No
___

### Q: Are there any off-chain mechanisms involved in the protocol (e.g., keeper bots, arbitrage bots, etc.)? We assume these mechanisms will not misbehave, delay, or go offline unless otherwise specified.
No
___

### Q: What properties/invariants do you want to hold even if breaking them has a low/unknown impact?
Each vault must remain solvent with respect to the amount of yield tokens and vault shares. In other words, the convertYieldTokenToShares and convertSharesToYieldToken should return the proper values to ensure that every token holder can redeem their funds appropriately. This should remain true with respect to pending withdraw requests which create "escrowed shares".

Any escrowed shares should not accrue fees or rewards while they are escrowed.

Accounts can only have a position that is natively held on the contract or in a single lending router. An account cannot have both a native balance and a position in a lending router.

Accounts that initiate a withdraw request cannot hold an additional balance. That is, a withdraw request should always include the account's entire balance and should not longer be able to increase that balance.

After each action via a lending router (enterPosition, exitPosition, migratePosition, liquidate, initiateWithdraw, forceWithdraw, claimRewards, healthFactor), any transient variables on the vault are cleared and cannot be re-used in another call back to the lending router in the same transaction.

All vault share transfers must be first authorized by a whitelisted lending router.
___

### Q: Please discuss any design choices you made.
Notional Exponent is designed to be extendable to new yield strategies and opportunities as well as new lending platforms. If there are ways to bypass the restrictions put in place by our system by the target lending platform (in this case Morpho) or a yield strategy then that may be a valid finding.
___

### Q: Please provide links to previous audits (if any) and all the known issues or acceptable risks.
All previous audits of various Notional systems are linked here. Some of the code from the leveraged vaults audits is re-used in this repository, although much of that code has been re-written to accommodate the Notional Exponent design.
https://github.com/notional-finance/contracts-v3/tree/master-v3/audits

The Notional Trading Module is used in Notional Exponent to both execute trades and receive oracle prices. It was audited in this contest:
https://audits.sherlock.xyz/contests/2

With code located here:
https://github.com/notional-finance/leveraged-vaults/blob/master/contracts/trading/TradingModule.sol

And deployed to this location:
https://etherscan.io/address/0x594734c7e06C3D483466ADBCe401C6Bd269746C8
___

### Q: Please list any relevant protocol resources.
Protocol documentation is in the `audit` folder of the repo.
___

### Q: Additional audit information.
Some of the code is derived from previous Notional Leveraged Vaults:
https://github.com/notional-finance/leveraged-vaults/tree/master

However, the code as been rewritten to work with a more modular system so the diff from the leveraged vaults code is significant. It would be more accurate to say that the strategies are derived from similar ideas from the Notional Leveraged Vault system.


# Audit scope

[notional-v4 @ 0096f1f64071cafbf20062a7c092c6ec89c28275](https://github.com/notional-finance/notional-v4/tree/0096f1f64071cafbf20062a7c092c6ec89c28275)
- [notional-v4/src/AbstractYieldStrategy.sol](notional-v4/src/AbstractYieldStrategy.sol)
- [notional-v4/src/oracles/AbstractCustomOracle.sol](notional-v4/src/oracles/AbstractCustomOracle.sol)
- [notional-v4/src/oracles/AbstractLPOracle.sol](notional-v4/src/oracles/AbstractLPOracle.sol)
- [notional-v4/src/oracles/Curve2TokenOracle.sol](notional-v4/src/oracles/Curve2TokenOracle.sol)
- [notional-v4/src/oracles/PendlePTOracle.sol](notional-v4/src/oracles/PendlePTOracle.sol)
- [notional-v4/src/proxy/AddressRegistry.sol](notional-v4/src/proxy/AddressRegistry.sol)
- [notional-v4/src/proxy/Initializable.sol](notional-v4/src/proxy/Initializable.sol)
- [notional-v4/src/proxy/TimelockUpgradeableProxy.sol](notional-v4/src/proxy/TimelockUpgradeableProxy.sol)
- [notional-v4/src/rewards/AbstractRewardManager.sol](notional-v4/src/rewards/AbstractRewardManager.sol)
- [notional-v4/src/rewards/ConvexRewardManager.sol](notional-v4/src/rewards/ConvexRewardManager.sol)
- [notional-v4/src/rewards/RewardManagerMixin.sol](notional-v4/src/rewards/RewardManagerMixin.sol)
- [notional-v4/src/routers/AbstractLendingRouter.sol](notional-v4/src/routers/AbstractLendingRouter.sol)
- [notional-v4/src/routers/MorphoLendingRouter.sol](notional-v4/src/routers/MorphoLendingRouter.sol)
- [notional-v4/src/single-sided-lp/AbstractSingleSidedLP.sol](notional-v4/src/single-sided-lp/AbstractSingleSidedLP.sol)
- [notional-v4/src/single-sided-lp/CurveConvex2Token.sol](notional-v4/src/single-sided-lp/CurveConvex2Token.sol)
- [notional-v4/src/staking/AbstractStakingStrategy.sol](notional-v4/src/staking/AbstractStakingStrategy.sol)
- [notional-v4/src/staking/PendlePT.sol](notional-v4/src/staking/PendlePT.sol)
- [notional-v4/src/staking/PendlePTLib.sol](notional-v4/src/staking/PendlePTLib.sol)
- [notional-v4/src/staking/PendlePT_sUSDe.sol](notional-v4/src/staking/PendlePT_sUSDe.sol)
- [notional-v4/src/staking/StakingStrategy.sol](notional-v4/src/staking/StakingStrategy.sol)
- [notional-v4/src/utils/Constants.sol](notional-v4/src/utils/Constants.sol)
- [notional-v4/src/utils/TokenUtils.sol](notional-v4/src/utils/TokenUtils.sol)
- [notional-v4/src/utils/TypeConvert.sol](notional-v4/src/utils/TypeConvert.sol)
- [notional-v4/src/withdraws/AbstractWithdrawRequestManager.sol](notional-v4/src/withdraws/AbstractWithdrawRequestManager.sol)
- [notional-v4/src/withdraws/ClonedCooldownHolder.sol](notional-v4/src/withdraws/ClonedCooldownHolder.sol)
- [notional-v4/src/withdraws/Dinero.sol](notional-v4/src/withdraws/Dinero.sol)
- [notional-v4/src/withdraws/Ethena.sol](notional-v4/src/withdraws/Ethena.sol)
- [notional-v4/src/withdraws/EtherFi.sol](notional-v4/src/withdraws/EtherFi.sol)
- [notional-v4/src/withdraws/GenericERC20.sol](notional-v4/src/withdraws/GenericERC20.sol)
- [notional-v4/src/withdraws/GenericERC4626.sol](notional-v4/src/withdraws/GenericERC4626.sol)
- [notional-v4/src/withdraws/Origin.sol](notional-v4/src/withdraws/Origin.sol)


