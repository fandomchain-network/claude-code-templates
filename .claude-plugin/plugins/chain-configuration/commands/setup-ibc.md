# Setup IBC Connections

You are an expert in Inter-Blockchain Communication (IBC) protocol. Your task is to establish IBC connections between Cosmos SDK blockchains, including light client creation, connection establishment, channel opening, and relayer configuration.

## Task Overview

Set up IBC with:
1. Light client creation
2. Connection establishment
3. Channel opening for token transfers
4. Relayer configuration (Hermes or rly)
5. Testing IBC transfers

## Instructions

### 1. Information Gathering

Ask for:
- **Source chain** (your chain details: chain-id, RPC, etc.)
- **Destination chain** (counterparty chain: Osmosis, Cosmos Hub, etc.)
- **Relayer choice** (Hermes recommended, or rly)
- **Channel type** (transfer, ICA, or custom)
- **Connection type** (new connection or existing)

### 2. Prerequisites

```bash
# Ensure IBC module is included in your chain
# Check app/app_config.go includes:
# - ibc
# - transfer
# - capability
# - ica (if using)

# Verify IBC genesis params in genesis.json
{app}d query ibc client params
{app}d query ibc connection params
```

### 3. Install Relayer

#### Option A: Hermes (Recommended)
```bash
# Install Hermes
cargo install ibc-relayer-cli --bin hermes --locked

# Or download binary
wget https://github.com/informalsystems/hermes/releases/download/v1.7.0/hermes-v1.7.0-x86_64-unknown-linux-gnu.tar.gz
tar -xzf hermes-v1.7.0-x86_64-unknown-linux-gnu.tar.gz
sudo mv hermes /usr/local/bin/

# Verify installation
hermes version
```

#### Option B: Go Relayer (rly)
```bash
git clone https://github.com/cosmos/relayer.git
cd relayer
make install

# Verify
rly version
```

### 4. Configure Relayer (Hermes)

Create `~/.hermes/config.toml`:
```toml
[global]
log_level = 'info'

[mode.clients]
enabled = true
refresh = true
misbehaviour = true

[mode.connections]
enabled = false

[mode.channels]
enabled = false

[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true

[rest]
enabled = true
host = '127.0.0.1'
port = 3000

[telemetry]
enabled = true
host = '127.0.0.1'
port = 3001

# Source Chain (Your chain)
[[chains]]
id = 'mychain-1'
type = 'CosmosSdk'
rpc_addr = 'http://localhost:26657'
grpc_addr = 'http://localhost:9090'
event_source = { mode = 'push', url = 'ws://localhost:26657/websocket', batch_delay = '500ms' }
rpc_timeout = '10s'
account_prefix = 'mychain'
key_name = 'relayer'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 400000
gas_price = { price = 0.025, denom = 'ustake' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 180000
clock_drift = '5s'
max_block_time = '30s'
trusting_period = '14days'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }

# Destination Chain (e.g., Osmosis)
[[chains]]
id = 'osmosis-1'
type = 'CosmosSdk'
rpc_addr = 'https://rpc.osmosis.zone:443'
grpc_addr = 'https://grpc.osmosis.zone:443'
event_source = { mode = 'push', url = 'wss://rpc.osmosis.zone:443/websocket', batch_delay = '500ms' }
rpc_timeout = '10s'
account_prefix = 'osmo'
key_name = 'relayer-osmosis'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 400000
gas_price = { price = 0.0025, denom = 'uosmo' }
gas_multiplier = 1.1
max_msg_num = 30
max_tx_size = 180000
clock_drift = '5s'
max_block_time = '30s'
trusting_period = '10days'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }
```

### 5. Add Relayer Keys

```bash
# Generate key for source chain
{app}d keys add relayer

# Import to Hermes
hermes keys add --chain mychain-1 --mnemonic-file <path-to-mnemonic>

# Repeat for destination chain
hermes keys add --chain osmosis-1 --mnemonic-file <path-to-mnemonic>

# Fund relayer addresses with tokens for gas fees
# Source chain: 100+ tokens
# Destination chain: 100+ tokens
```

### 6. Create IBC Client

```bash
# Create client on both chains
hermes create client --host-chain mychain-1 --reference-chain osmosis-1
hermes create client --host-chain osmosis-1 --reference-chain mychain-1

# Verify clients created
{app}d query ibc client states
```

