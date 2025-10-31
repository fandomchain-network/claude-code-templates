# Add Query to Cosmos SDK Module

You are an expert Cosmos SDK developer. Your task is to add a new query endpoint to an existing Cosmos SDK module, including the protobuf definition, query handler, and CLI integration.

## Task Overview

Add a complete query implementation with:
1. Protobuf query definition in `query.proto`
2. Query handler in keeper
3. Response type definition
4. CLI query command (via autocli)
5. Pagination support (if needed)

## Instructions

### 1. Information Gathering

Ask the user for:
- **Module name** (e.g., "tokenfactory", "staking")
- **Query name** (e.g., "BondingCurvePrice", "TokenInfo")
- **Query parameters** (what inputs does the query need?)
- **Response data** (what information should be returned?)
- **Pagination needed?** (for queries returning lists)

### 2. Implementation Steps

#### Step 1: Update query.proto

Add query definition to `proto/{project}/{module}/v1/query.proto`:

```protobuf
// Example: Query token information by denom
message QueryTokenInfoRequest {
  string denom = 1;
}

message QueryTokenInfoResponse {
  string denom = 1;
  string name = 2;
  string symbol = 3;
  uint64 total_supply = 4;
  string creator = 5;
  bool enabled = 6;
}

// For paginated queries (list of items)
message QueryListTokensRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QueryListTokensResponse {
  repeated TokenInfo tokens = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}

message TokenInfo {
  string denom = 1;
  string name = 2;
  string symbol = 3;
}
```

Add to Query service:
```protobuf
service Query {
  // ... existing queries ...

  // TokenInfo returns information about a specific token
  rpc TokenInfo(QueryTokenInfoRequest) returns (QueryTokenInfoResponse) {
    option (google.api.http).get = "/{project}/{module}/v1/token/{denom}";
  }

  // ListTokens returns all tokens with pagination
  rpc ListTokens(QueryListTokensRequest) returns (QueryListTokensResponse) {
    option (google.api.http).get = "/{project}/{module}/v1/tokens";
  }
}
```

**Key points:**
- Use descriptive request/response message names
- Include REST endpoint annotation with `google.api.http`
- Use pagination for list queries
- Return structured data (use nested messages if complex)

#### Step 2: Regenerate Proto Code

Run: `make proto-gen`

#### Step 3: Implement Query Handler

##### For Simple Queries (single item):

In `x/{module}/keeper/query_{query_name}.go`:

```go
package keeper

import (
    "context"

    sdk "github.com/cosmos/cosmos-sdk/types"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"

    "github.com/{org}/{project}/x/{module}/types"
)

func (k Keeper) TokenInfo(goCtx context.Context, req *types.QueryTokenInfoRequest) (*types.QueryTokenInfoResponse, error) {
    if req == nil {
        return nil, status.Error(codes.InvalidArgument, "invalid request")
    }

    ctx := sdk.UnwrapSDKContext(goCtx)

    // Validate input
    if req.Denom == "" {
        return nil, status.Error(codes.InvalidArgument, "denom cannot be empty")
    }

    // Fetch data from state using Collections
    token, err := k.Tokens.Get(ctx, req.Denom)
    if err != nil {
        return nil, status.Error(codes.NotFound, "token not found")
    }

    // Build response
    return &types.QueryTokenInfoResponse{
        Denom:       token.Denom,
        Name:        token.Name,
        Symbol:      token.Symbol,
        TotalSupply: token.TotalSupply,
        Creator:     token.Creator,
        Enabled:     token.Enabled,
    }, nil
}
```

##### For Paginated Queries (lists):

```go
import (
    "cosmossdk.io/collections"
    "github.com/cosmos/cosmos-sdk/types/query"
)

func (k Keeper) ListTokens(goCtx context.Context, req *types.QueryListTokensRequest) (*types.QueryListTokensResponse, error) {
    if req == nil {
        return nil, status.Error(codes.InvalidArgument, "invalid request")
    }

    ctx := sdk.UnwrapSDKContext(goCtx)

    var tokens []types.TokenInfo

    // Iterate through collection with pagination
    _, pageRes, err := query.CollectionPaginate(
        ctx,
        k.Tokens,
        req.Pagination,
        func(key string, value types.Token) (types.TokenInfo, error) {
            return types.TokenInfo{
                Denom:  value.Denom,
                Name:   value.Name,
                Symbol: value.Symbol,
            }, nil
        },
    )
    if err != nil {
        return nil, status.Error(codes.Internal, err.Error())
    }

    return &types.QueryListTokensResponse{
        Tokens:     tokens,
        Pagination: pageRes,
    }, nil
}
```

##### For Computed Queries (calculations):

```go
func (k Keeper) BondingCurvePrice(goCtx context.Context, req *types.QueryBondingCurvePriceRequest) (*types.QueryBondingCurvePriceResponse, error) {
    if req == nil {
        return nil, status.Error(codes.InvalidArgument, "invalid request")
    }

    ctx := sdk.UnwrapSDKContext(goCtx)

    // Fetch state
    denom, err := k.Denom.Get(ctx, req.Denom)
    if err != nil {
        return nil, status.Error(codes.NotFound, "denom not found")
    }

    // Perform calculations
    currentPrice := k.CalculateCurrentPrice(ctx, denom)
    marketCap := k.CalculateMarketCap(ctx, denom)

    return &types.QueryBondingCurvePriceResponse{
        CurrentPrice:       currentPrice.String(),
        MarketCap:          marketCap.String(),
        VirtualReservesFandom: denom.VirtualFandomReserves.String(),
        VirtualReservesToken:  denom.VirtualTokenReserves.String(),
    }, nil
}
```

