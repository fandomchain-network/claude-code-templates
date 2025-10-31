# Create Bonding Curve Implementation

You are an expert in DeFi and bonding curve mathematics. Create a bonding curve implementation for a Cosmos SDK module.

## Task

Implement bonding curve logic including:
1. Mathematical formula (constant product, linear, exponential, or custom)
2. Buy/sell functions with proper calculations
3. Fee mechanisms (trading fees, protocol fees)
4. Slippage protection
5. Price calculation and estimation
6. Reserve management

## Instructions

### 1. Gather Requirements

Ask user for:
- **Curve type**: constant-product, linear, exponential, sigmoid, custom
- **Initial reserves**: Virtual reserves for base and token
- **Fee structure**: Buy fee %, sell fee %, fee distribution
- **Safety limits**: Max slippage, max price impact
- **Token economics**: Initial supply, max supply, decimals

### 2. Implement Core Formulas

#### Constant Product (x * y = k)
```go
// bonding_curve.go
package keeper

import (
    "cosmossdk.io/math"
)

const (
    TOKEN_DECIMALS = 1_000_000  // 6 decimals
    BASE_DECIMALS  = 1_000_000_000  // 9 decimals

    INITIAL_VIRTUAL_BASE_RESERVES  = math.NewInt(30).Mul(BASE_DECIMALS)
    INITIAL_VIRTUAL_TOKEN_RESERVES = math.NewInt(1_000_000).Mul(TOKEN_DECIMALS)

    BUY_FEE_BPS  = 100  // 1%
    SELL_FEE_BPS = 100  // 1%
)

// CalculateBuyReturn calculates tokens received for base input
func (k Keeper) CalculateBuyReturn(
    baseIn math.Int,
    baseReserves math.Int,
    tokenReserves math.Int,
) (tokensOut math.Int, err error) {
    // Apply fee
    feeAmount := baseIn.Mul(math.NewInt(BUY_FEE_BPS)).Quo(math.NewInt(10000))
    baseInAfterFee := baseIn.Sub(feeAmount)

    // Constant product: k = x * y
    k := baseReserves.Mul(tokenReserves)

    // New base reserves after buy
    newBaseReserves := baseReserves.Add(baseInAfterFee)

    // New token reserves (k / new_base)
    newTokenReserves := k.Quo(newBaseReserves)

    // Tokens out
    tokensOut = tokenReserves.Sub(newTokenReserves)

    return tokensOut, nil
}

// CalculateSellReturn calculates base received for token input
func (k Keeper) CalculateSellReturn(
    tokensIn math.Int,
    baseReserves math.Int,
    tokenReserves math.Int,
) (baseOut math.Int, err error) {
    // Constant product
    k := baseReserves.Mul(tokenReserves)

    // New token reserves
    newTokenReserves := tokenReserves.Add(tokensIn)

    // New base reserves
    newBaseReserves := k.Quo(newTokenReserves)

    // Base out before fee
    baseOutBeforeFee := baseReserves.Sub(newBaseReserves)

    // Apply fee
    feeAmount := baseOutBeforeFee.Mul(math.NewInt(SELL_FEE_BPS)).Quo(math.NewInt(10000))
    baseOut = baseOutBeforeFee.Sub(feeAmount)

    return baseOut, nil
}

// GetCurrentPrice calculates current spot price
func (k Keeper) GetCurrentPrice(
    baseReserves math.Int,
    tokenReserves math.Int,
) math.Int {
    // Price = base_reserves / token_reserves
    return baseReserves.Mul(TOKEN_DECIMALS).Quo(tokenReserves)
}
```

#### Linear Bonding Curve
```go
// Price = m * supply + b
func (k Keeper) CalculateLinearPrice(supply math.Int, m, b math.Int) math.Int {
    return m.Mul(supply).Add(b)
}

// Buy cost = integral from supply to supply+amount
func (k Keeper) CalculateLinearBuyCost(
    currentSupply math.Int,
    amount math.Int,
    m, b math.Int,
) math.Int {
    // Integral: (m/2)*x^2 + b*x
    s1 := currentSupply
    s2 := currentSupply.Add(amount)

    cost1 := m.Mul(s1).Mul(s1).Quo(math.NewInt(2)).Add(b.Mul(s1))
    cost2 := m.Mul(s2).Mul(s2).Quo(math.NewInt(2)).Add(b.Mul(s2))

    return cost2.Sub(cost1)
}
```

### 3. Implement Message Handlers

