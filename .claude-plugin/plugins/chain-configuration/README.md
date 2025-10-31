# Chain Configuration Plugin

**Version:** 1.0.0
**Category:** Configuration
**License:** MIT

## Overview

The Chain Configuration plugin provides comprehensive tools for setting up and managing Cosmos SDK blockchain configuration, including genesis state, validator setup, IBC connections, and chain parameters.

## Features

- **Genesis Configuration**: Initialize and customize genesis.json with accounts, validators, and module params
- **Validator Management**: Configure validators, staking parameters, and consensus settings
- **IBC Setup**: Establish IBC connections, create channels, and configure light clients
- **Parameter Management**: Update module parameters and governance settings
- **Network Deployment**: Multi-node network configuration for testnets and mainnets

## Commands

### `/setup-genesis`
Initializes and configures the genesis.json file with:
- Initial accounts and balances
- Validator configurations
- Module parameters (staking, gov, bank, etc.)
- Chain metadata (chain-id, denom, etc.)
- Custom module genesis state

**Usage:** `/setup-genesis <chain-id> <base-denom>`

### `/configure-validators`
Sets up validator configuration including:
- Validator keys generation
- Staking parameters
- Commission rates
- Self-delegation amounts
- Consensus configuration
- Multi-validator setup for networks

**Usage:** `/configure-validators <num-validators> <stake-amount>`

### `/setup-ibc`
Establishes IBC connections with other chains:
- Creates IBC clients (light clients)
- Establishes connections
- Opens channels for token transfers
- Configures relayer settings
- Tests IBC transfers

**Usage:** `/setup-ibc <counterparty-chain> <connection-id>`

## Agents

### `genesis-configurator`
**Model:** Sonnet
**Specialization:** Genesis file configuration and chain initialization

Expert agent for designing and configuring genesis state. Helps set up initial chain state, configure module parameters, add initial accounts and validators, and ensure proper chain initialization.

**Use cases:**
- Creating genesis.json from scratch
- Migrating genesis state between versions
- Adding custom module genesis state
- Validating genesis file correctness
- Multi-chain genesis coordination

### `ibc-architect`
**Model:** Sonnet
**Specialization:** IBC protocol setup and cross-chain communication

Specialized agent for establishing IBC connections between chains. Handles light client creation, connection establishment, channel opening, and relayer configuration for robust cross-chain communication.

**Use cases:**
- Setting up IBC connections
- Troubleshooting IBC issues
- Optimizing relayer configuration
- Multi-chain IBC topology design
- IBC upgrade coordination

## Requirements

- Cosmos SDK v0.50+
- IBC-Go v8+
- Go 1.21+
- Hermes or rly (for IBC relaying)

## Installation

This plugin is part of the Cosmos Blockchain Builder marketplace. Install via Claude Code marketplace or manually include the plugin directory.

## Examples

```bash
# Initialize genesis for a new testnet
/setup-genesis mychain-testnet-1 ustake

# Configure a 4-validator network
/configure-validators 4 100000000ustake

# Establish IBC connection to Osmosis
/setup-ibc osmosis-1 connection-0
```

## Configuration Files

This plugin helps manage:
- `genesis.json` - Initial chain state
- `config.toml` - Node configuration
- `app.toml` - Application configuration
- `client.toml` - Client configuration

## Related Plugins

- **cosmos-scaffold**: For creating custom modules
- **testing-validation**: For validating chain configuration
- **bonding-curves-amm**: For DeFi-specific genesis configuration

## Support

For issues, questions, or contributions, visit: https://github.com/fandomchain/fandomchain-official-repo
