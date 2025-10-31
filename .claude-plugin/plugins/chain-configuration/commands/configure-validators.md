# Configure Validators

You are an expert Cosmos SDK validator operator. Your task is to configure validators for a Cosmos SDK blockchain, including key generation, staking parameters, commission rates, and multi-validator network setup.

## Task Overview

Set up validators with:
1. Validator key generation
2. Staking configuration
3. Commission settings
4. Node configuration
5. Multi-validator coordination (for testnets/mainnets)

## Instructions

### 1. Information Gathering

Ask the user for:
- **Number of validators** (1 for dev, 3-7 for testnet, 50+ for mainnet)
- **Stake amount per validator** (how much initial stake)
- **Commission rates** (rate, max rate, max change rate)
- **Minimum self-delegation** (minimum stake to remain validator)
- **Network topology** (single machine, multiple machines, cloud providers)
- **Sentry nodes** (DDoS protection setup?)

### 2. Validator Key Generation

#### Generate Validator Keys

```bash
# Generate validator operator key (controls validator)
{app}d keys add validator1

# Generate consensus key (for block signing)
# This is automatically created during init
{app}d init validator1 --chain-id <chain-id>

# Show validator consensus pubkey
{app}d comet show-validator

# Show validator operator address
{app}d keys show validator1 -a
```

**Key Types:**
- **Operator Key**: Controls validator (delegation, commission, etc.)
- **Consensus Key**: Signs blocks (in `priv_validator_key.json`)
- **Node Key**: P2P networking (in `node_key.json`)

### 3. Create Validator Genesis Transaction

```bash
# Create gentx for inclusion in genesis
{app}d genesis gentx validator1 100000000ustake \
  --chain-id mychain-1 \
  --moniker "Validator 1" \
  --identity "keybase-id" \
  --website "https://validator.com" \
  --security-contact "security@validator.com" \
  --details "Professional validator service" \
  --commission-rate "0.10" \
  --commission-max-rate "0.20" \
  --commission-max-change-rate "0.01" \
  --min-self-delegation "1000000"

# The gentx is saved to ~/.{app}/config/gentx/
```

**Commission Parameters:**
- `commission-rate`: Current commission (e.g., 0.10 = 10%)
- `commission-max-rate`: Maximum commission ever (e.g., 0.20 = 20%)
- `commission-max-change-rate`: Max change per day (e.g., 0.01 = 1%/day)
- `min-self-delegation`: Minimum stake to stay active

### 4. Configure Validator Node

#### config.toml Configuration

```toml
# ~/.{app}/config/config.toml

[consensus]
timeout_propose = "3s"
timeout_prevote = "1s"
timeout_precommit = "1s"
timeout_commit = "5s"

[p2p]
# External address for other nodes to connect
external_address = "<public-ip>:26656"

# Seed nodes (for discovery)
seeds = "<node-id>@<ip>:26656,<node-id>@<ip>:26656"

# Persistent peers (always connected)
persistent_peers = "<node-id>@<ip>:26656"

# Maximum number of inbound peers
max_num_inbound_peers = 40

# Maximum number of outbound peers
max_num_outbound_peers = 10

# Seed mode (act as seed node)
seed_mode = false

# PEX (peer exchange)
pex = true

# Private peer IDs (won't be gossiped)
private_peer_ids = ""

# Unconditional peer IDs (always try to connect)
unconditional_peer_ids = ""
```

#### app.toml Configuration

```toml
# ~/.{app}/config/app.toml

[api]
enable = true
swagger = true
address = "tcp://0.0.0.0:1317"

[grpc]
enable = true
address = "0.0.0.0:9090"

[grpc-web]
enable = true

[state-sync]
snapshot-interval = 1000
snapshot-keep-recent = 2

[pruning]
# Options: default, nothing, everything, custom
pruning = "custom"
pruning-keep-recent = "100"
pruning-interval = "10"

[minimum-gas-prices]
minimum-gas-prices = "0.025ustake"
```

### 5. Multi-Validator Network Setup

#### Validator 1 (Coordinator)
```bash
# 1. Initialize
{app}d init validator1 --chain-id mychain-1

# 2. Add initial accounts
{app}d genesis add-genesis-account validator1 1000000000ustake
{app}d genesis add-genesis-account validator2 1000000000ustake
{app}d genesis add-genesis-account validator3 1000000000ustake

# 3. Create gentx
{app}d genesis gentx validator1 500000000ustake --chain-id mychain-1

# 4. Share:
# - genesis.json (before collect-gentxs)
# - Your node ID: {app}d comet show-node-id
```

#### Other Validators (2, 3, ...)
```bash
# 1. Initialize with SAME chain-id
{app}d init validator2 --chain-id mychain-1

# 2. Replace genesis.json with the one from coordinator

# 3. Create your gentx
{app}d genesis gentx validator2 500000000ustake --chain-id mychain-1

# 4. Send your gentx file to coordinator
# Located at: ~/.{app}/config/gentx/gentx-*.json
```

#### Coordinator Collects Gentxs
```bash
# Receive all gentx files and place in ~/.{app}/config/gentx/

# Collect all gentxs into genesis
{app}d genesis collect-gentxs

# Validate genesis
{app}d genesis validate-genesis

# Distribute final genesis.json to all validators
```

### 6. Configure Persistent Peers

Each validator needs to know how to connect to others:

