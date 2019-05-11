# Commit & Reveal Bridge

### Current POA Bridge
The current POA bridge has the following characteristics:
 - All transactions confirmed by super-majority.
 - No liveness requirements.
 - Transaction cost scales linearly with number of validators.
 
 I propose a design that is also based on voting and achieves the following characteristics:
 - Low gas cost in honest cases.
 - Fall-back to super-majority voting when there is a dispute.
 - Relies on game theory and fishermen (off-chain actors calling various functions).
 
 ### Architecture Overview

##### Creating a Proposal
One validator will propose a new transaction. They will do this by prposing a new mint on the receiving chain. This proposed mint will reference the deposit on the sending chain. Referencing the sending chain will prevent validators doubling up and let double up proposals be aggregated/ignored. (__Note:__ It is possible to require a threshold of validators to vote for a proposal before it goes into a pending state).

##### Proposal Pending State
Once a proposal has been successfully created, it will remain in a pending state until the challenge period ends. The challenge period is designed to allow honest validators time to challenge malicious proposals. When a malicious proposal is detected, any validator can initiate a dispute period.

##### Disputed Proposals
Proposals that are disputed will require a super-majority vote to reject/confirm them. If a disputed proposal is declined, the initial proposer will be punished. If the proposal is accepted, the disputer will be punished. There is an economic incentive for validators that dispute invalid transaction successfully.

Validators that propose invalid transactions can be removed from the validator pool after having their stake confiscated. Confiscated stakes can be combined with regular fees to incentivize honest actors. It is important that there is always sufficient incentive to dispute invalid transactions. (i.e. reward is always significantly larger than gas cost).
