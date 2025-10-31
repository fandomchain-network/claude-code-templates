# Add Message to Cosmos SDK Module

You are an expert Cosmos SDK developer. Your task is to add a new message type to an existing Cosmos SDK module, including the protobuf definition, message handler, validation logic, and event emission.

## Task Overview

Add a complete message implementation to an existing module with:
1. Protobuf message definition in `tx.proto`
2. Message handler in keeper
3. Validation logic
4. Event emission
5. CLI command (via autocli)
6. Error handling

## Instructions

### 1. Information Gathering

Ask the user for:
- **Module name** (e.g., "tokenfactory", "nftmarket")
- **Message name** (e.g., "CreateDenom", "MintNFT")
- **Message fields** with types:
  - Field name and type (string, uint64, int64, bool, sdk.Coin, etc.)
  - Which field is the signer
  - Which fields are optional
- **Business logic** (what should the message do?)
- **Required permissions** (who can execute: anyone, owner, admin?)

### 2. Implementation Steps

#### Step 1: Update tx.proto

Add the message definition to `proto/{project}/{module}/v1/tx.proto`:

```protobuf
// Example: Adding CreateToken message
message MsgCreateToken {
  option (cosmos.msg.v1.signer) = "creator";

  string creator = 1;
  string name = 2;
  string symbol = 3;
  uint64 initial_supply = 4;
  string description = 5;
}

message MsgCreateTokenResponse {
  string denom = 1;  // Return the created token denom
}
```

Add to the Msg service:
```protobuf
service Msg {
  // ... existing messages ...

  rpc CreateToken(MsgCreateToken) returns (MsgCreateTokenResponse);
}
```

