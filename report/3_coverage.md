# Code Coverage

Code coverage using the [Solidity-Coverage](https://github.com/sc-forks/solidity-coverage) tool was used to measure the what portions of the codebase was run with the given test suite. 

The test coverage results are located in the code coverage directory and can be viewed by opening the `index.html` file or by observing the image below:

Coverage Results
![alt-text](https://github.com/raychu86/SpankchainAudit/blob/master/coverage/coverage.png)

[Report Directory](../coverage)

## Oyente Report 

### Auction.sol

```contract Auction
EVM code coverage: 93.3%
Callstack bug: False
Money concurrency bug: True
Time dependency bug: False
Reentrancy bug: False
Assertion failure: True

Money concurrency bug:
Flow 1:
Auction:237:13:
msg.sender.transfer(buyer.depositInWei)
^
Flow 2:
Auction:233:13:
msg.sender.transfer(SafeMath.sub(buyer.depositInWei, buyer.bidWeiAmount))
^
Auction:214:24: Assertion failure:
SafeMath.add(buyer.depositInWei, msg.value)
^
currentAuctionState = 1461501637330902918203684832716283019655932542976
minDepositInWei = 82306579331706070424569484146079930342289088062704051215418675662991080259071
processingPhaseStartBlock = 1
```

It is highly unlikely this assertion failure will be reached since it would require an obscene amount of wei in the form of a deposit. 
