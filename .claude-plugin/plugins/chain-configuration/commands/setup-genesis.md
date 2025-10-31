# Setup Genesis Configuration

You are an expert Cosmos SDK blockchain operator. Your task is to initialize and configure the genesis.json file for a new blockchain, setting up initial state, accounts, validators, and module parameters.

## Task Overview

Create a production-ready genesis.json with:
1. Chain metadata (chain-id, genesis_time)
2. Initial accounts and balances
3. Validator genesis transactions
4. Module parameters (staking, gov, bank, mint, distribution, slashing)
5. Custom module genesis state
6. Proper validation

## Instructions

### 1. Information Gathering

Ask the user for:
- **Chain ID** (e.g., "mychain-1", "testnet-1")
- **Base denomination** (e.g., "ustake", "utoken")
- **Initial accounts** (addresses and balances)
- **Validator count** (single validator or multi-validator network)
- **Network type** (mainnet, testnet, devnet)
- **Custom module params** (if any modules need non-default settings)
- **Genesis time** (start time, or "now")

### 2. Genesis Initialization

#### Step 1: Initialize Chain
```bash
# Initialize node with chain ID
{app}d init <moniker> --chain-id <chain-id>

# This creates ~/.{app}/config/genesis.json with defaults
```

#### Step 2: Configure Chain Metadata

Edit genesis.json:
```json
{
  "chain_id": "mychain-1",
  "genesis_time": "2024-01-01T00:00:00Z",
  "initial_height": "1",
  "app_state": {
    // Module genesis states...
  }
}
```

### 3. Add Initial Accounts

```bash
# Add accounts with initial balances
{app}d genesis add-genesis-account <address> 1000000000000{denom}
{app}d genesis add-genesis-account <address2> 500000000000{denom}

# For development, create test keys
{app}d keys add alice
{app}d keys add bob
{app}d genesis add-genesis-account alice 1000000000{denom}
{app}d genesis add-genesis-account bob 1000000000{denom}
```

### 4. Configure Module Parameters

#### Bank Module
```json
"bank": {
  "params": {
    "send_enabled": [],
    "default_send_enabled": true
  },
  "balances": [
    {
      "address": "cosmos1...",
      "coins": [{"denom": "ustake", "amount": "1000000000"}]
    }
  ],
  "supply": [],
  "denom_metadata": [
    {
      "description": "The native staking token",
      "denom_units": [
        {"denom": "ustake", "exponent": 0},
        {"denom": "stake", "exponent": 6}
      ],
      "base": "ustake",
      "display": "stake",
      "name": "Stake Token",
      "symbol": "STAKE"
    }
  ]
}
```

#### Staking Module
```json
"staking": {
  "params": {
    "unbonding_time": "1814400s",  // 21 days
    "max_validators": 100,
    "max_entries": 7,
    "historical_entries": 10000,
    "bond_denom": "ustake",
    "min_commission_rate": "0.050000000000000000"  // 5%
  },
  "validators": [],
  "delegations": []
}
```

#### Gov Module (Governance)
```json
"gov": {
  "params": {
    "min_deposit": [
      {"denom": "ustake", "amount": "10000000"}  // 10 stake
    ],
    "max_deposit_period": "172800s",  // 2 days
    "voting_period": "172800s",  // 2 days
    "quorum": "0.334000000000000000",  // 33.4%
    "threshold": "0.500000000000000000",  // 50%
    "veto_threshold": "0.334000000000000000",  // 33.4%
    "min_initial_deposit_ratio": "0.000000000000000000",
    "burn_vote_quorum": false,
    "burn_proposal_deposit_prevote": false,
    "burn_vote_veto": true
  }
}
```

#### Mint Module
```json
"mint": {
  "minter": {
    "inflation": "0.130000000000000000",  // 13% initial
    "annual_provisions": "0.000000000000000000"
  },
  "params": {
    "mint_denom": "ustake",
    "inflation_rate_change": "0.130000000000000000",
    "inflation_max": "0.200000000000000000",  // 20%
    "inflation_min": "0.070000000000000000",  // 7%
    "goal_bonded": "0.670000000000000000",  // 67%
    "blocks_per_year": "6311520"
  }
}
```

#### Distribution Module
```json
"distribution": {
  "params": {
    "community_tax": "0.020000000000000000",  // 2%
    "base_proposer_reward": "0.000000000000000000",
    "bonus_proposer_reward": "0.000000000000000000",
    "withdraw_addr_enabled": true
  },
  "fee_pool": {
    "community_pool": []
  }
}
```

