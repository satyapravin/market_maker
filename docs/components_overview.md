# System Components Overview

This document provides a comprehensive description of all components in the C++ market making trading system.

## System Architecture

The system is a **C++ multi-process trading system** for centralized exchange market making.

---

## C++ Trading System

### Process Architecture

The C++ component uses a **multi-process architecture** where each exchange runs specialized processes:

#### 1. **Trader Process** (Single Instance)
**Main Strategy Execution Process**

**Components**:
- **TraderLib** (`trader_lib.hpp/cpp`)
  - Core trading library
  - Manages strategy container lifecycle
  - Coordinates ZMQ adapters
  - Provides statistics and monitoring

- **StrategyContainer** (`strategy_container.hpp/cpp`)
  - Manages single strategy instance
  - Routes events to strategy
  - Handles ZMQ adapter lifecycle
  - Delegates orders to MiniOMS

- **AbstractStrategy** (`abstract_strategy.hpp/cpp`)
  - Base class for all strategies
  - Defines event handler interface
  - Provides common utilities
  - Implementations: `MarketMakingStrategy`

- **MiniOMS** (`mini_oms.hpp/cpp`)
  - Order state management
  - State machine (NEW → SENT → ACK → FILLED/CANCELLED)
  - Order routing to Trading Engine
  - Order statistics and queries

- **MiniPMS** (`mini_pms.hpp/cpp`)
  - Position tracking
  - Balance management
  - Position reconciliation
  - Risk limit monitoring

- **ZMQ Adapters**:
  - **ZmqOMSAdapter** - Order events (subscribes from Trading Engine)
  - **ZmqMDSAdapter** - Market data (subscribes from Market Server)
  - **ZmqPMSAdapter** - Position updates (subscribes from Position Server)

**Data Flow**:
- Receives market data → Strategy processes → Generates orders → MiniOMS → Trading Engine
- Receives order events → MiniOMS → Strategy
- Receives position updates → MiniPMS → Strategy

#### 2. **Market Server** (Per Exchange)
**Purpose**: Real-time market data aggregation

**Components**:
- **MarketServerService** (`market_server_service.hpp/cpp`)
  - Service orchestration
  - Statistics collection
  - Health monitoring

- **MarketServerLib** (`market_server_lib.hpp/cpp`)
  - Core market data processing
  - ZMQ publishing
  - Message handling

- **Exchange Subscribers** (implements `IExchangeSubscriber`):
  - **BinanceSubscriber** - Binance public WebSocket
  - **DeribitSubscriber** - Deribit public WebSocket
  - **GrvtSubscriber** - GRVT public WebSocket

- **Market Data Parsers**:
  - Exchange-specific message parsing
  - Normalization to protobuf format
  - Orderbook aggregation

- **ZMQ Publisher** (`zmq_publisher.hpp/cpp`)
  - Publishes orderbooks and trades
  - Topic-based filtering
  - High-performance messaging

**Data Flow**:
Exchange WebSocket → Subscriber → Parser → Market Server → ZMQ → Trader

**Responsibilities**:
- Subscribe to orderbook and trade streams
- Parse exchange-specific message formats
- Normalize to common protobuf format
- Publish via ZMQ to Trader process

#### 3. **Trading Engine** (Per Exchange)
**Purpose**: Order execution and management

**Components**:
- **TradingEngineService** (`trading_engine_service.hpp/cpp`)
  - Service lifecycle management
  - Configuration loading
  - Statistics and health monitoring

- **TradingEngineLib** (`trading_engine_lib.hpp/cpp`)
  - Core order execution logic
  - HTTP + WebSocket coordination
  - Order state management

- **Exchange OMS** (implements `IExchangeOMS`):
  - **BinanceOMS** - Binance private WebSocket + HTTP
  - **DeribitOMS** - Deribit private WebSocket + HTTP
  - **GrvtOMS** - GRVT private WebSocket + HTTP

- **HTTP Handler** (`http_handler.hpp/cpp`)
  - REST API calls
  - Authentication and signing
  - Rate limiting
  - Retry logic

- **ZMQ Subscriber** (`zmq_subscriber.hpp/cpp`)
  - Subscribes to order requests from Trader
  - Topic filtering
  - Message deserialization

- **ZMQ Publisher** (`zmq_publisher.hpp/cpp`)
  - Publishes order events to Trader
  - Order status updates
  - Fill notifications

**Data Flow**:
Trader → ZMQ → Trading Engine → Exchange (WebSocket/HTTP) → Trading Engine → ZMQ → Trader

**Responsibilities**:
- Receive order requests from Trader
- Execute orders via WebSocket or HTTP
- Handle order cancellations and modifications
- Publish order events back to Trader
- Manage authentication and rate limiting

