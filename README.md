# Asymmetric Perpetuals

**A derivatives exchange for shorting low-liquidity, high-volatility assets (memecoins) on Solana.**

> **[📺 Watch the Demo](https://drive.google.com/file/d/1xxhQyD1LbU969Dg4W8wHucpsOxv_E5SK/view?usp=drive_link)**

## The Problem

Memecoins have found product-market fit in crypto, but they're missing a critical trading feature: **the ability to short**. Traditional shorting mechanisms (lending markets, perpetual futures orderbooks) don't work for memecoins because:

- **Low liquidity** → Market makers won't participate
- **High volatility** → Traditional risk models break down
- **Asymmetric nature** → Everyone expects memecoins to eventually go to zero

Without institutional market makers, retail traders need a new way to both provide liquidity *and* take short positions.

## The Solution

Asymmetric Perpetuals introduces a novel derivative exchange design that allows users to short memecoins by:

1. **Pooling long liquidity AMM-style** → Retail users provide liquidity into a shared pool that shorts trade against
2. **Creating individual short contracts** → Each short position is isolated with its own margin and terms
3. **Implementing asymmetric incentives** → Longs receive an amplified payoff (via an "asymmetry factor") to compensate for the risk of providing liquidity in a market expected to decline

### How It Works

The exchange uses an **asymmetry factor** — a multiplier derived from an AMM-style mathematical curve that determines the relative payoff between longs and shorts. This factor dynamically adjusts based on market conditions:
- **More short liquidity** → Asymmetry factor increases (longs need more incentive)
- **More long liquidity** → Asymmetry factor decreases (shorts become more attractive)

When you open a short position, a smart contract is created with your margin, locking in the current asymmetry factor at that moment. 

Similar to traditional perpetual futures, a periodic **funding rate** rebalances funds between longs and shorts based on price movements:
- If price goes **up** → Shorts pay longs (payment scaled by asymmetry factor)
- If price goes **down** → Longs pay shorts (payment unscaled)

This creates a self-balancing market where the asymmetry factor continuously adjusts based on the ratio of long to short liquidity, ensuring the right incentives for both sides.

## Architecture

This monorepo contains three main components:

### **asymmetric-perpetuals-contracts**
Solana smart contracts (Anchor framework) that implement the core perpetual logic:
- Contract creation and management
- Liquidity provision and withdrawal
- Funding rate calculations
- Price synchronization

### **asymmetric-perpetuals-backend**
Off-chain infrastructure written in Rust:
- **Listener**: Monitors Solana blockchain for relevant events
- **Indexer**: Processes and stores contract state
- **Price Oracles**: Collects real-time price data from DEX pools
- **Price API**: Serves OHLCV candle data and WebSocket feeds
- **Chain Worker**: Executes periodic on-chain operations (funding rates)

See the [backend repository](https://github.com/pysel/asymmetric-perpetuals-backend) for detailed documentation.

### **asymmetric-perpetuals-frontend**
Next.js web application providing:
- Wallet connection and transaction signing
- Real-time trading charts (TradingView integration)
- Position management interface
- Market selection and analytics

## Key Features

- **Solana-native**: Fast settlement and low transaction costs
- **No KYC**: Permissionless access via wallet connection
- **Real-time data**: WebSocket price feeds and live position updates
- **Isolated risk**: Each short position is a separate contract
- **Dynamic incentives**: Asymmetry factor adjusts based on market conditions

## Limitations & Considerations

- **Price manipulation risk**: Thin liquidity makes spot price manipulation easier (mitigated by focusing on migrated pump.fun tokens and taker fees)
- **Unpredictable long PNL**: Since long liquidity is pooled, individual LP returns depend on the current mix of short positions and their asymmetry factors
- **Oracle dependency**: Relies on spot DEX prices for funding rate calculations

## Getting Started

Each subrepository contains its own setup instructions:

1. **Contracts**: Deploy or connect to existing program on Solana devnet/mainnet
2. **Backend**: Run infrastructure services (requires NATS, Redis, InfluxDB)
3. **Frontend**: Configure environment variables and start Next.js dev server

Refer to the README in each directory for detailed setup steps.

---

**Status**: Active development. Core functionality implemented and tested on Solana devnet.