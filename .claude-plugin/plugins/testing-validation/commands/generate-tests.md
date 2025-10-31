# Generate Tests for Cosmos SDK Module

You are an expert in Go testing and Cosmos SDK testing patterns. Generate comprehensive test suites for Cosmos SDK modules.

## Task

Generate tests for:
1. Keeper functions (state management)
2. Message handlers (transactions)
3. Query handlers
4. Genesis import/export
5. Simulation operations

## Instructions

### 1. Keeper Tests

```go
// keeper_test.go
package keeper_test

import (
    "testing"
    "github.com/stretchr/testify/require"
    "github.com/stretchr/testify/suite"
)

type KeeperTestSuite struct {
    suite.Suite
    keeper Keeper
    ctx    sdk.Context
}

func (suite *KeeperTestSuite) SetupTest() {
    // Setup test keeper and context
    suite.keeper, suite.ctx = setupKeeper(suite.T())
}

func TestKeeperTestSuite(t *testing.T) {
    suite.Run(t, new(KeeperTestSuite))
}

func (suite *KeeperTestSuite) TestCreateDenom() {
    // Happy path
    denom := types.Denom{...}
    err := suite.keeper.Denom.Set(suite.ctx, denom.Denom, denom)
    suite.Require().NoError(err)

    // Verify stored
    stored, err := suite.keeper.Denom.Get(suite.ctx, denom.Denom)
    suite.Require().NoError(err)
    suite.Require().Equal(denom, stored)
}
```

### 2. Message Handler Tests

```go
// msg_server_test.go
func TestMsgServerCreateDenom(t *testing.T) {
    keeper, ctx := setupKeeper(t)
    msgServer := keeper.NewMsgServerImpl(keeper)

    msg := &types.MsgCreateDenom{
        Creator:  "cosmos1...",
        Subdenom: "test",
    }

    resp, err := msgServer.CreateDenom(ctx, msg)
    require.NoError(t, err)
    require.NotNil(t, resp)
    require.NotEmpty(t, resp.Denom)

    // Verify state
    stored, err := keeper.Denom.Get(ctx, resp.Denom)
    require.NoError(t, err)
    require.Equal(t, msg.Subdenom, stored.Subdenom)
}

func TestMsgServerCreateDenom_Duplicate(t *testing.T) {
    // Test error case: duplicate denom
    keeper, ctx := setupKeeper(t)
    msgServer := keeper.NewMsgServerImpl(keeper)

    msg := &types.MsgCreateDenom{...}

    // First call succeeds
    _, err := msgServer.CreateDenom(ctx, msg)
    require.NoError(t, err)

    // Second call fails
    _, err = msgServer.CreateDenom(ctx, msg)
    require.Error(t, err)
    require.Contains(t, err.Error(), "already exists")
}
```

### 3. Query Tests

```go
// query_test.go
func TestQueryDenomInfo(t *testing.T) {
    keeper, ctx := setupKeeper(t)

    // Setup test data
    denom := types.Denom{...}
    keeper.Denom.Set(ctx, denom.Denom, denom)

    // Query
    resp, err := keeper.DenomInfo(ctx, &types.QueryDenomInfoRequest{
        Denom: denom.Denom,
    })

    require.NoError(t, err)
    require.Equal(t, denom.Denom, resp.Denom)
}
```

### 4. Test Helpers

```go
// test_helpers.go
func setupKeeper(t testing.TB) (Keeper, sdk.Context) {
    storeKey := storetypes.NewKVStoreKey(types.StoreKey)
    ctx := testutil.DefaultContext(storeKey, storetypes.NewTransientStoreKey("transient"))

    keeper := NewKeeper(
        codec.NewProtoCodec(interfaceRegistry),
        runtime.NewKVStoreService(storeKey),
        authtypes.NewModuleAddress(govtypes.ModuleName).String(),
        bankKeeper,
        authKeeper,
    )

    return keeper, ctx
}
```

## Test Coverage

Generate tests for:
- ✅ Happy path
- ✅ Error cases
- ✅ Edge cases
- ✅ Input validation
- ✅ State changes
- ✅ Event emission
- ✅ Authorization

## Best Practices

1. Use testify/require for assertions
2. Use table-driven tests for variations
3. Mock external dependencies
4. Test one thing per test
5. Clear test names describing what's tested
6. Setup/teardown in appropriate hooks

## Resources

- Cosmos SDK testing guide
- Go testing best practices
