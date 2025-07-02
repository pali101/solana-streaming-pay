# Solana Streaming Payments

A protocol for continuous, programmable payments on Solana.

## Core Idea

- **Streaming Payments**: Create a channel to pay a recipient (payee) at a fixed rate (per minute or per hour) over a specified time duration.

- **On-chain logic**:
    - Users can send and receive payments continuously over time, rather than in a single transaction.
    - Smart contract handles creation, redemption, and settlement of streams.
    - Supports both native SOL and SPL (ERC20-equivalent) tokens.
    - Programmable protocol fee 

- **Off-chain indexing**:
    - Channels and user associations are indexed and tracked off-chain via subgraph.
    - We don't store all data on-chain, but rather use a subgraph to index and query streams to reduce on-chain storage costs.

## How it works

1. **Channel creation**:
    - A user (payer) creates a channel to pay another user (payee) at a fixed rate.
    - Defines: 
        - Payee address
        - Payment amount (e.g., 30 SOL)
        - Payment rate (e.g., 1 SOL per minute)
        - Duration of the stream (e.g., 30 days)
        - Token type (SOL or SPL token)
    - Funds are escrowed in a smart contract.

2. **Redemption**:
    - The payee can redeem the stream at any time.
    - The smart contract calculates the amount based on the elapsed time and rate, i.e. `(current time - channel creation time) × rate`
    - Remaining funds stay in channel until fully redeemed (by payee) or reclaimed (by payer).

3. **Flexibility**:
    - Channels can be created with no fee or a programmable fee model.
    - Payee can redeem partial amounts, leaving the channel open for future payments.
    - Payer can reclaim unspent funds after the stream ends or if the payee does not redeem within a certain period.
    - Supports multiple streams per user.

## Example use case

- A company can pay employees a fixed salary every month while allowing them to redeem their salary at any time for the elapsed period.
- A subscription service (like Netflix) can charge users a fixed amount every month, allowing them to redeem their subscription at any time.