# Run Cosmos SDK Simulations

You are an expert in Cosmos SDK simulation testing. Run and analyze simulation tests for chaos testing and non-determinism detection.

## Task

Execute simulation testing with:
1. Random operation sequences
2. Invariant checking
3. Non-determinism detection
4. Import/export validation
5. State machine testing

## Instructions

### 1. Setup Simulations

```go
// simulation.go
package simulation

import (
    "math/rand"
    simtypes "github.com/cosmos/cosmos-sdk/types/simulation"
)

// Weighted operations
func WeightedOperations(
    appParams simtypes.AppParams,
    cdc codec.JSONCodec,
    ak types.AccountKeeper,
    bk types.BankKeeper,
    k keeper.Keeper,
) simulation.WeightedOperations {
    return simulation.WeightedOperations{
        simulation.NewWeightedOperation(
            weightMsgCreateDenom,
            SimulateMsgCreateDenom(ak, bk, k),
        ),
        simulation.NewWeightedOperation(
            weightMsgBuy,
            SimulateMsgBuy(ak, bk, k),
        ),
    }
}

// Simulate message
func SimulateMsgCreateDenom(ak, bk, k) simtypes.Operation {
    return func(
        r *rand.Rand,
        app *baseapp.BaseApp,
        ctx sdk.Context,
        accs []simtypes.Account,
        chainID string,
    ) (simtypes.OperationMsg, []simtypes.FutureOperation, error) {
        // Select random account
        simAccount, _ := simtypes.RandomAcc(r, accs)

        // Generate random message
        msg := &types.MsgCreateDenom{
            Creator:  simAccount.Address.String(),
            Subdenom: simtypes.RandStringOfLength(r, 10),
        }

        // Execute
        txCtx := simulation.OperationInput{...}
        return simulation.GenAndDeliverTxWithRandFees(txCtx)
    }
}
```

### 2. Run Simulations

```bash
# Full simulation (100 blocks)
go test -mod=readonly ./app -run TestFullAppSimulation \
  -Enabled=true \
  -NumBlocks=100 \
  -BlockSize=200 \
  -Commit=true \
  -Seed=42 \
  -v -timeout 24h

# Import/Export
go test -mod=readonly ./app -run TestAppImportExport \
  -Enabled=true \
  -NumBlocks=50 \
  -v

# Non-determinism
go test -mod=readonly ./app -run TestAppSimulationAfterImport \
  -Enabled=true \
  -NumBlocks=50 \
  -v
```

### 3. Invariants

```go
// invariants.go
func RegisterInvariants(ir sdk.InvariantRegistry, k keeper.Keeper, bk types.BankKeeper) {
    ir.RegisterRoute(types.ModuleName, "supply",
        SupplyInvariant(k, bk))
    ir.RegisterRoute(types.ModuleName, "reserves",
        ReservesInvariant(k, bk))
}

func SupplyInvariant(k keeper.Keeper, bk types.BankKeeper) sdk.Invariant {
    return func(ctx sdk.Context) (string, bool) {
        // Check total supply matches expected
        var broken bool
        var msg string

        // Validate supply
        // ...

        return sdk.FormatInvariant(
            types.ModuleName, "supply",
            msg,
        ), broken
    }
}
```

## Analysis

Analyze results:
- Operations executed
- Errors encountered
- Invariant violations
- Non-determinism detected
- State consistency

## Resources

- Cosmos SDK simulation docs
- App simulation examples
