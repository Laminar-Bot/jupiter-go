# jupiter-go

[![Go Reference](https://pkg.go.dev/badge/github.com/Laminar-Bot/jupiter-go.svg)](https://pkg.go.dev/github.com/Laminar-Bot/jupiter-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/Laminar-Bot/jupiter-go)](https://goreportcard.com/report/github.com/Laminar-Bot/jupiter-go)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Go client for the [Jupiter](https://jup.ag) Aggregator API - the leading DEX aggregator on Solana.

## Features

- 💱 **Swap Quotes** - Get best-price quotes across all Solana DEXs
- 📝 **Swap Transactions** - Build ready-to-sign swap transactions
- 💰 **Price API** - Fetch token prices in SOL or USDC
- 🎯 **Slippage Control** - Configure slippage tolerance
- ⚡ **Priority Fees** - Set priority fees for faster execution
- 🛣️ **Route Information** - Full routing details for transparency

## Installation
```bash
go get github.com/Laminar-Bot/jupiter-go
```

## Quick Start
```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/Laminar-Bot/jupiter-go"
    "github.com/shopspring/decimal"
)

func main() {
    client := jupiter.NewClient(jupiter.DefaultConfig())

    // Get a quote: Buy tokens with 0.5 SOL
    quote, err := client.GetQuote(context.Background(), &jupiter.QuoteRequest{
        InputMint:   jupiter.NativeSOL,
        OutputMint:  "DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263", // BONK
        Amount:      jupiter.SOLToLamports(decimal.NewFromFloat(0.5)),
        SlippageBps: 100, // 1%
    })
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("You'll receive: %s tokens\n", quote.OutAmount)
    fmt.Printf("Price impact: %s%%\n", quote.PriceImpactPct)
}
```

## Execute a Swap
```go
// Get quote first (see above)

// Build swap transaction
swap, err := client.GetSwapTransaction(ctx, &jupiter.SwapRequest{
    QuoteResponse:             quote,
    UserPublicKey:             "YourWalletAddress...",
    PrioritizationFeeLamports: 50000, // Optional priority fee
})
if err != nil {
    log.Fatal(err)
}

// swap.SwapTransaction contains base64-encoded transaction
// Decode, sign with your wallet, and send to RPC
```

## Price API
```go
// Get single token price
price, err := client.GetPrice(ctx, "DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263")
fmt.Printf("BONK price: $%s\n", price.Price)

// Get multiple prices
prices, err := client.GetPrices(ctx, []string{
    "DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263", // BONK
    "7GCihgDB8fe6KNjn2MYtkzZcRjQy3t9GHdC8uHYmW2hr", // POPCAT
})
```

## Helper Functions
```go
// Build a buy quote (SOL -> Token)
quote, _ := jupiter.BuildBuyQuote(tokenMint, amountSOL, slippageBps)

// Build a sell quote (Token -> SOL)  
quote, _ := jupiter.BuildSellQuote(tokenMint, amountTokens, decimals, slippageBps)

// Convert SOL to lamports
lamports := jupiter.SOLToLamports(decimal.NewFromFloat(1.5))

// Extract price from quote
price := jupiter.PriceFromQuote(quote) // Returns SOL per token
```

## Configuration
```go
client := jupiter.NewClient(jupiter.Config{
    BaseURL:         "https://quote-api.jup.ag/v6", // default
    PriceURL:        "https://price.jup.ag/v6",     // default
    TimeoutSec:      30,
    DefaultSlippage: 100,  // 1% in basis points
    DefaultPriorityFee: 50000, // lamports
})
```

## Constants
```go
jupiter.NativeSOL  // So11111111111111111111111111111111111111112
jupiter.USDC       // EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
jupiter.USDT       // Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB

jupiter.SlippageTight    // 50 bps (0.5%)
jupiter.SlippageNormal   // 100 bps (1%)
jupiter.SlippageRelaxed  // 300 bps (3%)
```

## Error Handling
```go
quote, err := client.GetQuote(ctx, req)
if err != nil {
    var apiErr *jupiter.APIError
    if errors.As(err, &apiErr) {
        switch apiErr.Code {
        case "ROUTE_NOT_FOUND":
            // No route available for this pair
        case "AMOUNT_TOO_SMALL":
            // Increase swap amount
        }
    }
    return err
}
```

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) first.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Links

- [Jupiter Documentation](https://station.jup.ag/docs)
- [Jupiter App](https://jup.ag)
- [Go Package Documentation](https://pkg.go.dev/github.com/Laminar-Bot/jupiter-go)
