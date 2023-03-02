# XIRTAM Smart Contract Repository

The official smart contract repository for XIRTAM.

This is where you'll find every smart contract used for XIRTAM, including detailed documentation for each smart contract.

Each section will include a detailed explanation of every relevant line/section of code.

The smart contracts are well commented on in themselves, so feel free to check out the code itself for more in-depth documentation.

## XIRTAM Token

#### IERC20
This interface is included because it allows us to interact with other contracts.
In our case, we need to ask for approval before swapping the tokens accumulated from fees to ETH.

You can see the interface being used in our swapTokensForEth function:
```c
/* line 709-721 */
function swapTokensForEth(uint256 tokenAmount) private { // functionality to swap the tokens accumulated from fees into ETH.
        address[] memory path = new address[](2);
        path[0] = address(this);
        path[1] = pancakeV2Router.WETH();
        _approve(address(this), address(pancakeV2Router), tokenAmount);
        pancakeV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(
            tokenAmount,
            0,
            path,
            marketingAddress,
            block.timestamp
        );
}
```

#### IERC20Metadata

This interface is simply used to get the name, symbol and decimals for our token.

#### Context

This abstract is used to retrieve the sender of each transaction. We can do this with the function:

```c
msg.sender(); // returns address of caller.
```

You can also retrieve the data of each transaction with:

```c
msg.data();
```

However, this is not used in this smart contract.

#### ERC20

This contract is essentially the most important contract we import, as it's what we use to 