#### Step 4: Register Query Server (if new file)

Ensure `x/{module}/keeper/keeper.go` implements QueryServer:

```go
var _ types.QueryServer = Keeper{}
```

#### Step 5: Write Tests

Create `x/{module}/keeper/query_{query_name}_test.go`:

```go
package keeper_test

import (
    "testing"

    "github.com/stretchr/testify/require"
    "github.com/{org}/{project}/x/{module}/types"
)

func TestQueryTokenInfo(t *testing.T) {
    keeper, ctx := setupKeeper(t)

    // Setup test data
    expectedToken := types.Token{
        Denom:  "testtoken",
        Name:   "Test Token",
        Symbol: "TEST",
    }
    keeper.Tokens.Set(ctx, expectedToken.Denom, expectedToken)

    // Execute query
    resp, err := keeper.TokenInfo(ctx, &types.QueryTokenInfoRequest{
        Denom: "testtoken",
    })

    // Verify
    require.NoError(t, err)
    require.NotNil(t, resp)
    require.Equal(t, expectedToken.Denom, resp.Denom)
    require.Equal(t, expectedToken.Name, resp.Name)
}

func TestQueryTokenInfo_NotFound(t *testing.T) {
    keeper, ctx := setupKeeper(t)

    // Query non-existent token
    resp, err := keeper.TokenInfo(ctx, &types.QueryTokenInfoRequest{
        Denom: "nonexistent",
    })

    // Verify error
    require.Error(t, err)
    require.Nil(t, resp)
    require.Contains(t, err.Error(), "not found")
}
```

### 3. CLI Command (AutoCLI)

If using autocli (recommended), queries are auto-generated. The query will be available as:

```bash
{app}d query {module} token-info [denom]
{app}d query {module} list-tokens --page-limit 50
```

### 4. REST/gRPC Gateway

The `google.api.http` annotation creates REST endpoints automatically:

```
GET /{project}/{module}/v1/token/{denom}
GET /{project}/{module}/v1/tokens?pagination.limit=50
```

### 5. Verification Steps

1. **Build**: `make install`
2. **Test**: `make test-unit`
3. **CLI Query**: Test with binary CLI
4. **REST Query**: Test REST endpoint
5. **gRPC Query**: Test with grpcurl (optional)

## Query Types & Patterns

### 1. Simple Get Query
Retrieve a single item by key:
```go
item, err := k.Collection.Get(ctx, key)
```

### 2. List Query with Pagination
Retrieve multiple items:
```go
query.CollectionPaginate(ctx, k.Collection, req.Pagination, transformFunc)
```

### 3. Filtered Query
Return items matching criteria:
```go
query.CollectionFilteredPaginate(ctx, k.Collection, req.Pagination, filterFunc, transformFunc)
```

### 4. Computed Query
Calculate values from stored data:
```go
// Fetch state, compute, return results
```

### 5. Aggregate Query
Sum/count across collection:
```go
// Iterate collection, accumulate values
```

## Pagination Best Practices

```go
import (
    "github.com/cosmos/cosmos-sdk/types/query"
    "cosmossdk.io/collections"
)

// Always support pagination for list queries
_, pageRes, err := query.CollectionPaginate(
    ctx,
    k.Collection,
    req.Pagination,
    func(key string, value types.Item) (types.ItemResponse, error) {
        // Transform stored type to response type
        return types.ItemResponse{
            Id:   value.Id,
            Name: value.Name,
        }, nil
    },
)
```

## Response Design Guidelines

1. **Be Explicit**: Return all relevant data, don't make clients query multiple times
2. **Use Strings for Large Numbers**: Return `math.Int` as strings to avoid JSON precision loss
3. **Include Metadata**: Add status, timestamps, computed values
4. **Nested Messages**: Use nested messages for complex structures
5. **Consistent Formatting**: Follow project conventions

Example comprehensive response:
```protobuf
message QueryTokenDetailsResponse {
  // Basic info
  string denom = 1;
  string name = 2;
  string symbol = 3;

  // Supply info
  string total_supply = 4;
  string circulating_supply = 5;

  // Bonding curve info (if applicable)
  BondingCurveInfo bonding_curve = 6;

  // Metadata
  string creator = 7;
  google.protobuf.Timestamp created_at = 8;
  bool enabled = 9;
}

message BondingCurveInfo {
  string current_price = 1;
  string market_cap = 2;
  string virtual_reserves_fandom = 3;
  string virtual_reserves_token = 4;
}
```

## Error Handling

Use appropriate gRPC error codes:
```go
import (
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

// Input validation errors
status.Error(codes.InvalidArgument, "invalid parameter")

// Not found errors
status.Error(codes.NotFound, "resource not found")

// Internal errors (unexpected)
status.Error(codes.Internal, "internal error")

// Permission errors
status.Error(codes.PermissionDenied, "access denied")
```

## Performance Considerations

1. **Limit Results**: Always use pagination for lists
2. **Index Optimization**: Use appropriate collection keys
3. **Avoid Heavy Computation**: For expensive calculations, consider caching
4. **Read-Only**: Queries should never modify state
5. **Deterministic**: Queries must return same results for same input

## Reference Examples

See FandomChain queries:
- `x/tokenfactory/keeper/query_bonding_curve.go` - Computed queries with calculations
- Price queries, progress tracking, estimation

These demonstrate:
- Complex calculations
- Multiple data sources
- Comprehensive responses
- Error handling

## Testing Checklist

- [ ] Happy path test
- [ ] Error case tests (not found, invalid input)
- [ ] Pagination tests (first page, last page, empty)
- [ ] Edge cases (boundary values)
- [ ] Performance tests (large datasets)
