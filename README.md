# Market Maker

A high-performance C++ market making system for centralized cryptocurrency exchanges.

## Overview

This is a multi-process trading system designed for algorithmic market making on centralized exchanges. The system features:

- **Multi-Exchange Support**: Binance, Deribit, and GRVT exchanges
- **Multi-Process Architecture**: Isolated processes for market data, order execution, position tracking, and strategy execution
- **High-Performance Communication**: ZeroMQ for inter-process messaging
- **Real-Time Data**: WebSocket connections for market data and order updates
- **Strategy Framework**: Extensible strategy development with GLFT model implementation

## Architecture

The system consists of four main processes per exchange:

| Process | Purpose |
|---------|---------|
| **Market Server** | Subscribes to public market data (orderbooks, trades) |
| **Trading Engine** | Handles order execution via HTTP and private WebSocket |
| **Position Server** | Tracks positions and account balances |
| **Trader** | Runs trading strategies with Mini OMS/PMS |

All processes communicate via ZeroMQ for high-performance, low-latency messaging.

## Quick Start

### Prerequisites

```bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y \
    build-essential cmake git pkg-config \
    libzmq3-dev libwebsockets-dev libssl-dev \
    libcurl4-openssl-dev libjsoncpp-dev libsimdjson-dev \
    libprotobuf-dev protobuf-compiler libuv1-dev
```

### Build

```bash
cd cpp
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Configure

```bash
# Set exchange API credentials
export BINANCE_API_KEY=your_api_key
export BINANCE_API_SECRET=your_api_secret
```

### Run

```bash
# Start all processes for Binance
./bin/market_server BINANCE ../config/market_server_binance.ini &
./bin/position_server BINANCE ../config/position_server_binance.ini &
./bin/trading_engine BINANCE ../config/trading_engine_binance.ini &
./bin/trader ../config/trader.ini &
```

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture Diagram](docs/architecture_diagram.md) | System architecture and component relationships |
| [Components Overview](docs/components_overview.md) | Detailed description of all system components |
| [Configuration Guide](docs/configuration.md) | Configuration file formats and options |
| [Deployment Guide](docs/deploy.md) | Production deployment instructions |
| [Exchange Integration](docs/exchange_guide.md) | Guide for adding new exchange support |
| [Install Dependencies](docs/install_dependencies.md) | Dependency installation for all platforms |
| [Interfaces](docs/interfaces.md) | Exchange interface documentation |

## Supported Exchanges

| Exchange | Status | Asset Types |
|----------|--------|-------------|
| **Binance** | ✅ Complete | Futures, Spot |
| **Deribit** | ✅ Complete | Perpetuals |
| **GRVT** | ✅ Complete | Perpetuals |

## Key Features

- **Clean Interface Design**: Standardized interfaces for all exchanges (OMS, PMS, DataFetcher, Subscriber)
- **Process Isolation**: Failure in one process doesn't affect others
- **Comprehensive Logging**: Structured logging with configurable levels
- **Error Handling**: Retry policies, circuit breakers, graceful degradation
- **Test Suite**: 200+ test cases covering integration, performance, and security

## Project Structure

```
market_maker/
├── cpp/                      # C++ trading system
│   ├── exchanges/            # Exchange implementations
│   │   ├── binance/          # Binance integration
│   │   ├── deribit/          # Deribit integration
│   │   └── grvt/             # GRVT integration
│   ├── market_server/        # Market data server
│   ├── trading_engine/       # Order execution engine
│   ├── position_server/      # Position tracking
│   ├── trader/               # Strategy framework
│   ├── strategies/           # Trading strategies
│   ├── utils/                # Shared utilities
│   ├── proto/                # Protocol buffer definitions
│   └── tests/                # Test suite
├── docs/                     # Documentation
└── README.md
```

## Status

⚠️ **Architecture Complete, Production Testing Required**

The system architecture is complete with all major components implemented. Production testing and validation is required before live deployment.

## License

See [LICENSE](LICENSE) for details.
