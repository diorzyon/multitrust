# Multi-Account Pallet for Substrate

## Project Overview

This project implements a **Multi-Account Pallet** for Substrate, designed to facilitate multi-signature account operations. The pallet allows a group of signatories to collectively control an account and execute transactions based on a predefined threshold. The pallet provides a deterministic interface for managing multi-accounts, prioritizing security and efficiency. The consequence of this design is that there is no indecision or ambiguity in the decision-making process. Once a transaction is approved by the required number of signatories, it is executed automatically. A signer cannot change their mind on an approved transaction, just like a distributed key system cannot unsign a transaction. So our system inherits more cryptographic features than governance.

## Background and Considerations

Multi-signature accounts are essential for decentralized organizations, secure fund management, and shared ownership models in blockchain networks. In designing this pallet, key considerations included:

- **Security**: The design uses deterministic account ID generation based on signatories and threshold, ensuring consistent account addresses across the network.

- **Flexibility**: Signatories can propose and approve any runtime call, enabling complex multi-party operations.

- **Efficiency**: The pallet uses bounded vectors to prevent storage bloat and ensures signatories are sorted for efficient lookups.

- **Usability**: The system remains simple enough for developers to integrate into dApps and parachains.

- **Key Precomputation**: The system allows users to generate their multi-account keys off-chain using the deterministic `multi_account_id` function.

## State Transition Function Design

The core logic of the pallet is structured as follows:

1. **Multi-Account Registration**:

   - A new multi-account is created with a deterministic account ID derived from the sorted list of signatories and threshold.

   - The account ID is computed off-chain using `multi_account_id`, and validation occurs on-chain.

   - Signatories must be provided in sorted order, and the caller is automatically inserted into the sorted list.

   - A threshold must be at least 1, ensuring at least one approval is required.

2. **Transaction Proposal and Approval**:

   - A signatory can propose a transaction by creating a call.

   - The proposer automatically approves the call upon creation.

   - Other signatories can approve the call using `approve_or_dispatch_call`.

   - Once the threshold is met, the call is automatically dispatched.

   - The pallet prevents double voting by ensuring each signatory can only approve once.

3. **Call Execution**:

   - When the approval threshold is reached, the call is dispatched immediately.

   - The call is executed with the multi-account as the origin.

   - The pallet tracks executed calls to prevent re-execution.

4. **Account Management**:

   - Multi-accounts can hold balances like regular accounts.

   - The account metadata (signatories and threshold) is stored on-chain.

   - The pallet ensures only registered signatories can propose or approve calls.

## Compromises and Improvements

- **No On-Chain Proposal Expiry**: Proposals remain open indefinitely unless manually deleted, which could lead to unused entries in storage.

- **No Call Cancellation**: Currently, there is no mechanism to cancel a proposed call or revoke an approval once given.

- **Storage Optimization**: The threshold is stored separately from signatories, requiring multiple storage reads. This could be optimized using a `NStorageMap` to store all account information together.

- **Lack of UI**: Currently, there is no frontend interface for easier interaction.

- **No Time-Locks**: Adding time constraints for transaction approvals could enhance security.

- **Runtime Upgrade**: If the runtime is upgraded, pending calls might fail. A solution could be to add the runtime version at the time of creation.

- **No Account Closure**: There is currently no mechanism to close a multi-account and recover any deposits.

## Running the Project

### Prerequisites

- Rust & Cargo installed

- Substrate development environment set up

- frame omni-node

### Build and Test

1. Clone the repository:

   ```sh
   git clone <repository-url>
   cd multitrust
   ```

2. Build the pallet:

   ```sh
   cargo build --release
   ```

3. Run tests:

   ```sh
   cargo test -p pallet-multi-account
   ```

## Security Considerations

- Although accounts are generated off-chain using deterministic derivation, the pallet ensures that the multi-account address is secure and does not impersonate an existing account by computing the same address deterministically.

- Transactions are automatically executed upon approval when the threshold is met, ensuring explicit intent before dispatch.

- The pallet ensures only registered signatories can vote, approve, or propose transactions.

- The pallet implements idempotent operations to prevent duplicate transactions through call hash tracking.

- Signatories must be sorted, preventing duplicate entries and ensuring consistent account ID generation.

- Double voting is prevented by checking if a signatory has already approved a call before adding them to the approval list.

## Decision Making Process

- **State Mutation**: The pallet uses `try_mutate` on the `Calls` storage to atomically update approvals and check thresholds, ensuring consistency.

- **Type Casting**: The pallet casts the length of bounded vectors as `u16` for threshold comparison because bounded vectors are limited, so overflow is unlikely.

- **Overflow Prevention**: The pallet ensures that the length of bounded vectors is within the bounds of `u32` to prevent overflow.

- **Deterministic Account ID**: The account ID is derived using Blake2-256 hashing of a constant prefix, sorted signatories, and threshold, ensuring deterministic generation across all nodes.

- **Automatic Approval**: When a signatory creates a call, they are automatically added to the approval list, reducing the number of transactions needed.

## RPC Interface

The pallet provides RPC endpoints for querying multi-account information:

- `multi_NumberOfAccountsHasApprovedCall`: Get the number of signatories that have approved a particular call hash.

- `multi_AccountSigners`: Get the list of signatories for a multi-account.

- `multi_AccountThreshold`: Get the threshold required for a multi-account.

- `multi_SignersWhoApprovedCall`: Get the list of signatories that have approved a particular call.
