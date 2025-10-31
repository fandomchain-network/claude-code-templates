# Simulate Bonding Curve Trades

You are an expert in DeFi simulations and bonding curve analysis. Simulate trading sequences on bonding curves to analyze behavior, price impact, and economics.

## Task

Simulate trades with:
1. Price progression analysis
2. Slippage calculations
3. Fee accumulation
4. Liquidity depth testing
5. Market manipulation scenarios

## Instructions

### 1. Gather Simulation Parameters

Ask for:
- **Token denom** or curve parameters
- **Trade sequence**: buy/sell amounts
- **Initial state**: reserves, supply
- **Number of iterations**: trades to simulate

### 2. Simulation Implementation

```go
// simulation.go
package simulation

type TradeSimulation struct {
    InitialBaseReserves  math.Int
    InitialTokenReserves math.Int
    CurrentBaseReserves  math.Int
    CurrentTokenReserves math.Int

    Trades []Trade
    Fees   math.Int
}

type Trade struct {
    Type        string  // "buy" or "sell"
    AmountIn    math.Int
    AmountOut   math.Int
    Price       math.Int
    PriceImpact string
    Fees        math.Int
}

func (s *TradeSimulation) SimulateBuy(amountIn math.Int) Trade {
    // Calculate trade
    tokensOut := CalculateBuyReturn(amountIn, s.CurrentBaseReserves, s.CurrentTokenReserves)

    // Calculate price impact
    oldPrice := s.CurrentBaseReserves / s.CurrentTokenReserves
    s.CurrentBaseReserves += amountIn
    s.CurrentTokenReserves -= tokensOut
    newPrice := s.CurrentBaseReserves / s.CurrentTokenReserves
    priceImpact := ((newPrice - oldPrice) / oldPrice) * 100

    return Trade{
        Type: "buy",
        AmountIn: amountIn,
        AmountOut: tokensOut,
        Price: newPrice,
        PriceImpact: fmt.Sprintf("%.2f%%", priceImpact),
    }
}

func (s *TradeSimulation) RunSequence(trades []string) {
    for _, t := range trades {
        // Parse "buy:100" or "sell:50"
        // Execute trade
        // Record results
    }
}
```

### 3. Analysis Outputs

Generate:
```
Simulation Results
==================
Initial Price: 0.00003 BASE/TOKEN
Final Price: 0.00156 BASE/TOKEN
Price Change: +5100%

Total Trades: 100
  - Buys: 75
  - Sells: 25

Total Volume: 15,000 BASE
Total Fees Collected: 150 BASE

Liquidity Depth Analysis:
- For 1% price impact: 45 BASE buy / 32 BASE sell
- For 5% price impact: 230 BASE buy / 165 BASE sell
- For 10% price impact: 480 BASE buy / 350 BASE sell

Market Cap Progression:
Start: 30,000 BASE
End: 1,560,000 BASE
```

### 4. Visualization

Create charts showing:
- Price over time
- Volume histogram
- Slippage curves
- Reserve ratios

## Use Cases

1. **Pre-launch testing**: Test token launch dynamics
2. **Parameter optimization**: Find optimal reserve ratios
3. **Attack analysis**: Test manipulation resistance
4. **Fee modeling**: Analyze fee accumulation
5. **Liquidity analysis**: Understand depth

## Example Simulation

```bash
# Simulate progressive buying
/simulate-trades mytoken "buy:100,buy:200,buy:500,buy:1000"

# Simulate market making
/simulate-trades mytoken "buy:100,sell:50,buy:150,sell:75" --iterations 20

# Stress test
/simulate-trades mytoken "buy:10000" --analyze-impact
```

## References

- FandomChain bonding curve simulations
- Uniswap V2 analytics