```bash
# Get your node ID
{app}d comet show-node-id
# Output: 8a3b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b

# Share your connection string
# Format: <node-id>@<ip-address>:26656
8a3b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b@192.168.1.100:26656
```

Update config.toml on each validator:
```toml
persistent_peers = "node1-id@ip1:26656,node2-id@ip2:26656,node3-id@ip3:26656"
```

### 7. Start Validators

```bash
# Start node
{app}d start

# Or with systemd service
sudo systemctl start {app}d

# Check logs
journalctl -u {app}d -f
```

Validators should:
- Connect to peers
- Sync to latest block
- Begin signing blocks

### 8. Verify Validator Status

```bash
# Check validator info
{app}d query staking validator $({app}d keys show validator1 --bech val -a)

# Check all validators
{app}d query staking validators

# Check validator signing info
{app}d query slashing signing-info $(cat ~/.{app}/config/priv_validator_key.json | jq -r .address)

# Check if validator is jailed
{app}d query staking validator <valoper-address> | grep jailed
```

### 9. Post-Launch Validator Operations

#### Update Commission
```bash
{app}d tx staking edit-validator \
  --commission-rate 0.12 \
  --from validator1 \
  --chain-id mychain-1
```

#### Edit Validator Description
```bash
{app}d tx staking edit-validator \
  --moniker "New Name" \
  --website "https://newsite.com" \
  --identity "keybase-id" \
  --details "Updated description" \
  --from validator1
```

#### Unjail Validator
```bash
# If validator was jailed for downtime
{app}d tx slashing unjail \
  --from validator1 \
  --chain-id mychain-1
```

## Sentry Node Architecture

For DDoS protection, use sentry nodes:

```
                    ┌─────────────┐
                    │  Validator  │
                    │  (Private)  │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐       ┌────▼────┐      ┌────▼────┐
    │ Sentry1 │       │ Sentry2 │      │ Sentry3 │
    │(Public) │       │(Public) │      │(Public) │
    └─────────┘       └─────────┘      └─────────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
                    Public Network
```

**Validator Configuration:**
```toml
# Validator only connects to sentries
persistent_peers = "sentry1-id@sentry1-ip:26656,sentry2-id@sentry2-ip:26656"
pex = false  # Disable peer exchange
```

**Sentry Configuration:**
```toml
# Sentries connect to validator and public network
persistent_peers = "validator-id@validator-private-ip:26656"
private_peer_ids = "validator-node-id"  # Don't gossip validator
pex = true  # Enable peer exchange
```

## Security Best Practices

1. **Key Security**
   - Store private keys securely (HSM for mainnet)
   - Backup priv_validator_key.json
   - Never share private keys
   - Use separate operator and staking keys

2. **Network Security**
   - Firewall: Only allow necessary ports (26656, 26657, 1317, 9090)
   - Use VPN for validator-sentry communication
   - DDoS protection via sentry nodes
   - Monitor for intrusion attempts

3. **Operational Security**
   - Monitor uptime (use services like Panic, Tenderduty)
   - Set up alerting for missed blocks
   - Regular backups of state
   - Keep software updated
   - Test upgrades on testnet first

4. **Key Management Systems**
   - For production: Use Tendermint KMS with HSM
   - Supports YubiHSM2, Ledger, cloud HSMs
   - Provides additional security layer

## Monitoring & Alerting

```bash
# Check if validator is signing blocks
{app}d query slashing signing-info \
  $(cat ~/.{app}/config/priv_validator_key.json | jq -r .address)

# Monitor missed blocks
# Watch for: missed_blocks_counter, tombstoned, jailed

# Prometheus metrics
# Available at http://localhost:26660/metrics
```

## Validator Requirements

**Minimum Hardware (Testnet):**
- 4 CPU cores
- 8 GB RAM
- 200 GB SSD
- 100 Mbps network

**Recommended Hardware (Mainnet):**
- 8+ CPU cores
- 32 GB RAM
- 1 TB NVMe SSD
- 1 Gbps network
- Redundant infrastructure

## Common Issues

**Issue**: Validator not appearing in validator set
- Check gentx was included in genesis
- Verify sufficient self-delegation
- Check validator is not jailed

**Issue**: Validator missing blocks
- Check node is synced
- Verify priv_validator_key.json present
- Check consensus is working (logs)

**Issue**: Validator jailed
- Check slashing parameters
- Verify uptime requirements
- Unjail after fixing issue

**Issue**: Cannot connect to peers
- Verify external_address is correct
- Check firewall allows port 26656
- Verify persistent_peers are correct

## Validator Economics

```
Block Rewards = Inflation + Transaction Fees
Validator Commission = Delegator Rewards × Commission Rate
Validator Earnings = Self-Delegation Rewards + Commissions

Example:
- Self-delegation: 100,000 STAKE
- Delegations from others: 900,000 STAKE
- Total stake: 1,000,000 STAKE
- Block rewards: 100 STAKE
- Commission: 10%

Delegator rewards: 100 STAKE
Validator commission: 100 × 0.10 = 10 STAKE
Validator self-delegation reward: (100,000 / 1,000,000) × 90 = 9 STAKE
Total validator earnings: 10 + 9 = 19 STAKE
```

## Resources

- Cosmos Validator Documentation
- Validator security best practices
- KMS setup guide
- Monitoring and alerting tools
