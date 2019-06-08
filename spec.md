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


## Interface
```solidity
function getValidators() public returns(address[]) {}
```
