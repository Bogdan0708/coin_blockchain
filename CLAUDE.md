# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Cosmos SDK blockchain development workspace** containing multiple blockchain projects, with the primary focus on **Mitch Chain** - a Layer-1 blockchain with native coin `MTC` (micro-denomination `uMTC`). The workspace also includes Go language source code and various blockchain-related projects.

## Primary Project: Mitch Chain

**Location**: `mitch-chain/`

Mitch Chain is a production-ready Cosmos SDK blockchain scaffolded with Ignite CLI, featuring:
- Native coin: `MTC` (base denomination: `uMTC`, 1 MTC = 1,000,000 uMTC)
- Account prefix: `mitch`
- Chain ID: `mitch-1`
- Custom modules in `x/app` for domain-specific logic (loyalty rewards, AI credits)
- IBC-ready with transfer and interchain accounts support
- CosmWasm smart contract support (optional)

### Architecture

```
mitch-chain/
├── app/                    # Core application wiring with dependency injection
├── cmd/appd/               # Binary entrypoint (command: appd)
├── proto/                  # Protobuf definitions (buf-managed)
├── x/app/                  # Custom module scaffold (keeper, types, messages)
├── deploy/aws/             # Production deployment (Terraform, systemd, bootstrap)
├── docs/                   # Operational runbooks
├── monitor/                # Prometheus + Grafana monitoring stack
├── networks/local/         # Docker Compose localnet
└── wasm-demo/              # CosmWasm contract examples
```

### Building and Running

**Prerequisites**:
- Go 1.24+ (required for SDK v0.52+ with new toolchain features)
- Ignite CLI (for proto generation and scaffolding)
- Docker & Docker Compose v2
- Node.js 18+ (for tooling)

**Build Commands**:
```bash
cd mitch-chain

# Build the node binary
make build                 # Outputs to build/appd

# Install system-wide
make install

# Run tests
make test                  # Includes govet, govulncheck, unit tests
make test-unit            # Unit tests only
make test-race            # Race condition detection
make test-cover           # Coverage report (coverage.html)

# Benchmarks
make bench
```

**Development Workflow**:
```bash
# Initialize a local node
./build/appd init local --chain-id mitch-1

# Add a test account
./build/appd keys add godja --keyring-backend test

# Add genesis account
./build/appd add-genesis-account $(./build/appd keys show godja -a --keyring-backend test) 100000000000uMTC

# Create genesis transaction
./build/appd gentx godja 25000000uMTC --chain-id mitch-1 --keyring-backend test

# Collect genesis transactions
./build/appd collect-gentxs

# Start the node
./build/appd start
```

**Using Ignite CLI**:
```bash
# Run with Ignite (auto-restarts on file changes)
make ignite-chain          # Same as: ignite chain serve --skip-proto

# Generate protobuf Go bindings after modifying .proto files
make proto-gen             # Same as: ignite generate proto-go --yes
go mod tidy
```

### Docker Local Network

```bash
# Start multi-node local network
make localnet-up           # docker compose -f networks/local/docker-compose.yml up -d --build

# Stop and clean up
make localnet-down
```

**Ports**:
- RPC: `26657`
- REST API: `1317`
- gRPC: `9090`
- P2P: `26656`
- Prometheus: `26660` (CometBFT metrics)

### Configuration

**Primary config**: `config.yml` (Ignite format)
- Defines accounts, validators, denom, faucet settings
- Used by Ignite CLI for genesis generation

**Key settings**:
- Default denom: `uMTC`
- Genesis accounts: `godja` (100,000,000,000 uMTC), `validator` (50,000,000,000 uMTC)
- Validator bonded amount: 25,000,000 uMTC
- Faucet: distributes 50,000,000 uMTC

### Custom Module Development

The `x/app` module is the default Ignite scaffold for custom logic.

**Adding new messages/queries**:
1. Define in `proto/app/app/v1/` (tx.proto, query.proto)
2. Run `make proto-gen` to generate Go bindings
3. Implement keeper methods in `x/app/keeper/`
4. Wire message/query servers in keeper
5. Add tests in `*_test.go` files

**Creating new modules**:
```bash
ignite scaffold module <module-name>
ignite scaffold message <message-name> [fields...] --module <module-name>
ignite scaffold query <query-name> [fields...] --module <module-name>
```

### Production Deployment

**AWS Infrastructure** (`deploy/aws/terraform/`):
- VPC with public (sentry) and private (validator) subnets
- Security groups isolating validators from public internet
- EC2 instances with encrypted EBS volumes
- IAM roles for SSM access
- Systemd service units for node management

