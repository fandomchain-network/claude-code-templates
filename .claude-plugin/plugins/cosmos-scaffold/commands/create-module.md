# Create Cosmos SDK Module

You are an expert Cosmos SDK developer specialized in module creation. Your task is to create a new Cosmos SDK module following best practices for Cosmos SDK v0.50+.

## Task Overview

Create a complete Cosmos SDK module with the following structure:

```
x/{module-name}/
├── keeper/
│   ├── keeper.go          # Keeper with Collections-based state management
│   ├── msg_server.go      # Message server implementation
│   ├── query_server.go    # Query server implementation
│   └── genesis.go         # Genesis import/export logic
├── types/
│   ├── keys.go            # Storage key prefixes
│   ├── codec.go           # Amino/Proto codec registration
│   ├── genesis.go         # Genesis state types
│   ├── params.go          # Module parameters
│   ├── expected_keepers.go # Interface definitions for dependencies
│   ├── errors.go          # Custom error definitions
│   └── events.go          # Event type constants
├── module/
│   ├── module.go          # Module interface implementation
│   ├── autocli.go         # AutoCLI configuration
│   └── depinject.go       # Dependency injection setup
└── proto/{project}/{module-name}/v1/
    ├── tx.proto           # Transaction message definitions
    ├── query.proto        # Query service definitions
    ├── genesis.proto      # Genesis state definitions
    └── params.proto       # Module parameters
```

## Instructions

### 1. Information Gathering

First, ask the user for:
- **Module name** (lowercase, e.g., "nftmarket", "staking")
- **Description** (brief description of module purpose)
- **Dependencies** (which other keepers needed: bank, auth, staking, etc.)
- **Initial state** (what data structures need to be stored)

### 2. Module Creation Steps

#### Step 1: Create Directory Structure
Create the directories: `x/{module-name}/keeper`, `x/{module-name}/types`, `x/{module-name}/module`

#### Step 2: Create Proto Definitions
Create protobuf files in `proto/{project}/{module-name}/v1/`:

**tx.proto:**
```protobuf
syntax = "proto3";
package {project}.{module}.v1;

option go_package = "github.com/{org}/{project}/x/{module}/types";

import "cosmos/msg/v1/msg.proto";
import "gogoproto/gogo.proto";

service Msg {
  option (cosmos.msg.v1.service) = true;

  // Add message definitions here
}

message MsgUpdateParams {
  option (cosmos.msg.v1.signer) = "authority";
  string authority = 1 [(gogoproto.moretags) = "yaml:\"authority\""];
  Params params = 2 [(gogoproto.nullable) = false];
}

message MsgUpdateParamsResponse {}
```

**query.proto:**
```protobuf
syntax = "proto3";
package {project}.{module}.v1;

option go_package = "github.com/{org}/{project}/x/{module}/types";

import "google/api/annotations.proto";
import "cosmos/base/query/v1beta1/pagination.proto";

service Query {
  // Params returns module parameters
  rpc Params(QueryParamsRequest) returns (QueryParamsResponse) {
    option (google.api.http).get = "/{project}/{module}/v1/params";
  }
}

message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1 [(gogoproto.nullable) = false];
}
```

**genesis.proto & params.proto:** Create with appropriate structure

#### Step 3: Generate Proto Code
Run: `make proto-gen` to generate Go code from proto definitions

#### Step 4: Create Types Files
Create files in `x/{module-name}/types/`:

**keys.go:**
```go
package types

import "cosmossdk.io/collections"

const (
    ModuleName = "{module-name}"
    StoreKey   = ModuleName
    RouterKey  = ModuleName
)

var (
    ParamsKey = collections.NewPrefix(0)
    // Add other collection prefixes
)
```

**codec.go, errors.go, events.go:** Create with standard patterns

#### Step 5: Create Keeper
Create `x/{module-name}/keeper/keeper.go` using Collections framework:

```go
package keeper

import (
    "cosmossdk.io/collections"
    storetypes "cosmossdk.io/store/types"
    "github.com/cosmos/cosmos-sdk/codec"
)

type Keeper struct {
    cdc          codec.BinaryCodec
    storeService storetypes.KVStoreService
    authority    string

    Schema collections.Schema
    Params collections.Item[types.Params]
    // Add other collections
}

func NewKeeper(/* params */) Keeper {
    sb := collections.NewSchemaBuilder(storeService)

    k := Keeper{
        cdc:          cdc,
        storeService: storeService,
        authority:    authority,
        Params:       collections.NewItem(sb, types.ParamsKey, "params", codec.CollValue[types.Params](cdc)),
    }

    schema, err := sb.Build()
    if err != nil {
        panic(err)
    }
    k.Schema = schema

    return k
}
```

#### Step 6: Create Module Implementation
Create `x/{module-name}/module/module.go` implementing AppModule interface

#### Step 7: Register in App
Update `app/app.go`:
- Import the module
- Add keeper to App struct
- Initialize keeper in NewApp()
- Register in module manager

Update `app/app_config.go`:
- Add module to init_genesis, begin_blockers, end_blockers

### 3. Testing
Generate basic keeper test file to verify module loads correctly

### 4. Documentation
Create `x/{module-name}/README.md` documenting:
- Module purpose
- State structure
- Messages
- Queries
- Events

## Best Practices

1. **Use Collections Framework**: For type-safe state management (Cosmos SDK v0.50+)
2. **Proper Error Handling**: Use module-specific error types
3. **Event Emission**: Emit events for all state changes
4. **Validation**: Validate all inputs in message handlers
5. **Comments**: Document all exported types and functions
6. **Genesis**: Implement proper genesis import/export
7. **Determinism**: Ensure all operations are deterministic

## Example Output

After completion, the user should be able to:
- Import and use the module in their chain
- Send transactions to the module
- Query module state
- Test the module with unit tests

## Notes

- Follow the patterns used in existing Cosmos SDK modules (bank, staking, gov)
- Use autocli for automatic CLI generation when possible
- Implement proper authorization checks (authority for params updates)
- Consider IBC integration if cross-chain functionality is needed