#### Slashing Module
```json
"slashing": {
  "params": {
    "signed_blocks_window": "100",
    "min_signed_per_window": "0.500000000000000000",  // 50%
    "downtime_jail_duration": "600s",  // 10 minutes
    "slash_fraction_double_sign": "0.050000000000000000",  // 5%
    "slash_fraction_downtime": "0.010000000000000000"  // 1%
  }
}
```

### 5. Add Validators

#### For Single Validator (Development)
```bash
# Create validator genesis transaction
{app}d genesis gentx <key-name> 100000000{denom} \
  --chain-id <chain-id> \
  --moniker "validator1" \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1

# Collect genesis transactions
{app}d genesis collect-gentxs
```

#### For Multiple Validators (Testnet/Mainnet)
```bash
# Each validator generates their gentx
{app}d genesis gentx <key> <amount> --chain-id <chain-id>

# Coordinator collects all gentx files
# Place all gentx files in ~/.{app}/config/gentx/

# Collect all genesis transactions
{app}d genesis collect-gentxs
```

### 6. Custom Module Genesis State

If you have custom modules (e.g., tokenfactory):
```json
"tokenfactory": {
  "params": {
    "min_create_deposit": "1000000",
    "buy_fee_bps": "100",
    "sell_fee_bps": "100"
  },
  "denoms": [],
  "port_id": "tokenfactory"
}
```

### 7. Validation

```bash
# Validate genesis file
{app}d genesis validate-genesis

# Check for common issues:
# - All addresses are valid bech32
# - Total supply matches balances
# - Module params are within valid ranges
# - Validator gentxs are valid
```

Common validation checks:
- Chain ID matches everywhere
- Genesis time is reasonable
- Total bank supply equals sum of balances
- Staking bond denom matches bank denom
- Validator stakes don't exceed available balances
- All addresses use correct prefix

### 8. Export and Distribution

```bash
# Export genesis to file
cp ~/.{app}/config/genesis.json ./genesis.json

# Generate checksum for verification
sha256sum genesis.json > genesis.json.sha256

# For networks, distribute genesis.json to all validators
```

## Configuration Templates

### Development Network (Single Validator)
- 1-3 pre-funded test accounts
- Single validator with 100M tokens
- Fast governance (1 minute voting)
- Low slashing penalties
- Short unbonding (1 day)

### Testnet (Multi-Validator)
- 10-20 validators
- Moderate initial funding
- Standard governance (2 days)
- Standard penalties
- Standard unbonding (21 days)

### Mainnet (Production)
- 50-100+ validators
- Carefully allocated initial supply
- Conservative governance (7+ days)
- Production slashing params
- Long unbonding (21-28 days)

## Best Practices

1. **Chain ID Format**: Use `<name>-<version>` (e.g., "mychain-1")
2. **Genesis Time**: Set to future time for coordinated launch
3. **Token Supply**: Plan total supply and distribution carefully
4. **Validator Diversity**: Encourage geographic and entity diversity
5. **Governance Params**: Conservative initially, can be changed via governance
6. **Testing**: Test genesis on devnet before testnet/mainnet
7. **Documentation**: Document all parameter choices
8. **Backups**: Keep backups of genesis and keys
9. **Coordination**: Use tools like gentx ceremony for mainnet launches

## Security Considerations

- **Key Management**: Never commit private keys to genesis
- **Initial Distribution**: Avoid excessive centralization
- **Validator Requirements**: Set reasonable minimums
- **Governance**: Ensure no single entity has veto power
- **Emergency Params**: Consider circuit breakers
- **Audit**: Have genesis reviewed before mainnet

## Common Issues

**Issue**: "genesis transaction failed validation"
- Check validator has sufficient balance for stake
- Verify commission rates are within bounds
- Ensure chain-id matches

**Issue**: "invalid genesis state"
- Run `validate-genesis` for detailed errors
- Check all denoms match
- Verify addresses use correct prefix

**Issue**: "total supply mismatch"
- Sum all account balances
- Add validator stakes
- Must equal total supply in bank module

## Example Complete Workflow

```bash
# 1. Initialize
mychain init validator1 --chain-id mychain-1

# 2. Add accounts
mychain keys add alice
mychain genesis add-genesis-account alice 1000000000ustake

# 3. Configure genesis.json (edit manually or via commands)
# Update params as needed

# 4. Create validator
mychain genesis gentx alice 500000000ustake \
  --chain-id mychain-1 \
  --moniker validator1 \
  --commission-rate 0.10

# 5. Collect gentxs
mychain genesis collect-gentxs

# 6. Validate
mychain genesis validate-genesis

# 7. Start chain
mychain start
```

## Additional Resources

- Cosmos SDK Genesis Documentation
- Chain parameter recommendations
- Mainnet launch checklist
- Genesis ceremony best practices