**Bootstrap scripts** (`deploy/aws/scripts/`):
- `bootstrap_sentry.sh`: Public RPC node setup
- `bootstrap_validator.sh`: Private validator node setup
- `snapshot_sync.sh`: State sync from S3 snapshots

**Deployment workflow**:
1. Build binary: `make build`
2. Deploy infrastructure: `cd deploy/aws/terraform && terraform apply`
3. SSH to nodes and configure peers/keys (see `docs/validators.md`)
4. Create validator: `appd tx staking create-validator ...`

### Monitoring

**Stack**: Prometheus + Grafana (`monitor/docker-compose.yml`)

```bash
# Start monitoring stack
docker compose -f monitor/docker-compose.yml up -d

# Access Grafana at http://localhost:3000
```

**Metrics endpoints**:
- CometBFT: `http://node:26660/metrics`
- App metrics: `http://node:26661/metrics`

**Key metrics**:
- Block height: `tendermint_consensus_height`
- Peers: `tendermint_p2p_peers`
- Missed blocks: validator signing info
- Txs per block: `tendermint_block_txs`

### Operations

**See operational guides**:
- `docs/runbook.md`: Standard operating procedures, backups, upgrades
- `docs/validators.md`: Validator creation, key management, security
- `docs/governance.md`: Proposal lifecycle, voting, parameter changes

**Common operations**:
```bash
# Query status
appd status

# Check balances
appd q bank balances <address>

# Send tokens
appd tx bank send <from> <to> <amount>uMTC --fees 2500uMTC --chain-id mitch-1

# Query validators
appd q staking validators

# Create validator
appd tx staking create-validator \
  --amount 1000000uMTC \
  --pubkey "$(appd tendermint show-validator)" \
  --moniker "my-validator" \
  --chain-id mitch-1 \
  --commission-rate 0.10 \
  --from <key-name> \
  --fees 2500uMTC
```

### Linting and Code Quality

```bash
make lint              # Run golangci-lint
make lint-fix          # Auto-fix issues
make govet             # Go vet
make govulncheck       # Security vulnerability scan
```

## Additional Projects

### Go Language Source

**Locations**: `go1.23.3/`, `go1.24.0.tar.gz`, `go1.23.3.tar.gz`

These directories contain Go programming language source code distributions, likely for building custom Go toolchains or studying the Go compiler/runtime.

### Ignite Scaffold

**Location**: `mitch-ignite/`

Another blockchain project scaffold using Ignite CLI. Structure similar to `mitch-chain/`.

## Development Best Practices

### When Modifying Protobuf Files

1. Edit `.proto` files in `proto/` directory
2. Run `make proto-gen` to regenerate Go bindings
3. Run `go mod tidy` to update dependencies
4. Rebuild: `make build`
5. Run tests: `make test`

### When Adding Dependencies

1. Add import to Go source
2. Run `go mod tidy`
3. Verify: `go mod verify`
4. Rebuild and test

### When Creating New Modules

Use Ignite scaffolding for consistency:
```bash
ignite scaffold module <name>        # New module
ignite scaffold type <name> [fields] # New state type
ignite scaffold message <name> [fields] # New tx message
ignite scaffold query <name> [fields]   # New query
```

### Testing Strategy

- Unit tests for keeper logic: `x/app/keeper/*_test.go`
- Integration tests in `app/sim_test.go`
- Simulation tests: `make test` runs them
- Local network testing: use `make localnet-up` before major changes

## Security Considerations

- **Validator Keys**: Never store on server disk; use HSM/Ledger
- **Private Keys**: Use file or OS keyring backend in production
- **Sentry Architecture**: Validators behind private network, sentries public
- **Min Gas Price**: Set in `app.toml` to prevent spam (default: 0.025uMTC)
- **Parameter Changes**: Always test on testnet via governance proposal first
- **Upgrades**: Use Cosmovisor for coordinated upgrades without downtime

## Troubleshooting

**Build fails with "toolchain" errors**:
- Ensure Go 1.24+ installed: `go version`
- Update: `go install golang.org/dl/go1.24@latest`

**Proto generation fails**:
- Install Ignite: `curl -L https://get.ignite.com/cli! | bash`
- Verify: `ignite version`
- Run: `ignite doctor`

**Node won't sync**:
- Check peers: `appd status | jq .sync_info.catching_up`
- Add persistent peers in `~/.appd/config/config.toml`
- Use state sync or snapshots for fast sync

**Tests fail**:
- Clean build: `make clean && make build`
- Update deps: `go mod tidy`
- Check test logs for specific failures
