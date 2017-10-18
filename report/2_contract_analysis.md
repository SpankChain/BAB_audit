# Contract Analysis

## General Findings

- Reentrancy bug to be exploited by the Spankchain team.
- Many typos and capitalization issues.
- Documentation might be better suited as a markdown file .
- Enums are typically first-letter-capitalized by convention.
- Run code through style-guide. Don’t make comments/code longer than 80 characters per line.
- Modifiers are generally named with lowerCamelCase.
- Be consistent with variable naming conventions. 
- Prefer `x > 0` to `0 < x` for code readability.
- Ensure that variable placement about logical operators is consistent, ie. a < b; a != c.
- Better practice to set the auction state to deployed in the constructor rather than next to variable declarations.
- Question: Why is the token instantiated in `startAuction` instead of when the token address is supplied to the contract?

## Modifiers

##### `auction_deployed_waiting_to_start`
  - Requires the auction to be in the "deployed" stage
##### `in_purchase_phase`
  - Requires the current block number to be less than the block demarcating the beginning of the "processing" phase 
  - Requires the auction to be in the "started" stage
##### `in_processing_phase`
  - Requires the current auction to be in the "started" phase
  - Requires the current block number to be greater than or equal to the block demarcating the beginning of the "processing" phase
  - Requires the current block number to be less than the block number demarcating the end of the auction
##### `auction_complete`
  - Requires the current block number to be greater than or equal to the block demarcating the end of the auction
  - Or requires the auction to be in the "success" phase
  - Or requires the auction to be in the "cancel" phase
##### `strike_price_set`
  - Requires the strike price to be greater than zero
##### `owner_only`
  - Requires the entity that sent the message to be the entity that instantiated the auction contract

## Functions

### Owner Functions 
##### `Auction`
###### Findings:
  - Should check for valid address inputs: `_tokenAddress`, `_weiWallet`, and `_tokenWallet`.
###### Function Description:
  - Instantiates the smart contract with the provided parameters

##### `startAuction`
###### Findings:
  - None
###### Function Description:
  - Defines the token contract relelvant to the auction
  - Requires that the token balance assigned to the contact is less than the maximum number of tokens for sale
  - Defines the block number demarcating the start of the processing phase
  - Defines the block number demarcating the end of the auction
  - Sets the current auction stage to "started"

##### `setStrikePrice`
###### Findings:
  - Use a modifier to check the condition `the strike price can only be set once`. 
###### Function Descriptions:
  - Requires the decided "strike price" to be greater than zero
  - Sets the "strike price" decided by the owner of the auction

##### `processBid`
###### Findings: 
  - Cannot  fully evaluate/verify this function without knowing off-chain encryption scheme.
###### Function Descriptions:
  - Requires that the bid amount is greater than the minimum amount 
  - Requires the strike price to be less than or equal to the bid
  - Requires the bid per token to be less than or equal to the total amount of money bid
  - Verifies the signature on the bid
  - Calculates the number of tokens purchased by the bid and updates the state of the buyer and the contract

##### `completeSuccessfulAuction`
###### Findings:
  - This function is vulnerable to reentrancy attacks.
  - Not changing AuctionState to success before transferring the `totalWeiRaised` allows a malicious team member to drain the deposits of the bidders who did not meet the strike price and the remaining deposits of bidders who successfully purchased tokens.
  - Swap lines 333 and 334 to fix.
###### Function Descriptions:
  - Requires that the number of tokens sold is less than the number of tokens allotted to the auction contract
  - Requires that the minimum number of tokens sold has reached the defined amount
  - Requires the amount of wei raised has reached the defined amount
  - Tranfers all unsold tokens to the wallet corresponding to the auction
  - Tranfers ether to the wallet corresponding to the auction
  - Sets the auction stage to be "success"

##### `cancelAuction`
###### Findings: 
  - Switch lines 363 and 364 as this paradigm generally leads to reentrancy. While this function is not vulnerable to a reentrancy bug, it is poor practice to write code that could be exposed to it in the situation that proper `require` statements did not precede it.
###### Function Description:
  - Requires that the auction state to not be "success"
  - Tranfers the ERC tokens back to the wallet corresponding to the auctions
  - Sets the auction stage to be "cancel'

### Bidder Functions
##### `deposit`
###### Findings:
  - None
###### Function Description:
  - Requires that the deposit is greater than the minimum required amount
  - Updates the state of the buyer's deposit

##### `withdraw`
###### Findings:
  - None
###### Function Description:
  - Requires that the buyer has not withdrawn tokens or wei
  - If the auction is successful, then withdraw tokens
  - Withdraw extra wei (extra wei is defined by the difference between the amount deposited and the total amount bid)
  
