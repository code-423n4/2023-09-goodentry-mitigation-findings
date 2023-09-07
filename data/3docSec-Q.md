# Lines of code

https://github.com/GoodEntry-io/ge/blob/master/contracts/TokenisableRange.sol#L251
https://github.com/GoodEntry-io/ge/blob/master/contracts/TokenisableRange.sol#L252


# Vulnerability details

(courtesy submission as gas / low-risk findings are out of scope for the contest)

The rework of the fees handling and TokenisableRange's `deposit` function made these two calls useless:
```Solidity
      uint val0 = u0 * ORACLE.getAssetPrice(address(TOKEN0.token)) / 10**TOKEN0.decimals;
      uint val1 = u1 * ORACLE.getAssetPrice(address(TOKEN1.token)) / 10**TOKEN1.decimals;
```
so they can be removed to save a considerable amount of gas.


## Assessed type

call/delegatecall