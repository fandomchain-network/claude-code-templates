# Cosmos Scaffold Plugin

**Version:** 1.0.0
**Category:** Code Generation
**License:** MIT

## Overview

The Cosmos Scaffold plugin provides powerful code generation tools for building Cosmos SDK modules. It automates the creation of modules, messages, queries, and protobuf definitions, following Cosmos SDK best practices.

## Features

- **Automated Module Creation**: Generate complete module structure with keeper, types, and handlers
- **Message Scaffolding**: Add new message types with proper validation and handlers
- **Query Generation**: Create query endpoints with protobuf definitions
- **Protobuf Management**: Automated proto file generation and compilation
- **Best Practices**: Follows Cosmos SDK v0.50+ conventions and patterns

## Commands

### `/create-module`
Creates a new Cosmos SDK module with complete structure including:
- Module directory structure
- Keeper with state management
- Protobuf definitions (tx.proto, query.proto, genesis.proto)
- Message and query handlers
- Module registration in app.go

**Usage:** `/create-module <module-name> <description>`

### `/add-message`
Adds a new message type to an existing module:
- Message protobuf definition
- Message handler in keeper
- Validation logic
- Event emission
- CLI command generation

**Usage:** `/add-message <module-name> <message-name> <fields>`

### `/add-query`
Adds a new query endpoint to an existing module:
- Query protobuf definition
- Query handler in keeper
- Response type definition
- CLI query command

**Usage:** `/add-query <module-name> <query-name> <params>`

## Agents

### `module-architect`
**Model:** Sonnet
**Specialization:** Module design and architecture

Expert agent for designing complex Cosmos SDK modules. Analyzes requirements, proposes optimal module structure, designs state management patterns, and ensures proper integration with other modules.

**Use cases:**
- Designing new modules from requirements
- Refactoring existing module architecture
- Optimizing keeper dependencies
- Planning module interactions

### `protobuf-engineer`
**Model:** Sonnet
**Specialization:** Protobuf definitions and code generation

Specialized agent for creating and maintaining protobuf definitions. Handles proto3 syntax, service definitions, message design, and ensures proper code generation with make proto-gen.

**Use cases:**
- Creating protobuf definitions
- Updating proto files
- Managing proto breaking changes
- Optimizing message serialization

## Requirements

- Cosmos SDK v0.50+
- Go 1.21+
- Protocol Buffers compiler
- Ignite CLI (optional, for advanced scaffolding)

## Installation

This plugin is part of the Cosmos Blockchain Builder marketplace. Install via Claude Code marketplace or manually include the plugin directory.

## Examples

```bash
# Create a new module for NFT management
/create-module nftmarket "NFT marketplace with auctions"

# Add a message to create an NFT listing
/add-message nftmarket CreateListing "nftId:string,price:uint64,duration:int64"

# Add a query to list active auctions
/add-query nftmarket ListActiveAuctions "owner:string"
```

## Related Plugins

- **chain-configuration**: For setting up genesis and validators
- **testing-validation**: For generating tests for your modules
- **bonding-curves-amm**: For DeFi-specific module patterns

## Support

For issues, questions, or contributions, visit: https://github.com/fandomchain/fandomchain-official-repo