### 7. Create IBC Connection

```bash
# Create connection between the two clients
hermes create connection --a-chain mychain-1 --b-chain osmosis-1

# This creates connection-0 on both chains
# Verify connections
{app}d query ibc connection connections
```

### 8. Create IBC Channel

```bash
# Create transfer channel
hermes create channel \
  --a-chain mychain-1 \
  --a-connection connection-0 \
  --a-port transfer \
  --b-port transfer

# This creates channel-0 for IBC transfers
# Verify channels
{app}d query ibc channel channels
```

### 9. Start Relayer

```bash
# Start Hermes in daemon mode
hermes start

# Or specific paths
hermes start --path mychain-osmosis

# Monitor logs
tail -f ~/.hermes/hermes.log
```

### 10. Test IBC Transfer

#### Send tokens from Source to Destination
```bash
# Transfer tokens via IBC
{app}d tx ibc-transfer transfer \
  transfer \
  channel-0 \
  osmo1... \
  100ustake \
  --from relayer \
  --chain-id mychain-1 \
  --node http://localhost:26657

# Wait for relayer to relay (usually 10-30 seconds)
# Check destination balance
osmosisd query bank balances osmo1...
```

#### Check IBC Denom
```bash
# IBC tokens have hash-based denoms
# Format: ibc/<hash of channel path and denom>
# Example: ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2
```

## Relayer Monitoring

### Check Relayer Status
```bash
# Hermes health check
curl http://localhost:3000/health

# List connections
hermes query connections --chain mychain-1

# List channels
hermes query channels --chain mychain-1
```

### Monitor Packets
```bash
# Check pending packets
hermes query packet pending --chain mychain-1 --port transfer --channel channel-0

# Clear pending packets
hermes clear packets --chain mychain-1 --port transfer --channel channel-0
```

## Advanced Configuration

### Multiple Channels
```bash
# ICS-20 Transfer channel
hermes create channel --a-chain mychain-1 --a-port transfer --b-port transfer

# ICS-27 Interchain Accounts
hermes create channel --a-chain mychain-1 --a-port icahost --b-port icahost
```

### Packet Filtering
```toml
# In config.toml
[[chains]]
id = 'mychain-1'
# ...

[chains.packet_filter]
policy = 'allow'
list = [
  ['transfer', 'channel-0'],
  ['transfer', 'channel-1'],
]
```

## Troubleshooting

**Issue**: Client expired
```bash
# Update client
hermes update client --host-chain mychain-1 --client 07-tendermint-0
```

**Issue**: Connection not working
```bash
# Check connection state
{app}d query ibc connection end connection-0

# State should be: STATE_OPEN
```

**Issue**: Packets stuck
```bash
# Clear packets manually
hermes clear packets --chain mychain-1 --port transfer --channel channel-0

# Check packet commitments
{app}d query ibc channel packet-commitments transfer channel-0
```

**Issue**: Relayer out of funds
```bash
# Check relayer balance
{app}d query bank balances $(hermes keys list --chain mychain-1)

# Add more funds
{app}d tx bank send <from> <relayer-address> 1000ustake
```

## IBC Transfer Examples

### Basic Transfer
```bash
{app}d tx ibc-transfer transfer transfer channel-0 \
  <recipient-address> 100ustake \
  --from <sender> \
  --packet-timeout-height 0-0 \
  --packet-timeout-timestamp 0
```

### Transfer with Timeout
```bash
# 1 hour timeout
TIMEOUT=$(($(date +%s) + 3600))000000000

{app}d tx ibc-transfer transfer transfer channel-0 \
  <recipient> 100ustake \
  --from <sender> \
  --packet-timeout-timestamp $TIMEOUT
```

### Query IBC Balance
```bash
# On destination, IBC tokens use hashed denom
{app}d query bank balances <address> | grep ibc
```

## Production Considerations

1. **Relayer Redundancy**: Run multiple relayers for reliability
2. **Monitoring**: Set up alerts for relayer failures
3. **Gas Management**: Monitor and refill relayer addresses
4. **Client Updates**: Automate client update submissions
5. **Packet Clearing**: Regular packet clearing to avoid congestion

## Resources

- IBC Documentation: https://ibc.cosmos.network
- Hermes Guide: https://hermes.informal.systems
- IBC Channel Registry: https://github.com/cosmos/chain-registry
