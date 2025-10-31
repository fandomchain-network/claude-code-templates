# Testing & Validation Plugin

**Version:** 1.0.0
**Category:** Testing
**License:** MIT

## Overview

Comprehensive testing and validation suite for Cosmos SDK blockchains. Provides tools for unit testing, integration testing, simulations, security audits, and quality assurance.

## Features

- **Test Generation**: Automated test generation for modules, keepers, and handlers
- **Integration Testing**: End-to-end testing with simulated chains
- **Security Audits**: Automated security analysis and vulnerability detection
- **Simulation Testing**: App simulation framework for chaos testing
- **Performance Testing**: Load testing and benchmarking
- **Code Quality**: Linting, formatting, and best practice validation

## Commands

### `/generate-tests`
Generates comprehensive test suites for:
- Keeper tests (state management)
- Message handler tests (transactions)
- Query tests (read operations)
- Genesis import/export tests
- Simulation tests

**Usage:** `/generate-tests <module-name>`

### `/run-simulations`
Executes simulation testing:
- Non-determinism detection
- Invariant checking
- Random operation sequences
- State machine testing
- Import/export validation

**Usage:** `/run-simulations <duration> <num-blocks>`

### `/security-audit`
Performs security analysis:
- Common vulnerability scanning
- Integer overflow detection
- Authorization check validation
- Input validation review
- Gas consumption analysis
- Reentrancy checks

**Usage:** `/security-audit <module-name>`

## Agents

### `test-engineer`
**Model:** Sonnet
**Specialization:** Test design and implementation

Expert in writing comprehensive test suites for Cosmos SDK modules. Designs test strategies, implements tests, and ensures code coverage.

**Use cases:**
- Generating keeper test suites
- Writing message handler tests
- Creating integration tests
- Implementing simulation tests
- Achieving high code coverage

### `security-auditor`
**Model:** Sonnet
**Specialization:** Security analysis and vulnerability detection

Security expert for analyzing Cosmos SDK code. Identifies vulnerabilities, reviews authorization logic, validates input handling, and ensures secure coding practices.

**Use cases:**
- Security code reviews
- Vulnerability scanning
- Authorization logic validation
- Input sanitization review
- Economic attack analysis

## Testing Patterns

### Unit Tests (Keeper)
```go
func TestKeeperCreateDenom(t *testing.T) {
    keeper, ctx := setupKeeper(t)
    // Test logic
}
```

### Message Handler Tests
```go
func TestMsgServerCreateDenom(t *testing.T) {
    // Test happy path
    // Test error cases
    // Test events
}
```

### Simulation Tests
```go
func SimulateMsgCreateDenom() simtypes.Operation {
    // Random operation generation
}
```

## Requirements

- Go 1.21+
- Cosmos SDK v0.50+
- Testing frameworks: testify, mockery

## Installation

Part of Cosmos Blockchain Builder marketplace.

## Examples

```bash
# Generate tests for tokenfactory module
/generate-tests tokenfactory

# Run 100 block simulation
/run-simulations 100 --seed 42

# Audit bonding curve security
/security-audit tokenfactory
```

## Test Coverage Goals

- Unit tests: 80%+ coverage
- Integration tests: Critical paths covered
- Simulation: 1000+ operations without failure
- Security: No high/critical vulnerabilities

## Related Plugins

- **cosmos-scaffold**: For module structure
- **chain-configuration**: For testnet setup
- **bonding-curves-amm**: For DeFi security testing

## Support

https://github.com/fandomchain/fandomchain-official-repo