#### 4. **Position Server** (Per Exchange)
**Purpose**: Position and balance tracking

**Components**:
- **PositionServerService** (`position_server_service.hpp/cpp`)
  - Service orchestration
  - Statistics collection
  - Health monitoring

- **PositionServerLib** (`position_server_lib.hpp/cpp`)
  - Core position tracking logic
  - ZMQ publishing
  - Balance management

- **Exchange PMS** (implements `IExchangePMS`):
  - **BinancePMS** - Binance private WebSocket
  - **DeribitPMS** - Deribit private WebSocket
  - **GrvtPMS** - GRVT private WebSocket + REST polling

- **ZMQ Publisher** (`zmq_publisher.hpp/cpp`)
  - Publishes position updates
  - Balance updates
  - Account updates

**Data Flow**:
Exchange WebSocket → PMS → Position Server → ZMQ → Trader

**Responsibilities**:
- Subscribe to position and balance streams
- Track real-time positions
- Calculate PnL
- Publish updates to Trader process

---

## Exchange Integration Layer

Each exchange (Binance, Deribit, GRVT) implements **4 specialized interfaces**:

### 1. **IExchangeSubscriber** (Market Server)
- **Purpose**: Public market data via WebSocket
- **Key Methods**:
  - `connect()` - Establish WebSocket connection
  - `subscribe_orderbook()` - Subscribe to orderbook updates
  - `subscribe_trades()` - Subscribe to trade stream
  - `set_orderbook_callback()` - Real-time orderbook callbacks
  - `set_trade_callback()` - Real-time trade callbacks

### 2. **IExchangeOMS** (Trading Engine)
- **Purpose**: Order management via private WebSocket
- **Key Methods**:
  - `connect()` - Establish authenticated WebSocket
  - `place_market_order()` - Place market order
  - `place_limit_order()` - Place limit order
  - `cancel_order()` - Cancel order
  - `replace_order()` - Modify order
  - `set_order_status_callback()` - Order event callbacks

### 3. **IExchangePMS** (Position Server)
- **Purpose**: Position management via private WebSocket
- **Key Methods**:
  - `connect()` - Establish authenticated WebSocket
  - `set_position_update_callback()` - Position update callbacks
  - `set_account_balance_update_callback()` - Balance update callbacks

### 4. **IExchangeDataFetcher** (Trader - Startup)
- **Purpose**: Startup state recovery via HTTP
- **Key Methods**:
  - `get_open_orders()` - Query current open orders
  - `get_positions()` - Query current positions
  - `get_balances()` - Query account balances
  - `set_auth_credentials()` - Set API credentials

---

## Utility Components

### 1. **Logging System** (`utils/logging/`)
- **Logger** (`logger.hpp/cpp`)
  - Centralized logging infrastructure
  - Log level management
  - File and console output

- **LogManager** (`log_manager.hpp`)
  - Singleton log manager
  - Logger instance management

- **Log Helper** (`log_helper.hpp`)
  - Convenience macros: `LOG_INFO_COMP`, `LOG_DEBUG_COMP`, `LOG_ERROR_COMP`, `LOG_WARN_COMP`
  - Component-based logging
  - Structured log format

### 2. **Configuration Management** (`utils/config/`)
- **ProcessConfigManager** (`process_config_manager.hpp/cpp`)
  - INI file parsing
  - Environment variable substitution
  - Thread-safe configuration access
  - Section-based organization

### 3. **ZMQ Utilities** (`utils/zmq/`)
- **ZmqPublisher** (`zmq_publisher.hpp/cpp`)
  - High-performance message publishing
  - Topic-based filtering
  - Connection management

- **ZmqSubscriber** (`zmq_subscriber.hpp/cpp`)
  - Message subscription
  - Topic filtering
  - Automatic reconnection

### 4. **HTTP Handler** (`utils/http/`)
- **HttpHandler** (`http_handler.hpp/cpp`)
  - REST API calls
  - Authentication (HMAC signatures)
  - Rate limiting
  - Retry logic with exponential backoff

### 5. **WebSocket Transport** (`exchanges/websocket/`)
- **IWebSocketTransport** (`i_websocket_transport.hpp`)
  - Abstract WebSocket interface
  - Allows injection for testing
  - Exchange-specific implementations

### 6. **App Service** (`utils/app_service/`)
- **AppService** (`app_service.hpp/cpp`)
  - Process lifecycle management
  - Daemonization support
  - Signal handling
  - Health monitoring
  - Statistics reporting

### 7. **Exchange Monitor** (`utils/oms/`)
- **ExchangeMonitor** (`exchange_monitor.hpp/cpp`)
  - Health monitoring
  - Performance metrics
  - Alert generation
  - Exchange status tracking

