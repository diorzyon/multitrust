# Multitrust

A blockchain that ensures multi-signature governance, secure collective decision-making, and on-chain execution.

## Project Overview

**Multitrust** is a Substrate-based blockchain designed to facilitate secure, multi-party operations through advanced multi-signature account management and social networking capabilities. The project combines deterministic multi-account generation with sybil-resistant identity management to create a robust foundation for decentralized governance and collective decision-making.

### Key Features

- **Multi-Account Pallet**: Deterministic multi-signature account management with threshold-based transaction execution
- **Connect Pallet**: Sybil-resistant social network infrastructure with user metadata storage
- **Smart Contracts**: Full support for WebAssembly smart contracts via `pallet-contracts`
- **RPC Integration**: Custom RPC endpoints for querying multi-account state and network statistics
- **Runtime APIs**: Extensible runtime APIs for off-chain integration

## Architecture

This project is built using [Substrate](https://substrate.io/), a modular blockchain framework. The runtime is composed of several FRAME pallets:

### Core Pallets

- **System**: Core blockchain functionality
- **Balances**: Account balance management
- **Timestamp**: Block timestamp tracking
- **Aura**: Block authoring consensus
- **Grandpa**: Finality gadget for block finalization
- **Transaction Payment**: Transaction fee handling
- **Sudo**: Administrative control (development only)

### Custom Pallets

#### Multi-Account Pallet (`pallets/multi-account`)

A sophisticated multi-signature account management system that enables:

- Deterministic account ID generation from signatories and threshold
- Threshold-based transaction approval and automatic execution
- Call proposal and approval tracking
- RPC endpoints for querying account state

See [`pallets/multi-account/README.md`](./pallets/multi-account/README.md) for detailed documentation.

#### Connect Pallet (`pallets/connect`)

A sybil-resistant social network foundation featuring:

- User registration with locked balance requirements
- Unique name and bio metadata storage
- Random profile picture generation
- Network statistics tracking

See [`pallets/connect/README.md`](./pallets/connect/README.md) for detailed documentation.

#### Smart Contracts (`pallet-contracts`)

Full WebAssembly smart contract support with:

- Contract deployment and execution
- Deposit-based storage management
- Chain extension support

## Getting Started

### Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) (latest stable version)
- [Substrate Prerequisites](https://docs.substrate.io/install/) for your operating system

### Build

Build the node in release mode:

```sh
cargo build --release
```

This will produce a binary at `./target/release/node-template`.

### Run a Development Node

Start a single-node development chain:

```sh
./target/release/node-template --dev
```

The `--dev` flag:
- Uses a temporary database (state is cleared on restart)
- Uses the "Alice" account as the default validator
- Enables detailed logging

To persist state between runs, specify a base path:

```sh
./target/release/node-template --dev --base-path ./my-chain-state
```

### Connect with Polkadot-JS Apps

Once your node is running, you can interact with it using [Polkadot-JS Apps](https://polkadot.js.org/apps/):

1. Open [Polkadot-JS Apps](https://polkadot.js.org/apps/)
2. Click the network selector (top left)
3. Select "Local Node" or enter `ws://127.0.0.1:9944`
4. The interface will connect to your local node

### Run Tests

Run the test suite for all pallets:

```sh
cargo test
```

Run tests for a specific pallet:

```sh
cargo test -p pallet-multi-account
cargo test -p pallet-connect
```

### Generate Documentation

Generate and view the Rust documentation:

```sh
cargo +nightly doc --open
```

## Project Structure

```
multitrust/
├── node/                 # Node implementation
│   ├── src/
│   │   ├── chain_spec.rs # Chain specification (genesis state)
│   │   ├── service.rs    # Node service implementation
│   │   └── ...
│   └── Cargo.toml
├── runtime/              # Runtime implementation
│   ├── src/
│   │   └── lib.rs        # Runtime composition
│   └── Cargo.toml
├── pallets/              # Custom FRAME pallets
│   ├── multi-account/   # Multi-signature account pallet
│   │   ├── src/
│   │   ├── rpc/         # RPC server implementation
│   │   └── README.md
│   └── connect/         # Social network pallet
│       ├── src/
│       ├── rpc/         # RPC server implementation
│       └── README.md
├── scripts/              # Utility scripts
└── Cargo.toml           # Workspace configuration
```

## Runtime Configuration

The runtime is configured with the following parameters:

- **Block Time**: 6 seconds (`MILLISECS_PER_BLOCK = 6000`)
- **SS58 Prefix**: 42 (Substrate generic)
- **Existential Deposit**: 500 units
- **Max Signatories**: 25 per multi-account
- **Max Bio Length**: 200 characters
- **Max Name Length**: 10 characters

## RPC Endpoints

### Multi-Account RPC

- `multi_NumberOfAccountsHasApprovedCall(id, call_hash)`: Get approval count for a call
- `multi_AccountSigners(id)`: Get signatories for a multi-account
- `multi_AccountThreshold(id)`: Get threshold for a multi-account
- `multi_SignersWhoApprovedCall(id, call_hash)`: Get list of approvers for a call

### Connect RPC

- `connect_total_registered()`: Get total number of registered users

## Development

### Adding a New Pallet

1. Create a new directory under `pallets/`
2. Implement the pallet following FRAME conventions
3. Add the pallet to `runtime/src/lib.rs`:
   - Implement the `Config` trait
   - Add to `construct_runtime!` macro
4. Update the node's RPC configuration if needed

### Benchmarking

Generate weight files for your pallets:

```sh
./target/release/node-template benchmark pallet \
  --chain dev \
  --pallet pallet_multi_account \
  --extrinsic '*' \
  --steps 20 \
  --repeat 10 \
  --output pallets/multi-account/src/weights.rs
```

## Security Considerations

- **Development Mode**: The `--dev` flag uses the "Alice" account as sudo. **Never use this in production**.
- **Multi-Account Security**: Multi-accounts use deterministic ID generation. Ensure signatories are properly validated.
- **Connect Pallet**: Requires locked balance to prevent sybil attacks. Adjust `MinimumLockableAmount` based on token economics.
- **Smart Contracts**: Contract calls are filtered. Review `CallFilter` in runtime configuration.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## License

This project is licensed under the MIT-0 License. See the [LICENSE](./LICENSE) file for details.

## Resources

- [Substrate Documentation](https://docs.substrate.io/)
- [FRAME Documentation](https://docs.substrate.io/reference/frame-pallets/)
- [Substrate Recipes](https://substrate.dev/recipes/)
- [Polkadot Wiki](https://wiki.polkadot.network/)

## Acknowledgments

This project is based on the [Substrate Node Template](https://github.com/substrate-developer-hub/substrate-node-template) and extends it with custom pallets for multi-signature governance and social networking capabilities.