```go
// msg_server_trade.go
func (k msgServer) Buy(goCtx context.Context, msg *types.MsgBuy) (*types.MsgBuyResponse, error) {
    ctx := sdk.UnwrapSDKContext(goCtx)

    // Get current reserves
    denom, err := k.Denom.Get(ctx, msg.Denom)
    if err != nil {
        return nil, err
    }

    // Calculate tokens out
    tokensOut, err := k.CalculateBuyReturn(
        msg.AmountIn,
        denom.BaseReserves,
        denom.TokenReserves,
    )
    if err != nil {
        return nil, err
    }

    // Check slippage protection
    if tokensOut.LT(msg.MinTokensOut) {
        return nil, types.ErrSlippageExceeded
    }

    // Transfer base from buyer
    err = k.bankKeeper.SendCoinsFromAccountToModule(
        ctx,
        buyer,
        types.ModuleName,
        sdk.NewCoins(sdk.NewCoin(BaseDenom, msg.AmountIn)),
    )
    if err != nil {
        return nil, err
    }

    // Transfer tokens to buyer
    err = k.bankKeeper.SendCoinsFromModuleToAccount(
        ctx,
        types.ModuleName,
        buyer,
        sdk.NewCoins(sdk.NewCoin(msg.Denom, tokensOut)),
    )
    if err != nil {
        return nil, err
    }

    // Update reserves
    denom.BaseReserves = denom.BaseReserves.Add(msg.AmountIn)
    denom.TokenReserves = denom.TokenReserves.Sub(tokensOut)
    k.Denom.Set(ctx, msg.Denom, denom)

    // Emit event
    ctx.EventManager().EmitEvent(...)

    return &types.MsgBuyResponse{TokensOut: tokensOut}, nil
}
```

### 4. Add Query Endpoints

```go
// query_price.go
func (k Keeper) Price(goCtx context.Context, req *types.QueryPriceRequest) (*types.QueryPriceResponse, error) {
    ctx := sdk.UnwrapSDKContext(goCtx)

    denom, err := k.Denom.Get(ctx, req.Denom)
    if err != nil {
        return nil, err
    }

    currentPrice := k.GetCurrentPrice(denom.BaseReserves, denom.TokenReserves)

    return &types.QueryPriceResponse{
        Price: currentPrice.String(),
        BaseReserves: denom.BaseReserves.String(),
        TokenReserves: denom.TokenReserves.String(),
    }, nil
}

// Estimate buy/sell
func (k Keeper) EstimateBuy(goCtx context.Context, req *types.QueryEstimateBuyRequest) (*types.QueryEstimateBuyResponse, error) {
    // Calculate without executing
    tokensOut, _ := k.CalculateBuyReturn(...)
    priceImpact := calculatePriceImpact(...)

    return &types.QueryEstimateBuyResponse{
        TokensOut: tokensOut.String(),
        PriceImpact: priceImpact.String(),
    }, nil
}
```

### 5. Security Measures

```go
// Validate slippage
const MAX_SLIPPAGE_PERCENT = 5  // 5%
if actualPrice.Sub(expectedPrice).Abs().Mul(100).Quo(expectedPrice).GT(math.NewInt(MAX_SLIPPAGE_PERCENT)) {
    return ErrSlippageExceeded
}

// Validate price impact
const MAX_PRICE_IMPACT_PERCENT = 10  // 10%
priceImpact := newPrice.Sub(oldPrice).Abs().Mul(100).Quo(oldPrice)
if priceImpact.GT(math.NewInt(MAX_PRICE_IMPACT_PERCENT)) {
    return ErrPriceImpactTooHigh
}

// Check for manipulation
if tokensOut.IsZero() || tokensOut.IsNegative() {
    return ErrInvalidOutput
}
```

## Best Practices

1. **Use math.Int**: Never use float64 for financial calculations
2. **Precision**: Maintain precision throughout calculations
3. **Fees First**: Apply fees before calculating outputs
4. **Slippage Protection**: Always enforce min output amounts
5. **Event Emission**: Emit events for indexing and transparency
6. **Atomic Updates**: Update all reserves atomically
7. **Testing**: Extensive tests for edge cases

## Testing

Write comprehensive tests:
- Basic buy/sell
- Fee calculation
- Slippage protection
- Price impact
- Large trades
- Reserve depletion scenarios
- Arithmetic edge cases

## Resources

- FandomChain TokenFactory: Reference implementation
- Uniswap V2: Constant product AMM
- Bancor: Original bonding curve implementation
