# Propose-Dispute POA Bridge

## Reasoning & Architecture
In the current POA Brdige, validators must reach a super-majority (N/2+1) to process transactions. This means that the transfer cost will grow linearly with the amount of validators. This bottleneck puts a limit on the amount of validators that can be included in the bridge. 

As an alternative we propose a propose & dispute style bridge. In this model transfers will be proposed on the recipient chain. This transfer proposal is essentially a claim that a particular transaction has occurred on the sending chain. Proposals will have a bond attached that will serve as an incentive for honest actors to initiate a dispute of the transaction.

The protocol is safe if we assume an honest actor can submit a dispute within the given dispute window. As the system's safety relies on selecting proper parameters, we will spend some time here discussing and defining the different parameters and their impact on the system.

### System Parameters
* ``validatorCount`` - The total number of validators in the system.
* ``superMajorityCount`` - The threshold that qualifies as a super-majority, usually 2N+1, where N is the validatorCount.
* ``collateralRequirement`` - Can either be a fixed number or dynamic number (based on the transfer value). Can either be a monetary amount or number of validators vouching for a particular transaction.
* ``disputeWindow`` - Can either be a fixed number or dynamic number (based on the transfer value). Dispute window should be higher on the foreign chain, as it would require a byzantine local-chain to censor transactions (assuming POA based local-chain with validator overlap).

#### validatorCount
Validator count will determine the required super-majority to settle disputes. Ideally, this validator set will match the POA chains pre-existing POA set. This will mean that they can include their own dispute transactions and have greater accountability to the ecosystem.

#### superMajorityCount
For simplicity, we will assume this to always be `validatorCount / 2 + 1`.

#### collateralRequirement
We will use a dynamic collateral requirement based on the amount of value being withdrawn to the foreign chain. This is because a validator cannot guarantee their transaction is included in the foreign chain in the same way a validator can guarantee their transaction is included locally.

Now that we have determined our collateral will be a function of the transfer's value, we must determine how to source the collateral. We have two options: either we let anyone put up the required collateral, or we require that a dynamic threshold of validators sign off on the transfer, which would by extension stake their collateral on that transfer. For this example we will use the second option and require a dynamic number of validators to sign-off on a proposal before it enters its dispute window.

#### disputeWindow
Dispute window will also be dynamic and set based on the size of the transfer in relation to the total assets held by the bridge. I.e. a transfer withdrawing 10% of the locked assets will take significantly longer than a transfer withdrawing <1%. To avoid sybil attacks on the withdrawals, the amount withdrawn for a given time period will determine the withdrawal times for the tranfers. Such that 10x 1% withdrawals will take as long as 1x 10% withdrawal.

## Logical Structure for ETH Contract
```solidity
contract Bridge {

  uint private validatorCount;
  uint private pendingWithdrawals;
  uint private tokensLocked;
  uint private validatorEarnings;   // Pending payment to validators in tokens
  uint private baseWithdrawalTime;  // Minimum dispute period for a withdrawal
  uint private timePerBasis;        // Seconds to wait per basis point withdrawn (0.01%)
  uint private proposalCount;
  
  struct WithdrawalProposal {
    address   beneficiary;
    uint      withdrawalAmount;
    uint      requiredSignatures;
    boolean   proposed;
    boolean   confirmed;
    uint      disputeEnds;
    address[] signatories;
  }
  
  function getWithdrawalTime(uint _amountToWithdraw) public returns(address[]) {
    // Returns withdrawal time based on the size of withdrawal and value in pending withdrawals
  }
  
  function proposeWithdrawal(uint _value, address _recipient, uint depositId) public returns(proposalId uint) {
    // Create new withdrawal proposal
    // depositId is the id of the deposit on the POA chain
  }
  
  function commitDeposit() public returns(proposalId uint) {
    // Commit deposit here first before starting withdrawal on foreign chain
  }
  
  function createWithdrawProposal(uint _amount, address _to) private returns(proposalId uint) {
    // Setup a new proposal and set the required signatures to reach pending status
  }
  
  function signOnWithdrawalProposal(uint _proposalId) public onlyValidator returns() {
    // Attach a validators signature to a proposal
    // If the proposal has sufficient signatures, set it to pendign and begin dispute period
  }
  
  function disputePendingWithdrawal(uint proposalId) {
    // Halt the withdrawal for proposalId
    // Open voting to resolve dispute for proposalId
  }
  
  function castVoteOnDispute(uint proposalId, boolean approve) {
    // Attach an affirmative or negative vote to a particular proposalId
  }
  
  function processWithdrawal() {
    // Require that now > proposalExpiry && proposal is not disputed
    // Process the withdrawal & reward the processee
  }
}
```

## Functionality for ETH Contract
```solidity
contract Bridge {

  uint private validatorCount;
  uint private pendingWithdrawals;
  uint private tokensLocked;
  uint private validatorEarnings;   // Pending payment to validators in tokens
  uint private baseWithdrawalTime;  // Minimum dispute period for a withdrawal
  uint private timePerBasis;        // Seconds to wait per basis point withdrawn (0.01%)
  uint private proposalCount;
  
  struct WithdrawalProposal {
    address   beneficiary;
    uint      withdrawalAmount;
    uint      requiredSignatures;
    boolean   proposed;
    boolean   confirmed;
    uint      disputeEnds;
    address[] signatories;
  }
  
  mapping(uint => WithdrawalProposal) private idToProposal;
  

  function getWithdrawalTime(uint _amountToWithdraw) public returns(address[]) {
    uint _pendingWithdrawals = pendingWithdrawals + _amountToWithdraw;
    uint withdrawalTime = baseWithdrawalTime + (10000 * _pendingWithdrawals / tokensLocked) * timePerBasis;
  }
  
  function proposeWithdrawal(uint _value, address _recipient) public returns(proposalId uint) {
    createWithdrawProposal(_value, _recipient);
  }
  
  function createWithdrawProposal(uint _amount, address _to) private returns(proposalId uint) {
    uint requiredSignatures = (10000 * _amount / tokensLocked) * validatorCount / 10000;
    
    proposalCount++;
    idToProposal[proposalCount] = WithdrawalProposal(_to, _amount, requiredSignatures);
  }
  
  function signOnWithdrawalProposal(uint _proposalId) public onlyValidator returns() {
    WithdrawalProposal storage withdrawal = idToProposal[_proposalId];
    
    require(validatorHasNotVoted(withdrawal));
    
    withdrawal.signatories.push(msg.sender);
    
    if (countSignatures(withdrawal) == withdrawal.requiredSignatures) {
      withdrawal.proposed = true;
      withdrawal.disputeEnds = getWithdrawalTime(withdrawal.withdrawalAmount);
      pendingWithdrawals += withdrawal.withdrawalAmount;
    }
  }

}

```

## Interface
```solidity
function getValidators() public returns(address[]) {}
```
