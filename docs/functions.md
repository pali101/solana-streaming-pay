# Contract Functions and Logic

## Public Functions (Entry Points)

### create_stream

Starts a new payment stream between a sender and a recipient, locking funds in a special account controlled by the contract.

#### How it works

- The sender calls this function with parameters including recipient, amount, token address (SOL or another token), and schedule (start and end time).
- The contract safely holds the funds until they are earned.
- Internally, the contract checks if the user is sending SOL or another token, and sets up the right type of account to hold the money.
- It then saves all details about the stream, so future withdrawals or changes can be processed.

Note - The front-end must first transfer the funds to the derived vault account, then call `create_stream` to initialize the stream.

### redeem_stream

Allows the recipient to withdraw (claim) the money they have earned so far from an active stream.

#### How it works

- The recipient calls this function whenever they want to claim what they have earned.
- The contract checks the current time and the stream's flow rate to calculate how much is available for withdrawal.
- It transfers the appropriate amount to the recipient.
- The contract updates its records to track how much has already been claimed for proper accounting and stop double spend.

### modify_stream

Lets the original sender adjust certain details of an ongoing stream (such as flow rate), within the contract’s rules.

#### How it works

- The sender can propose changes, like increasing flow rate, reduce end time.
- The contract checks if these changes are allowed (for example, update reduce time cannot be less than current time).
- If the changes are valid, the contract updates the stream's details and emit event.

## Internal and Helper Functions

While users can only interact with the entry functions (create, redeem, modify), the contract utilizes various helper functions behind the scenes.

**PDA Derivation note**: 
We can utilize combination of `payer`, `payee` and `token` to derive address to reduce on-chain storage. If we derive it using this combination, we don't need to store these in struct. 

The Stream struct stores:
- initial_amount (u64)
- flow_rate (u64)
- start_time (i64)
- end_time (i64)
- last_redeemed_time (i64)
- total_redeemed_amount (u64).

### Helper for create_stream

- Native payment helper: Checks necessary condition related to native payment.
- Token (ERC20) payment helper: Checks necessary conditions (e.g. `allowance`) for utilizing another token.
- Save stream details: Stores all stream details into a special account for easier tracking and management. 

### Helper for redeem_stream

- Native payout: Handles the payout to the recipient if the stream is in SOL.
- Token payout: Handles payouts in other tokens, with similar checks.
- Update stream records: updates the stream’s records so the contract knows how much has been claimed.

**SECURITY NOTE**: Update records before actually processing the payout (prevents reentrancy attack).

### Helper for Modify Stream

- Check change validity: Confirms that any requested changes to a stream follow the rules and don’t result in errors (like paying out less than has already been withdrawn).
- Apply Changes: Updates the stream’s stored information to reflect approved changes.