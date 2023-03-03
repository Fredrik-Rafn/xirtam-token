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

This contract is essentially the most important contract we import, as it's what we use to control balances of each address, allowances, token name and symbol, transfer functions, etc.

You can view this as the base of the smart contract. It gives us the very basic functions in order to have a functional function.

The original OpenZeppelin documentation is kept in the smart contract if you're curious about the individual functions of the ERC20 contract.

#### Ownable

Ownable is a widely-used contract to control who can use certain functions in the smart contract. Ex. we don't want anyone to be able to change what address the fees go to, so that function has the "onlyOwner" attribute:

```c
/* line 694-696 */
function setFeesAddress(address marketing) external onlyOwner { // allows us to change the fee receiving address.
        marketingAddress = marketing;
}
```

The contract also has functions to change the ownership of the smart contract, which is essentially just a safety measure, in case there is a need to change it.

#### IPancakeV2Factory

This interface is used to create the pair between XIRTAM and ETH. That is only function that we import from the factory.

#### IPancakeV2Router02

This interface is used to swap the tokens into ETH, allowing the fee-receiving address to receive ETH instead of XIRTAM tokens as fees.

#### XIRTAM

This is the important part of the smart contract. This is where we customize the token to make it do whatever we need it to do.

XIRTAM is kept extremely simple; it's just a simple token with a fee that is received in ETH.

For detailed information about each relevant line of code, please view the source code of the smart contract by going to the contracts folder and clicking on xirtam_token.sol.

##### Transfer function

```c
/* line 731-770 */
function _transfer(
        address from,
        address to,
        uint256 amount
    ) internal override {
        uint256 trade_type = 0;
        // normal transaction ( address to address )
        if (!isInternalTransaction) {
            //buy
            if (automatedMarketMakerPairs[from]) {
                trade_type = 1;
            }
            //sell
            else if (automatedMarketMakerPairs[to]) {
                trade_type = 2;
                bool overMinimumTokenBalance = balanceOf(address(this)) >=
                    minimumTokensBeforeSwap;
                // marketing auto-bnb
                if (overMinimumTokenBalance) {
                    swapTokens();
                }
            }

            // buy ( address to pair )
            uint256 txFees;
            if (trade_type == 1 && !excludedFromFees[to]) {
                txFees = (amount * fee) / 1000;
                amount -= txFees;
                super._transfer(from, address(this), txFees);
            }
            //sell ( pair to address)
            else if (trade_type == 2 && !excludedFromFees[from]) {
                txFees = (amount * fee) / 1000;
                amount -= txFees;
                super._transfer(from, address(this), txFees);
            }
        }
        // transfer tokens
        super._transfer(from, to, amount);
}
```

Even though this function might look complex to some, it's actually extremely simple.

Here you can see three examples of how the process of the function goes on a buy, on a sell and on a regular transfer from one wallet to another.

###### If a BUY occurs:

1. Declares 3 variables, "from", "to", and "amount".
In this case, "from" is the XIRTAM/ETH pair, "to" is the address that processed the transaction (the buyer), and "amount" is the amount of tokens that is bought.

2. Sets "trade_type" to 1, which will be the variable to tell the rest of the code that this is a buy.

3. It then checks if the transaction is a buy AND if the buyer is not excluded from fees (token contract, fee receiver and owner is excluded from fees).

4. If the buyer is not excluded - which we assume they aren't in this case, because the token contract, fee receiver, and owner will never buy or sell - we then deduct the fee from the amount they're trying to buy, so we know how much to send to the fee receiver and how much to send to the buyer.

5. After that, the tokens from the fees are sent to the contract, and since the rest of the if-statements are false, it will skip straight to line 769, where the buyer's tokens are transferred.

###### If a SELL occurs:

1. Declares 3 variables, "from", "to", and "amount".
In this case, "from" is the address that processed the transaction (the seller), "to" is the XIRTAM/ETH pair, and "amount" is the amount of tokens that is sold.

2. Sets "trade_type" to 2, which will be the variable to tell the rest of the code that this is a sell.
Because this is also a sell, it will also check if the contract's token balance is above the threshold to do a swap. If it is, then it performs a swap with the tokens in the contract and sends the ETH to the fee receiver.

3. It then checks if the transaction is a sell AND if the seller is not excluded from fees (token contract, fee receiver and owner is excluded from fees).

4. If the seller is not excluded - which we assume they aren't in this case, because the token contract, fee receiver, and owner will never buy or sell - we then deduct the fee from the amount they're trying to sell, so we know how much to send to the fee receiver and how much to send to the seller.

5. After that, the tokens from the fees are sent to the contract, and since the rest of the if-statements are false, it will skip straight to line 769, where the seller's tokens are transferred.

###### If a NORMAL TRANSFER occurs:

1. Declares 3 variables, "from", "to", and "amount".
In this case, "from" is the address sending the tokens, "to" is the address receiving the tokens, and "amount" is the amount of tokens, that "from" is sending to "to".

2. Because neither "from" or "to" is the XIRTAM/ETH pair, it won't set the trade_type to anything, meaning it will skip all the code until line 769, where it sends the tokens to the "to"-address.
Because it skips the rest of the code, nothing will be deducted from "amount" and the "to"-address will receive 100% of the tokens that "from" is sending.

## Developer
All smart contracts on this repository is developed by me, @bcrypt2.
