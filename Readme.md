# Cosmos Blockchain Builder: Complete SDK Development Toolkit

⚡ **Production-Ready** — Comprehensive toolkit for building secure, scalable Cosmos SDK blockchains

🎯 **Specialized Agents** — 8 domain experts covering architecture, configuration, DeFi, and security

A unified marketplace providing everything needed for intelligent blockchain development, from module scaffolding to security auditing, with 4 focused, single-purpose plugins for Cosmos SDK.

## Overview

This marketplace delivers a complete toolkit for Cosmos SDK blockchain development:

- **4 Focused Plugins** - Granular, single-purpose plugins optimized for minimal token usage and composability
- **8 Specialized Agents** - Domain experts in architecture, configuration, DeFi economics, and security auditing
- **12 Development Commands** - Optimized utilities for scaffolding, configuration, testing, and security scanning
- **Clear Organization** - 4 categories (Code Generation, Configuration, DeFi, Testing) for easy discovery

## Key Features

- **Granular Plugin Architecture**: 4 focused plugins optimized for minimal token usage
- **Comprehensive Tooling**: 12 development commands including module scaffolding, chain configuration, and security auditing
- **100% Agent Coverage**: All plugins include specialized agents with deep domain expertise
- **Clear Organization**: 4 categories with focused scope for easy discovery
- **Efficient Design**: Average 3 components per plugin (follows Anthropic's best practices)

## How It Works

Each plugin is completely isolated with its own agents and commands:

- **Install only what you need** - Each plugin loads only its specific agents and commands
- **Minimal token usage** - No unnecessary resources loaded into context
- **Mix and match** - Compose multiple plugins for complex workflows
- **Clear boundaries** - Each plugin has a single, focused purpose

**Example**: Installing `cosmos-scaffold` loads 2 agents and 3 commands (~200 tokens), not the entire marketplace.

## Quick Start

### Step 1: Add the Marketplace

Add this marketplace to Claude Code:

```
/plugin marketplace add https://github.com/fandomchain-network/claude-code-templates
```

This makes all 4 plugins available for installation, but does not load any agents or tools into your context.

### Step 2: Install Plugins

Browse available plugins:

```
/plugin
```

Install the plugins you need:

```bash
# Essential development plugins
/plugin install cosmos-scaffold          # Module scaffolding with 2 specialized agents
/plugin install chain-configuration      # Chain setup with 2 configuration agents

# DeFi & advanced features
/plugin install bonding-curves-amm       # AMM design with 2 DeFi agents

# Security & quality
/plugin install testing-validation       # Security auditing with 2 testing agents
```

Each installed plugin loads only its specific agents and commands into Claude's context.

## Plugins

### 🔨 cosmos-scaffold
**Category:** Code Generation  
**Agents:** 2 | **Commands:** 3

Code generation tools for building Cosmos SDK modules with automated scaffolding. Generate complete module structures, messages, queries, and protobuf definitions following best practices.

**Key Features:**
- Automated module creation with keeper, types, and handlers
- Message scaffolding with validation
- Query generation with protobuf definitions
- Protocol buffers management

**Agents:**
- `module-architect` - Module design and architecture expert
- `protobuf-engineer` - Protobuf definitions and code generation specialist

**Commands:**
- `/create-module` - Create new Cosmos SDK modules
- `/add-message` - Add message types to modules
- `/add-query` - Add query endpoints to modules

### ⚙️ chain-configuration
**Category:** Configuration  
**Agents:** 2 | **Commands:** 3

Blockchain configuration toolkit for genesis setup, validator configuration, IBC connections, and chain parameter management. Establish production-ready chain state from scratch.

**Key Features:**
- Genesis configuration with accounts and validators
- Validator management and consensus settings
- IBC setup for cross-chain communication
- Parameter management and governance

**Agents:**
- `genesis-configurator` - Genesis file configuration expert
- `ibc-architect` - IBC protocol setup specialist

**Commands:**
- `/setup-genesis` - Initialize genesis.json
- `/configure-validators` - Set up validator configuration
- `/setup-ibc` - Establish IBC connections

### 💰 bonding-curves-amm
**Category:** DeFi  
**Agents:** 2 | **Commands:** 3

Advanced DeFi toolkit for implementing bonding curves, automated market makers (AMM), TokenFactory modules, and trading mechanics. Build mathematically sound DeFi primitives.

**Key Features:**
- Bonding curve design (linear, exponential, constant product)
- TokenFactory module integration
- AMM implementation with various formulas
- Trading simulations and price discovery

**Agents:**
- `amm-architect` - Automated Market Maker design expert
- `curve-optimizer` - Bonding curve mathematics specialist

**Commands:**
- `/create-bonding-curve` - Implement bonding curves
- `/configure-tokenfactory` - Set up TokenFactory modules
- `/simulate-trades` - Test trading behavior

### 🧪 testing-validation
**Category:** Testing  
**Agents:** 2 | **Commands:** 3

Comprehensive testing and validation suite for unit tests, integration tests, simulations, and security audits. Ensure production-ready code quality.

**Key Features:**
- Test generation for modules and keepers
- Integration and simulation testing
- Security audits and vulnerability detection
- Performance and quality validation

**Agents:**
- `test-engineer` - Test design and implementation expert
- `security-auditor` - Security analysis and vulnerability detection specialist

**Commands:**
- `/generate-tests` - Generate test suites
- `/run-simulations` - Execute simulation testing
- `/security-audit` - Perform security analysis

## Documentation

### Core Guides

- **[Getting Started](./docs/getting-started.md)** - Setup and installation guide
- **[Plugin Architecture](./docs/architecture.md)** - How plugins work together
- **[Agent Usage](./docs/agents.md)** - Working with specialized agents
- **[Command Reference](./docs/commands.md)** - Complete command documentation

### Plugin Documentation

- [Cosmos Scaffold](./.claude-plugin/plugins/cosmos-scaffold/README.md) - Module scaffolding guide
- [Chain Configuration](./.claude-plugin/plugins/chain-configuration/README.md) - Chain setup guide
- [Bonding Curves & AMM](./.claude-plugin/plugins/bonding-curves-amm/README.md) - DeFi development guide
- [Testing & Validation](./.claude-plugin/plugins/testing-validation/README.md) - Testing guide

## Requirements

- **Cosmos SDK**: v0.50+
- **Go**: 1.21+
- **Protocol Buffers**: Latest version
- **Claude Code**: Latest version with plugin support

## Examples

### Full Workflow Example

```bash
# 1. Install all plugins
/plugin install cosmos-scaffold
/plugin install chain-configuration
/plugin install testing-validation

# 2. Create a new NFT marketplace module
/create-module nftmarket "NFT marketplace with auctions"

# 3. Add a create listing message
/add-message nftmarket CreateListing "nftId:string,price:uint64,duration:int64"

# 4. Add query for active auctions
/add-query nftmarket ListActiveAuctions "owner:string"

# 5. Generate comprehensive tests
/generate-tests nftmarket

# 6. Run security audit
/security-audit nftmarket

# 7. Setup testnet
/setup-genesis nftchain-testnet-1 unft

# 8. Configure validators
/configure-validators 4 1000000000unft
```

### DeFi Development Example

```bash
# Install DeFi plugin
/plugin install bonding-curves-amm

# Create constant product AMM
/create-bonding-curve constant-product reserve_base=30 reserve_token=1000000

# Configure TokenFactory
/configure-tokenfactory token-amm

# Simulate trading
/simulate-trades mytoken buy:100,buy:200,buy:500

# Run security audit
/security-audit tokenfactory
```

### IBC Integration Example

```bash
# Install chain configuration
/plugin install chain-configuration

# Setup IBC connection
/setup-ibc osmosis-1 connection-0

# Configure multi-chain topology
/setup-ibc juno-1 connection-1
```

## Categories

- **Code Generation** (1 plugin): Module scaffolding and code generation
- **Configuration** (1 plugin): Chain setup and genesis management
- **DeFi** (1 plugin): AMM and bonding curve implementation
- **Testing** (1 plugin): Testing, validation, and security auditing

## Support

- **Repository**: [GitHub](https://github.com/fandomchain/fandomchain-official-repo)
- **Issues**: [Report Issues](https://github.com/fandomchain/fandomchain-official-repo/issues)
- **Discussions**: [Community Forum](https://github.com/fandomchain/fandomchain-official-repo/discussions)
- **Email**: contact@fandomchain.com

## License

MIT License - See [LICENSE](./LICENSE) for details

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

---

**Built by the FandomChain Team** | Optimized for Claude Sonnet 4.5 & Haiku 4.5