**Key points:**
- Use `option (cosmos.msg.v1.signer)` to specify who signs
- Response messages should include relevant return data
- Use appropriate protobuf field numbers (sequential, don't reuse)
- Import necessary types (cosmos_proto, gogoproto, etc.)

#### Step 2: Regenerate Proto Code

Run: `make proto-gen`

This generates:
- Go types in `x/{module}/types/tx.pb.go`
- gRPC service definitions

#### Step 3: Create Message Constructor and Validation

In `x/{module}/types/messages_{message}.go`:

```go
package types

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "cosmossdk.io/errors"
)

var _ sdk.Msg = &MsgCreateToken{}

func NewMsgCreateToken(creator, name, symbol string, initialSupply uint64, description string) *MsgCreateToken {
    return &MsgCreateToken{
        Creator:       creator,
        Name:          name,
        Symbol:        symbol,
        InitialSupply: initialSupply,
        Description:   description,
    }
}

// ValidateBasic performs stateless validation
func (msg *MsgCreateToken) ValidateBasic() error {
    _, err := sdk.AccAddressFromBech32(msg.Creator)
    if err != nil {
        return sdkerrors.Wrapf(sdkerrors.ErrInvalidAddress, "invalid creator address (%s)", err)
    }

    if msg.Name == "" {
        return sdkerrors.Wrap(ErrInvalidInput, "name cannot be empty")
    }

    if msg.Symbol == "" {
        return sdkerrors.Wrap(ErrInvalidInput, "symbol cannot be empty")
    }

    if len(msg.Symbol) > 10 {
        return sdkerrors.Wrap(ErrInvalidInput, "symbol too long (max 10 chars)")
    }

    if msg.InitialSupply == 0 {
        return sdkerrors.Wrap(ErrInvalidInput, "initial supply must be greater than 0")
    }

    return nil
}
```

#### Step 4: Implement Message Handler

In `x/{module}/keeper/msg_server_{message}.go`:

```go
package keeper

import (
    "context"
    "fmt"

    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/{org}/{project}/x/{module}/types"
)

func (k msgServer) CreateToken(goCtx context.Context, msg *types.MsgCreateToken) (*types.MsgCreateTokenResponse, error) {
    ctx := sdk.UnwrapSDKContext(goCtx)

    // 1. Validation (business logic)
    // Check if token already exists, verify permissions, etc.

    // 2. State changes
    // Use keeper methods to modify state via Collections

    // 3. Emit events
    ctx.EventManager().EmitEvents(sdk.Events{
        sdk.NewEvent(
            types.EventTypeCreateToken,
            sdk.NewAttribute(types.AttributeKeyCreator, msg.Creator),
            sdk.NewAttribute(types.AttributeKeyTokenName, msg.Name),
            sdk.NewAttribute(types.AttributeKeySymbol, msg.Symbol),
            sdk.NewAttribute(types.AttributeKeyInitialSupply, fmt.Sprintf("%d", msg.InitialSupply)),
        ),
    })

    // 4. Return response
    return &types.MsgCreateTokenResponse{
        Denom: generatedDenom,
    }, nil
}
```

**Handler Best Practices:**
- Validate business logic (not just basic validation)
- Use keeper collections for state access
- Emit detailed events for indexing
- Return meaningful response data
- Handle errors with specific error types
- Ensure deterministic execution (no randomness, external calls)

#### Step 5: Update Event Types

In `x/{module}/types/events.go`:

```go
const (
    EventTypeCreateToken = "create_token"

    AttributeKeyCreator       = "creator"
    AttributeKeyTokenName     = "token_name"
    AttributeKeySymbol        = "symbol"
    AttributeKeyInitialSupply = "initial_supply"
)
```

#### Step 6: Update Error Types (if needed)

In `x/{module}/types/errors.go`:

```go
var (
    ErrTokenAlreadyExists = sdkerrors.Register(ModuleName, 2, "token already exists")
    ErrInvalidInput       = sdkerrors.Register(ModuleName, 3, "invalid input")
)
```

#### Step 7: Verify msg_server.go Interface

Ensure `x/{module}/keeper/msg_server.go` has the msgServer struct:

```go
package keeper

import "github.com/{org}/{project}/x/{module}/types"

type msgServer struct {
    Keeper
}

var _ types.MsgServer = msgServer{}

func NewMsgServerImpl(keeper Keeper) types.MsgServer {
    return &msgServer{Keeper: keeper}
}
```

#### Step 8: Write Tests

Create `x/{module}/keeper/msg_server_{message}_test.go`:

```go
package keeper_test

import (
    "testing"
    "github.com/stretchr/testify/require"
)

func TestMsgServerCreateToken(t *testing.T) {
    // Setup test keeper
    // Create message
    // Execute handler
    // Verify state changes
    // Verify events
}
```

### 3. CLI Command (AutoCLI)

If using autocli (recommended), the CLI command is auto-generated. Otherwise, manually add to `x/{module}/client/cli/tx.go`.

### 4. Verification Steps

After implementation:

1. **Build**: Run `make install` to ensure compilation
2. **Test**: Run `make test-unit` to verify tests pass
3. **Integration**: Test with `{app}d tx {module} create-token ...`
4. **Query**: Verify state changes with queries
5. **Events**: Check event emission in transaction results

## Message Types Reference

Common field types:
- `string` - Text (addresses use bech32 format)
- `uint64` - Unsigned integer (amounts, IDs)
- `int64` - Signed integer (timestamps)
- `bool` - Boolean flag
- `bytes` - Binary data
- `repeated` - Arrays/slices
- Custom types from other modules

Cosmos SDK types:
- `cosmos.base.v1beta1.Coin` - Token amounts
- `google.protobuf.Timestamp` - Timestamps
- `google.protobuf.Duration` - Durations

## Security Considerations

1. **Authorization**: Verify sender has permission
2. **Input Validation**: Validate all inputs thoroughly
3. **State Consistency**: Ensure atomic state updates
4. **Reentrancy**: Cosmos SDK handles this, but be aware
5. **Integer Overflow**: Use sdk.Int or math.Int for large numbers
6. **Determinism**: No external calls, randomness, or time dependencies (except block time)

## Best Practices

- Keep handlers focused (single responsibility)
- Use helper functions in keeper for complex logic
- Emit events for all significant state changes
- Return meaningful data in response messages
- Write comprehensive tests (happy path + error cases)
- Document the message purpose and parameters
- Follow Cosmos SDK naming conventions

## Example Complete Implementation

See FandomChain's `x/tokenfactory/keeper/msg_server_denom.go` for reference implementations of:
- `CreateDenom`
- `BuyWithBondingCurve`
- `SellWithBondingCurve`

These demonstrate proper validation, state management, event emission, and error handling.
