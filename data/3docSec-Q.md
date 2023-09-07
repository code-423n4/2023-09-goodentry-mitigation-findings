# Lines of code

https://github.com/GoodEntry-io/ge/blob/c7c7de57902e11e66c8186d93c5bb511b53a45b8/contracts/GeVault.sol#L230
https://github.com/GoodEntry-io/ge/blob/master/contracts/TokenisableRange.sol#L251
https://github.com/GoodEntry-io/ge/blob/master/contracts/TokenisableRange.sol#L252


# Vulnerability details

# Users withdrawing from GeVault lose their portion of fees

## Lines of code

https://github.com/GoodEntry-io/ge/blob/c7c7de57902e11e66c8186d93c5bb511b53a45b8/contracts/GeVault.sol#L230

## Details

Referring to the other finding I submitted:
> New from fees rework: fees can still be stolen with a flash-loan on GeVault

the opposite is also true: for the same issue affecting GeVault's `withdraw` function, when users withdraw their funds, GeVault does not award them with the fees accrued by their stake before withdrawal.

This is submitted as low-risk finding because funds are not lost, and being value net-negative, it's not a valid attack path.

The suggested fix is similar to the quoted finding:
```diff
  function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {
    require(poolMatchesOracle(), "GEV: Oracle Error");
    if (liquidity == 0) liquidity = balanceOf(msg.sender);
    require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance");
    require(liquidity > 0, "GEV: Withdraw Zero");
    
+    removeFromAllTicks(); 
    uint vaultValueX8 = getTVL();
    uint valueX8 = vaultValueX8 * liquidity / totalSupply();
    amount = valueX8 * 10**ERC20(token).decimals() / oracle.getAssetPrice(token);
    uint fee = amount * getAdjustedBaseFee(token == address(token1)) / 1e4;
    
    _burn(msg.sender, liquidity);
-    removeFromAllTicks();
    ERC20(token).safeTransfer(treasury, fee);
    uint bal = amount - fee;

    if (token == address(WETH)){
      WETH.withdraw(bal);
      (bool success, ) = payable(msg.sender).call{value: bal}("");
      require(success, "GEV: Error sending ETH");
    }
    else {
      ERC20(token).safeTransfer(msg.sender, bal);
    }
    
    // if pool enabled, deploy assets in ticks, otherwise just let assets sit here until totally withdrawn
    if (isEnabled) deployAssets();
    emit Withdraw(msg.sender, token, amount, liquidity);
  }
```

-----

# Unused code

## Lines of code

https://github.com/GoodEntry-io/ge/blob/master/contracts/TokenisableRange.sol#L251
https://github.com/GoodEntry-io/ge/blob/master/contracts/TokenisableRange.sol#L252

## Details

The rework of the fees handling and TokenisableRange's `deposit` function made these two calls useless:
```Solidity
      uint val0 = u0 * ORACLE.getAssetPrice(address(TOKEN0.token)) / 10**TOKEN0.decimals;
      uint val1 = u1 * ORACLE.getAssetPrice(address(TOKEN1.token)) / 10**TOKEN1.decimals;
```
so they can be removed to save a considerable amount of gas.




## Assessed type

call/delegatecall