### 8. **Message Handlers** (`utils/handlers/`)
- **MessageHandler** (`message_handler.hpp/cpp`)
  - Generic message processing
  - Topic-based routing
  - Callback management

- **MessageHandlerManager** (`message_handler_manager.hpp/cpp`)
  - Multiple handler management
  - Handler lifecycle
  - Topic routing

---

## Strategy Components

### 1. **Base Strategy** (`strategies/base_strategy/`)
- **AbstractStrategy** (`abstract_strategy.hpp/cpp`)
  - Pure virtual interface
  - Event handlers: `on_market_data()`, `on_order_event()`, `on_position_update()`, `on_trade_execution()`
  - Lifecycle: `start()`, `stop()`, `is_running()`
  - Configuration and risk management

### 2. **Market Making Strategy** (`strategies/mm_strategy/`)
- **MarketMakingStrategy** (`market_making_strategy.hpp/cpp`)
  - Implements `AbstractStrategy`
  - Uses GLFT model for quote generation
  - Order management and risk controls
  - Inventory-aware quoting

- **GLFT Target** (`models/glft_target.hpp/cpp`)
  - GLFT model implementation for market making
  - Quote price calculation
  - Spread management

- **Strategy Config** (`market_making_strategy_config.hpp`)
  - Configuration parameters
  - Risk limits
  - Model parameters

---

## Protocol Buffers

### Message Types (`proto/`)

1. **Order Messages** (`order.proto`)
   - `OrderRequest` - Order placement request
   - `OrderEvent` - Order status updates
   - Order types, sides, status enums

2. **Market Data** (`market_data.proto`)
   - `OrderBookSnapshot` - Orderbook data
   - `Trade` - Trade execution data
   - Price level structures

3. **Position Messages** (`position.proto`)
   - `PositionUpdate` - Position changes
   - Position tracking fields

4. **Account Balance** (`acc_balance.proto`)
   - `AccountBalanceUpdate` - Balance changes
   - `AccountBalance` - Account state

---

## Data Flow Summary

### Complete Trading Flow

1. **Market Data**: Exchange → Market Server → ZMQ → Trader → Strategy
2. **Order Generation**: Strategy → MiniOMS → ZMQ → Trading Engine → Exchange
3. **Order Events**: Exchange → Trading Engine → ZMQ → MiniOMS → Strategy
4. **Position Updates**: Exchange → Position Server → ZMQ → MiniPMS → Strategy

### Inter-Process Communication

- **ZMQ Topics**:
  - Market data: `market_data` topic
  - Order requests: `orders` topic
  - Order events: `order_events` topic
  - Position updates: `position_updates` topic

- **ZMQ Endpoints**:
  - Market Server publishes: `tcp://127.0.0.1:7001`
  - Trading Engine subscribes: `tcp://127.0.0.1:7003`
  - Position Server publishes: `tcp://127.0.0.1:7002`
  - Trader subscribes to all above

---

## Component Dependencies

### Trader Process Dependencies
- Requires: Market Server, Trading Engine, Position Server (all running)
- Provides: Strategy execution, order generation

### Market Server Dependencies
- Requires: Exchange WebSocket connectivity
- Provides: Market data to Trader

### Trading Engine Dependencies
- Requires: Exchange API credentials, Trader (for order requests)
- Provides: Order execution to Trader

### Position Server Dependencies
- Requires: Exchange WebSocket connectivity
- Provides: Position updates to Trader

---

## Key Design Principles

1. **Process Isolation**: Each process handles one responsibility
2. **Interface-Based Design**: Exchanges implement standardized interfaces
3. **ZMQ Communication**: High-performance inter-process messaging
4. **Dual Connectivity**: HTTP + WebSocket for redundancy
5. **Strategy Abstraction**: Pluggable strategy framework
6. **Centralized Logging**: Structured logging throughout
7. **Configuration Management**: Per-process INI configuration
8. **Error Handling**: Comprehensive error handling and recovery

---

## Component Interaction Example

**Example: Placing a Market Order**

1. **Strategy** (`MarketMakingStrategy`) decides to place order
2. **MiniOMS** validates order and assigns client order ID
3. **ZmqOMSAdapter** serializes order request to protobuf
4. **ZMQ** transports message to Trading Engine
5. **Trading Engine** receives order request
6. **Exchange OMS** (`BinanceOMS`) places order via WebSocket
7. **Exchange** executes order and sends confirmation
8. **Exchange OMS** receives order event
9. **Trading Engine** publishes order event via ZMQ
10. **ZmqOMSAdapter** receives order event
11. **MiniOMS** updates order state
12. **Strategy** receives order event callback
13. **Strategy** processes fill and updates internal state

This demonstrates the complete round-trip through all components for a single order execution.

