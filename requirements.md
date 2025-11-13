# Reactive Trading System - Complete Requirements Specification
## EARS Format (Easy Approach to Requirements Syntax)

**Version:** 0.1  
**Date:** November 13, 2025  
**Language:** English  
**Target Platform:** Julia with Genie.jl Web Framework

---

## Table of Contents

1. [Document Overview](#document-overview)
2. [System Architecture](#system-architecture)
3. [Ubiquitous Requirements](#ubiquitous-requirements)
4. [Event-Driven Requirements](#event-driven-requirements)
5. [Unwanted Behaviors](#unwanted-behaviors)
6. [State-Driven Requirements](#state-driven-requirements)
7. [Optional Features](#optional-features)
8. [Complex Requirements](#complex-requirements)
9. [Requirements Traceability Matrix](#requirements-traceability-matrix)
10. [Glossary](#glossary)

---

## Document Overview

This document specifies requirements for the Reactive Trading System using the EARS (Easy Approach to Requirements Syntax) format. The system integrates:

- **Reactive Programming**: Rocket.jl for event-driven architecture
- **Online Statistics**: OnlineStatsBase.jl for streaming computations
- **Trading Engine**: Fastback.jl for account management and order execution
- **Exchange Integration**: Modular exchange connectivity (e.g., Binance.jl for Binance)
- **Technical Indicators**: OnlineTechnicalIndicators.jl for signal generation
- **Performance Analytics**: OnlinePortfolioAnalytics.jl for metrics
- **Time Resampling**: OnlineResamplers.jl (https://github.com/femtotrader/OnlineResamplers.jl) for streaming data aggregation
- **Time Management**: Timestamps64 for precise temporal operations
- **Web Interface**: Genie.jl for user interaction and visualization

### EARS Patterns

- **Ubiquitous**: Requirements that apply everywhere
- **Event-Driven**: WHEN [trigger] THEN [response]
- **Unwanted**: IF [condition] THEN [system response]
- **State-Driven**: WHILE [state] THEN [behavior]
- **Optional**: WHERE [feature enabled] THEN [capability]
- **Complex**: Combinations of multiple patterns

---

## System Architecture

### Core Components

1. **Reactive Pipeline**: Observable-based data flow from market data to execution
2. **Strategy Engine**: Modular strategy framework with AbstractTradingStrategy interface
3. **Data Sources**: REST API, WebSocket, CSV files, simulated data
4. **Execution Layer**: Paper trading and live trading with Fastback.jl
5. **Risk Management**: Position limits, drawdown controls, daily loss limits
6. **Analytics Engine**: Real-time metrics with OnlinePortfolioAnalytics.jl
7. **Web Interface**: Genie.jl application for monitoring and control
8. **Notification System**: Telegram and Pushover integration for alerts

---

## 1. UBIQUITOUS REQUIREMENTS

### 1.1 Performance and Reliability

**REQ-UBI-001:** The system SHALL process market data updates with a maximum latency of 100ms.

**REQ-UBI-002:** The system SHALL be available 99% of the time during market hours.

**REQ-UBI-003:** The system SHALL log all transactions and trading decisions with millisecond-precision timestamps using Timestamps64.

**REQ-UBI-004:** The system SHALL validate data integrity before processing any market data.

**REQ-UBI-005:** The system SHALL support parallel computation for multi-strategy backtesting.

**REQ-UBI-006:** The system SHALL use OnlineStatsBase.jl for all streaming statistical computations.

**REQ-UBI-007:** The system SHALL use OnlineResamplers.jl for streaming time-based data aggregation and resampling.

**REQ-UBI-008:** The system SHALL generate unique order identifiers using Fastback.jl's oid!() function.

### 1.2 Security

**REQ-UBI-009:** The system SHALL encrypt all API keys and credentials with AES-256.

**REQ-UBI-010:** The system SHALL implement two-factor authentication for administrative access.

**REQ-UBI-011:** The system SHALL maintain an immutable audit trail of all operations.

**REQ-UBI-012:** The system SHALL never log sensitive credentials in plain text.

### 1.3 Architecture

**REQ-UBI-013:** The system SHALL implement a reactive architecture based on observables using Rocket.jl.

**REQ-UBI-014:** The system SHALL process data asynchronously without blocking other components.

**REQ-UBI-015:** The system SHALL maintain modular separation between data ingestion, strategy engine, risk management, order execution, and analytics.

**REQ-UBI-016:** The system SHALL support hot-reload of configurations without system restart.

---

## 2. EVENT-DRIVEN REQUIREMENTS

### 2.1 Market Data Processing

**REQ-EVT-001:** WHEN a new market data tick arrives THEN the system SHALL update technical indicators within 50ms.

**REQ-EVT-002:** WHEN market data is received THEN the system SHALL format it into standardized MarketData structures with Timestamps64.

**REQ-EVT-003:** WHEN a data source connection is lost THEN the system SHALL attempt automatic reconnection every 5 seconds for 5 minutes maximum.

**REQ-EVT-004:** WHEN data anomalies are detected (abnormal prices) THEN the system SHALL suspend trading and alert the administrator.

**REQ-EVT-005:** WHEN a new order book update is received THEN the system SHALL recalculate available liquidity levels.

**REQ-EVT-006:** WHEN CSV data is loaded THEN the system SHALL parse it into MarketData objects with proper timestamp conversion.

**REQ-EVT-007:** WHEN a CSV file contains invalid data THEN the system SHALL skip invalid rows and log warnings.

### 2.2 Strategy Signal Generation

**REQ-EVT-008:** WHEN market data arrives THEN strategies SHALL process it via the fit!() method for streaming updates.

**REQ-EVT-009:** WHEN a strategy analyzes data THEN it SHALL generate signals (BUY, SELL, HOLD) with quantity and confidence level.

**REQ-EVT-010:** WHEN a trading signal is generated THEN the system SHALL propagate it through the reactive pipeline.

**REQ-EVT-011:** WHEN the Moving Average Crossover strategy detects fast MA crossing above slow MA THEN it SHALL generate a BUY signal.

**REQ-EVT-012:** WHEN the Moving Average Crossover strategy detects fast MA crossing below slow MA THEN it SHALL generate a SELL signal.

**REQ-EVT-013:** WHEN a time bar completes (1min, 5min, hourly) THEN the system SHALL execute update logic for all active strategies.

### 2.3 Order Execution

**REQ-EVT-014:** WHEN an entry signal is generated THEN the system SHALL verify risk management conditions before placing orders.

**REQ-EVT-015:** WHEN an order is partially filled THEN the system SHALL update the position and recalculate exposure.

**REQ-EVT-016:** WHEN an order is rejected by the broker THEN the system SHALL log the reason and notify the strategy manager.

**REQ-EVT-017:** WHEN an order is placed on an exchange THEN the system SHALL use the appropriate exchange connector's REST API for order execution.

**REQ-EVT-018:** WHEN an order execution is confirmed THEN the system SHALL update P&L in real-time using update_pnl!().

**REQ-EVT-019:** WHEN paper trading executes an order THEN the system SHALL apply configurable fill ratio to simulate slippage.

**REQ-EVT-020:** WHEN paper trading executes an order THEN the system SHALL calculate and apply configurable commission fees.

### 2.4 Risk Management Events

**REQ-EVT-021:** WHEN maximum drawdown is reached THEN the system SHALL liquidate all positions and suspend trading.

**REQ-EVT-022:** WHEN daily loss limit is hit THEN the system SHALL stop all new orders until the next trading day.

**REQ-EVT-023:** WHEN a risk limit is breached THEN the system SHALL trigger immediate alerts via configured notification drivers.

**REQ-EVT-024:** WHEN volatility exceeds 2x the 20-period moving average THEN the system SHALL tighten trailing stops by 50% if profitable.

### 2.5 Web Interface Events

**REQ-EVT-025:** WHEN a user imports candle data THEN the Web Interface SHALL initiate download from the selected exchange.

**REQ-EVT-026:** WHEN a backtest is initiated THEN the Web Interface SHALL execute it using the Reactive Trading System.

**REQ-EVT-027:** WHEN optimization completes THEN the Web Interface SHALL display the best parameter configuration.

**REQ-EVT-028:** WHEN paper trading is started THEN the Web Interface SHALL connect to real-time market data streams.

**REQ-EVT-029:** WHEN live trading is started THEN the Web Interface SHALL connect to the exchange for order execution.

**REQ-EVT-030:** WHEN a notification driver is configured THEN the Web Interface SHALL send a test notification for verification.

---

## 3. UNWANTED BEHAVIORS

### 3.1 Error Protection

**REQ-UNW-001:** IF the system detects an infinite loop in a strategy THEN the system SHALL terminate that strategy execution and alert the administrator.

**REQ-UNW-002:** IF an order exceeds defined position limits THEN the system SHALL reject the order and log a warning.

**REQ-UNW-003:** IF available memory drops below 20% THEN the system SHALL stop non-critical operations and free resources.

**REQ-UNW-004:** IF a strategy generates more than 10 orders in less than one second THEN the system SHALL suspend that strategy for 60 seconds.

**REQ-UNW-005:** IF price data is not received for 30 seconds THEN the system SHALL enter degraded mode and avoid new entries.

**REQ-UNW-006:** IF a CSV file does not exist THEN the system SHALL throw a clear error message.

**REQ-UNW-007:** IF CSV column mappings are invalid THEN the system SHALL report specific mapping errors with examples.

### 3.2 Loss Prevention

**REQ-UNW-008:** IF cumulative loss exceeds 5% of initial capital in one day THEN the system SHALL close all positions and stop trading.

**REQ-UNW-009:** IF an abnormal spread is detected between expected and execution price (>2%) THEN the system SHALL cancel pending orders.

**REQ-UNW-010:** IF a flash crash is detected (drop >5% in <1min) THEN the system SHALL suspend trading for 5 minutes.

**REQ-UNW-011:** IF maximum drawdown threshold is exceeded THEN the system SHALL automatically liquidate positions.

**REQ-UNW-012:** IF Sharpe Ratio drops below minimum threshold THEN the system SHALL alert of performance degradation.

**REQ-UNW-013:** IF Win Rate drops significantly THEN the system SHALL suggest strategy review.

### 3.3 System Protection

**REQ-UNW-014:** IF exchange API rate limits are reached THEN the system SHALL implement automatic backoff with exponential retry.

**REQ-UNW-015:** IF network latency exceeds critical threshold THEN the system SHALL alert of connectivity issues.

**REQ-UNW-016:** IF data quality degrades (missing data >5%) THEN the system SHALL signal data quality problems.

**REQ-UNW-017:** IF average slippage becomes excessive THEN the system SHALL recommend execution parameter adjustments.

**REQ-UNW-018:** IF system uptime drops below 99% THEN the system SHALL trigger infrastructure diagnostics.

---

## 4. STATE-DRIVEN REQUIREMENTS

### 4.1 Trading Modes

**REQ-STA-001:** WHILE the system is in "backtesting" mode THEN the system SHALL use historical data and simulate execution without broker connection.

**REQ-STA-002:** WHILE the system is in "paper trading" mode THEN the system SHALL receive real-time data but simulate executions.

**REQ-STA-003:** WHILE the system is in "live trading" mode THEN the system SHALL execute real orders on the connected broker.

**REQ-STA-004:** WHILE a position is open THEN the system SHALL continuously monitor stop loss and take profit levels.

**REQ-STA-005:** WHILE the market is closed THEN the system SHALL enter idle mode and prepare data for the next session.

**REQ-STA-006:** WHILE backtesting is running THEN the Web Interface SHALL display real-time trade activity and metrics.

**REQ-STA-007:** WHILE optimization is running THEN the Web Interface SHALL display progress information and allow navigation without interruption.

### 4.2 Connection States

**REQ-STA-008:** WHILE broker connection is "connecting" THEN the system SHALL block all order placement attempts.

**REQ-STA-009:** WHILE the system is in maintenance mode THEN the system SHALL reject new strategies and maintain existing positions.

**REQ-STA-010:** WHILE parameter optimization is executing THEN the system SHALL disable live trading.

**REQ-STA-011:** WHILE an exchange testnet mode is active THEN the system SHALL clearly display testnet mode in the interface.

**REQ-STA-012:** WHILE data is being imported THEN the Web Interface SHALL display import progress and status.

### 4.3 Strategy States

**REQ-STA-013:** WHILE a strategy is active THEN it SHALL process all market data updates via its update function.

**REQ-STA-014:** WHILE a strategy has configurable parameters THEN they SHALL be stored in a parameters dictionary.

**REQ-STA-015:** WHILE Multiple strategies are running THEN the system SHALL apply defined priority rules for conflicting signals.

---

## 5. OPTIONAL FEATURES

### 5.1 Advanced Trading Features

**REQ-OPT-001:** WHERE margin trading is configured THEN the system SHALL support trading on margin with capital requirement verification.

**REQ-OPT-002:** WHERE notifications are enabled THEN the system SHALL send alerts via email, SMS, Telegram, or Pushover for critical events.

**REQ-OPT-003:** WHERE multi-asset trading is specified THEN the system SHALL support simultaneous trading of stocks, crypto, and forex.

**REQ-OPT-004:** WHERE reporting is requested THEN the system SHALL generate daily, weekly, and monthly performance reports.

**REQ-OPT-005:** WHERE alternative data sources are configured THEN the system SHALL support integration with Bloomberg and Reuters.

**REQ-OPT-006:** WHERE advanced order types are enabled THEN the system SHALL support LIMIT, STOP_LOSS, and other order types as supported by the connected exchange.

### 5.2 Analysis and Optimization

**REQ-OPT-007:** WHERE walk-forward analysis is enabled THEN the system SHALL perform rolling window validation for strategy robustness.

**REQ-OPT-008:** WHERE genetic optimization is configured THEN the system SHALL support evolutionary parameter optimization.

**REQ-OPT-009:** WHERE advanced metrics are requested THEN the system SHALL calculate Sharpe, Sortino, Calmar, PSR, and MAE/MFE.

**REQ-OPT-010:** WHERE machine learning is specified THEN the system SHALL support ML-based algorithmic trading strategies.

**REQ-OPT-011:** WHERE realistic backtesting is configured THEN the system SHALL simulate slippage and transaction costs.

**REQ-OPT-012:** WHERE optimization history is requested THEN the Web Interface SHALL provide a History page listing all optimization runs.

### 5.3 Data Management

**REQ-OPT-013:** WHERE CSV headers are present THEN the system SHALL automatically detect and use them.

**REQ-OPT-014:** WHERE CSV headers are absent THEN the system SHALL accept column indices for data mapping.

**REQ-OPT-015:** WHERE additional CSV columns exist (bid, ask, metadata) THEN the system SHALL optionally parse them if specified.

**REQ-OPT-016:** WHERE data export is requested THEN the Web Interface SHALL support CSV/JSON export of metrics and trades.

**REQ-OPT-017:** WHERE external analysis is needed THEN the system SHALL provide API integration with Python/R tools.

### 5.4 Visualization and Reporting

**REQ-OPT-018:** WHERE chart visualization is requested THEN the Web Interface SHALL display interactive charts using LightweightCharts.jl.

**REQ-OPT-019:** WHERE comparative analysis is needed THEN the system SHALL display metrics side-by-side for multiple strategies.

**REQ-OPT-020:** WHERE performance distribution is requested THEN the system SHALL generate win/loss histograms.

---

## 6. COMPLEX REQUIREMENTS

### 6.1 Advanced Order Management

**REQ-CPX-001:** WHILE the system is in "live trading" mode, WHEN an entry signal is generated, IF market conditions are favorable (spread < 0.1%, liquidity > threshold) THEN the system SHALL place a limit order at the best available price.

**REQ-CPX-002:** WHILE a position is open, WHEN volatility exceeds 2x the 20-period moving average, IF unrealized profit is positive THEN the system SHALL tighten the trailing stop by 50%.

**REQ-CPX-003:** WHEN an order is placed, IF the order is not filled within 60 seconds THEN the system SHALL cancel the order AND re-evaluate market conditions, IF conditions remain valid THEN replace the order with adjusted pricing.

**REQ-CPX-004:** WHEN loading CSV data, IF timestamp formats vary THEN the system SHALL detect and parse common formats (Unix timestamps, ISO 8601, DateTime strings), IF parsing fails THEN log specific format errors with row numbers.

### 6.2 Strategy Correlation and Portfolio Management

**REQ-CPX-005:** WHILE managing multiple strategies, IF correlation between two strategies exceeds 0.8 THEN the system SHALL reduce capital allocation to the lower-performing strategy by 30%.

**REQ-CPX-006:** WHEN a major market event is detected, IF at least 3 strategies generate simultaneous exit signals THEN the system SHALL close all positions within 10 seconds.

**REQ-CPX-007:** WHILE running backtests with CSV data, IF data spans the backtest date range THEN the system SHALL filter data appropriately AND replay events in chronological order.

### 6.3 Backtesting and Validation

**REQ-CPX-008:** WHILE executing a backtest, IF a strategy generates a trade THEN the system SHALL verify historical liquidity was sufficient for execution, IF insufficient THEN simulate slippage proportional to order book imbalance.

**REQ-CPX-009:** WHEN parameter optimization is launched, IF the search space contains more than 10,000 combinations THEN the system SHALL use Bayesian optimization to reduce exploration to maximum 500 tests.

**REQ-CPX-010:** WHEN optimization is configured with walk-forward analysis, IF data is split into training and testing periods THEN the system SHALL validate parameters on out-of-sample data and report forward-tested metrics separately.

### 6.4 Real-Time Monitoring and Alerts

**REQ-CPX-011:** WHILE trading is active, WHEN the system calculates rolling statistics (1, 3, 6, 12 months), IF Sharpe Ratio degradation is detected across multiple timeframes THEN the system SHALL trigger multi-level alerts and suggest parameter review.

**REQ-CPX-012:** WHEN the Web Interface displays live trading, IF notification drivers are configured (Telegram or Pushover) THEN the system SHALL send real-time alerts for trade executions and risk breaches, WHERE user preferences specify event types.

### 6.5 Exchange Integration

**REQ-CPX-013:** WHEN connecting to an exchange, IF testnet mode is enabled THEN the system SHALL use testnet endpoints, ELSE use production endpoints, AND WHILE connected, the system SHALL manage data streams (WebSocket or equivalent) for ticker, klines, and orderbook data with automatic reconnection on failure.

**REQ-CPX-014:** WHEN placing orders on an exchange, IF order precision requirements are not met THEN the system SHALL round prices and quantities according to exchange specifications, AND IF the order still fails THEN log specific exchange error codes and retry with exponential backoff.

---

## 7. FUNCTIONAL REQUIREMENTS BY MODULE

### 7.1 Core Trading System

#### 7.1.1 Strategy Framework

**REQ-FNC-001:** The system SHALL provide AbstractTradingStrategy as the base interface for all strategies.

**REQ-FNC-002:** All strategies SHALL implement OnlineStatsBase for streaming statistical computations.

**REQ-FNC-003:** Strategies SHALL implement the fit!() method for processing market data updates.

**REQ-FNC-004:** Strategies SHALL provide last_signal() method to access the most recent trading signal.

**REQ-FNC-005:** Strategy parameters SHALL be stored in a configurable dictionary.

**REQ-FNC-006:** The system SHALL provide StrategyFactory for automatic strategy integration.

#### 7.1.2 Example Strategies

**REQ-FNC-007:** The system SHALL include a Random Trading strategy with:
- Reproducible random seed (Random.seed!(42))
- Configurable signal probability (default: 1%)
- Configurable buy/sell ratio (default: 60%/40%)
- Realistic fill ratio (default: 0.75)
- Configurable commissions (default: 0.1%)

**REQ-FNC-008:** The system SHALL include a Moving Average Crossover (Golden Cross) strategy with:
- Streaming MA calculations via OnlineTechnicalIndicators.jl
- Configurable fast and slow MA periods (default: 50, 200)
- Crossover detection logic
- Confidence level calculation based on MA divergence

**REQ-FNC-009:** Example strategies SHALL demonstrate complete integration with:
- Account creation with configurable initial capital
- Instrument registration via register_instrument!()
- Metrics collection with periodic_collector() and drawdown_collector()
- Real-time P&L updates
- Hourly data sampling
- DrawdownMode.Percentage for drawdown calculations

#### 7.1.3 Data Sources

**REQ-FNC-010:** The system SHALL support multiple data source types:
- REST API endpoints
- WebSocket streams
- CSV file import
- Simulated data generation

**REQ-FNC-011:** All data sources SHALL implement a standardized AbstractDataSource interface.

**REQ-FNC-012:** CSV data sources SHALL support:
- Custom column name mappings
- Flexible timestamp format parsing
- Optional parsing of additional columns (bid, ask, metadata)
- Automatic header detection
- Column index specification when headers are absent

**REQ-FNC-013:** The system SHALL provide example CSV formats compatible with major exchanges (Kraken, Binance).

#### 7.1.4 Account and Order Management

**REQ-FNC-014:** The system SHALL use Fastback.jl for:
- Account management with equity tracking
- Order identifier generation via oid!()
- Position management
- P&L calculation and updates

**REQ-FNC-015:** The system SHALL support order types:
- Market orders (mandatory)
- Limit orders (optional)
- Stop loss orders (optional)

**REQ-FNC-016:** Paper trading SHALL simulate:
- Configurable commission fees
- Configurable fill ratios for slippage
- Realistic order execution timing

### 7.2 Technical Indicators

**REQ-FNC-017:** The system SHALL provide OnlineTechnicalIndicators.jl for streaming indicator calculations.

**REQ-FNC-018:** The system SHALL support minimum 30 standard technical indicators including:
- Simple Moving Average (SMA)
- Exponential Moving Average (EMA)
- Relative Strength Index (RSI)
- Moving Average Convergence Divergence (MACD)
- Bollinger Bands
- Average True Range (ATR)
- Average Directional Index (ADX)
- Stochastic Oscillator

**REQ-FNC-019:** The system SHALL allow creation of custom indicators via a simple API.

**REQ-FNC-020:** Indicators SHALL calculate efficiently using circular buffers for streaming data.

### 7.2.1 Time-Based Resampling

**REQ-FNC-021:** The system SHALL use OnlineResamplers.jl (https://github.com/femtotrader/OnlineResamplers.jl) for streaming data aggregation.

**REQ-FNC-022:** The system SHALL support time-based resampling of tick data into:
- Standard intervals (1min, 5min, 15min, 30min, 1hour, 4hour, daily)
- Custom time intervals as specified by strategy requirements

**REQ-FNC-023:** The system SHALL aggregate OHLCV data in streaming fashion without reprocessing historical data.

**REQ-FNC-024:** The system SHALL support multiple concurrent resamplers for different time intervals.

**REQ-FNC-025:** Resamplers SHALL integrate seamlessly with the reactive pipeline for real-time data flow.

### 7.3 Performance Analytics

**REQ-FNC-026:** The system SHALL use OnlinePortfolioAnalytics.jl for all performance metric calculations.

**REQ-FNC-027:** The system SHALL calculate real-time financial performance metrics:
- Start Equity and End Equity
- Net Profit (%)
- Total Return (%)
- Compounding Annual Return (CAGR)
- Annualized Return
- Cumulative P&L
- Daily P&L
- Equity Curve via Fastback.jl

**REQ-FNC-028:** The system SHALL calculate risk metrics:
- Maximum Drawdown (%)
- Current Drawdown
- Drawdown Recovery Time
- Volatility (annualized standard deviation)
- Value at Risk (VaR) at 95% and 99% confidence levels
- Sharpe Ratio
- Probabilistic Sharpe Ratio (PSR) via OnlinePortfolioAnalytics.jl
- Sortino Ratio
- Alpha and Beta
- Annual Standard Deviation and Variance
- Information Ratio
- Tracking Error
- Treynor Ratio

**REQ-FNC-029:** The system SHALL calculate trading activity metrics:
- Win Rate and Loss Rate
- Profit Factor
- Profit-Loss Ratio
- Average Win and Average Loss (%)
- Expectancy
- Total Orders executed
- Trade Frequency
- Position Holding Time
- Fill Rate
- Portfolio Turnover (%)
- Total Fees
- Estimated Strategy Capacity
- Lowest Capacity Asset

**REQ-FNC-030:** The system SHALL calculate operational metrics:
- Latency (signal to execution, ms)
- Slippage (price difference)
- Commission Impact
- Market Impact
- System Uptime (%)
- Data Quality (%)

**REQ-FNC-031:** The system SHALL calculate robustness metrics:
- Calmar Ratio
- Maximum Adverse Excursion (MAE)
- Maximum Favorable Excursion (MFE)
- Consecutive Wins/Losses
- Recovery Time

**REQ-FNC-032:** The system SHALL maintain rolling statistics over periods:
- 1 month (30 days)
- 3 months (90 days)
- 6 months (180 days)
- 12 months (365 days)

**REQ-FNC-033:** For each rolling period, the system SHALL calculate:
- Average Win Rate, Average Loss Rate
- Profit Loss Ratio, Win Rate, Loss Rate
- Expectancy
- Start Equity, End Equity
- Compounding Annual Return
- Drawdown
- Total Net Profit
- Sharpe Ratio, PSR, Sortino Ratio
- Alpha, Beta
- Annual Standard Deviation, Annual Variance
- Information Ratio, Tracking Error, Treynor Ratio
- Portfolio Turnover
- VaR 95% and 99%
- Drawdown Recovery

**REQ-FNC-034:** The system SHALL integrate metrics collectors into the reactive pipeline:
- periodic_collector() for equity sampling
- drawdown_collector() for drawdown tracking
- Custom collectors for additional metrics

**REQ-FNC-035:** The system SHALL store transaction history with Timestamps64 for retrospective analysis.

### 7.4 Risk Management

**REQ-FNC-036:** The system SHALL provide configurable risk limits:
- Maximum position size per asset
- Maximum total exposure as percentage of capital
- Daily loss limit
- Maximum drawdown threshold

**REQ-FNC-037:** The system SHALL support position management:
- Stop loss orders
- Take profit targets
- Trailing stops with configurable parameters

**REQ-FNC-038:** The system SHALL calculate position sizing based on:
- Risk per trade (e.g., 1% of capital)
- Volatility-based sizing (ATR)
- Fixed fractional sizing

**REQ-FNC-039:** The system SHALL implement circuit breakers:
- Automatic trading suspension on extreme losses
- Manual override capabilities
- Resumption validation procedures

### 7.5 Exchange Integration

**REQ-FNC-040:** The system SHALL provide an abstract exchange interface that all exchange implementations must follow.

**REQ-FNC-041:** The system SHALL support multiple exchange implementations through a pluggable architecture.

**REQ-FNC-042:** Each exchange implementation SHALL support:
- Authentication with API credentials
- Market data streaming (WebSocket or equivalent)
- Order execution via REST API or equivalent
- Account information retrieval
- Balance and position queries

**REQ-FNC-043:** The system SHALL handle exchange-specific characteristics:
- Price and quantity precision rules per trading pair
- Minimum and maximum order sizes
- Rate limiting and API quotas
- Exchange-specific error codes
- Trading rules and restrictions

**REQ-FNC-044:** The system SHALL synchronize exchange account data with Fastback.jl portfolio management.

#### 7.5.1 Binance Integration Example

**REQ-FNC-045:** WHERE Binance connectivity is required THEN the system SHALL use Binance.jl (https://github.com/rzhli/Binance.jl) as the Binance implementation.

**REQ-FNC-046:** WHERE Binance is configured THEN the system SHALL support:
- API key and secret authentication
- Testnet environment option for testing
- Credential validation before trading initialization

**REQ-FNC-047:** WHERE Binance WebSocket is used THEN the system SHALL support:
- Ticker streams for price updates
- Kline (candlestick) streams
- Order book (depth) streams
- Automatic reconnection on disconnection

### 7.6 Reactive Pipeline

**REQ-FNC-048:** The system SHALL implement a reactive pipeline using Rocket.jl with chainable observables:
- Market Data → Strategy Processing → Signal Generation → Order Execution → Metrics Update

**REQ-FNC-049:** The system SHALL handle errors centrally in the reactive pipeline without stopping the entire flow.

**REQ-FNC-050:** The system SHALL allow addition of custom reactive operators for pipeline extension.

**REQ-FNC-051:** The system SHALL maintain backward compatibility when adding new pipeline components.

### 7.7 Backtesting Engine

**REQ-FNC-052:** The system SHALL provide a BacktestEngine that:
- Loads historical data from configured sources
- Replays market events chronologically
- Simulates order execution with realistic fills
- Applies commission and slippage models
- Generates comprehensive performance reports

**REQ-FNC-053:** The system SHALL support backtest configuration:
- Start and end date selection
- Initial capital specification
- Commission rates
- Slippage models
- Strategy parameter sets

**REQ-FNC-054:** The system SHALL produce backtest reports including:
- Final equity and returns
- Complete trade log
- Performance metrics (all categories from REQ-FNC-022 through REQ-FNC-026)
- Equity curve visualization
- Drawdown chart

**REQ-FNC-055:** The system SHALL support parameter optimization:
- Grid search over parameter ranges
- Random sampling for large parameter spaces
- Bayesian optimization (optional)
- Walk-forward analysis
- In-sample vs out-of-sample metrics

**REQ-FNC-061:** The system SHALL allow specification of optimization objectives:
- Maximize Sharpe Ratio (default)
- Maximize returns
- Minimize drawdown
- Maximize profit factor
- Custom objective functions

### 7.9 Web Interface - Genie.jl

#### 7.9.1 Navigation and Home

**REQ-FNC-062:** The Web Interface SHALL provide a home page with:
- Navigation menu accessible from all pages
- Links to GitHub repository
- Links to project documentation
- Project overview information

**REQ-FNC-063:** The Web Interface SHALL provide consistent navigation across:
- Home
- Strategies
- Import Candles
- Manage Candles
- Backtest
- Optimization History
- Live Trading
- Settings

#### 7.9.2 Strategy Management

**REQ-FNC-064:** The Web Interface SHALL display on the Strategies page:
- List of all available trading strategies
- Strategy descriptions
- Configurable parameters for each strategy
- Default parameter values

**REQ-FNC-065:** The Web Interface SHALL retrieve strategy information from the Reactive Trading System backend.

#### 7.8.3 Data Import and Management

**REQ-FNC-061:** The Web Interface SHALL provide on the Import Candles page:
- Exchange selection dropdown (supporting multiple exchanges including Binance Spot, Kraken, and others)
- Symbol input field for trading pair
- Start date picker
- End date picker with default value of yesterday at 23:59:59
- Import submission button

**REQ-FNC-062:** The Web Interface SHALL display import progress and status during data download.

**REQ-FNC-063:** The Web Interface SHALL provide on the Manage Candles page:
- Table with columns: Exchange, Symbol, Start Date, End Date, Actions
- Update action button for modifying date ranges
- Delete action button with confirmation prompt
- Show action button for visualization

**REQ-FNC-064:** The Web Interface SHALL display candle data visualization using LightweightCharts.jl:
- Interactive OHLCV candlestick charts
- Zoom and pan controls
- Formatted time axis
- Scaled price axis

#### 7.8.4 Backtesting Interface

**REQ-FNC-065:** The Web Interface SHALL provide on the Backtest page:
- Exchange selection for data source
- Symbol selection for trading pair
- Start and end date pickers
- Strategy selection dropdown
- Parameter input fields specific to selected strategy
- Backtest start button

**REQ-FNC-066:** WHILE a backtest is running, the Web Interface SHALL display:
- Real-time trade activity
- Real-time performance metrics updates
- Progress indicator

**REQ-FNC-067:** WHEN a backtest completes, the Web Interface SHALL display:
- Final results and statistics
- Complete performance metrics
- Equity curve chart
- Export to JSON button

**REQ-FNC-068:** The Web Interface SHALL generate JSON exports containing:
- Trade activity log
- Performance metrics
- Backtest configuration parameters
- Timestamped filename

#### 7.8.5 Optimization Interface

**REQ-FNC-069:** The Web Interface SHALL provide on the Backtest page optimization features:
- Parameter range specification inputs
- Training/testing data split configuration
- Walk-forward analysis options
- Optimization metric selection (default: maximize Sharpe Ratio)
- Optimization start button

**REQ-FNC-070:** WHILE optimization is running, the Web Interface SHALL:
- Display progress information
- Allow navigation without interrupting the optimization
- Update progress metrics periodically

**REQ-FNC-071:** WHEN optimization completes, the Web Interface SHALL display:
- Best parameter configuration
- Performance comparison table of all tested combinations
- Visualization of parameter space results
- Export option for optimization results

**REQ-FNC-072:** The Web Interface SHALL provide an Optimization History page:
- List of all optimization runs with metadata
- Start time, duration, and status for each run
- Ability to view full results of past optimizations
- Ability to resume viewing in-progress optimizations

#### 7.8.6 Live Trading Interface

**REQ-FNC-073:** The Web Interface SHALL provide on the Live page:
- Mode selection: Paper Trading or Live Trading
- Strategy selection dropdown
- Parameter configuration inputs
- Exchange API credential inputs (for live trading only)
- Start and Stop trading buttons

**REQ-FNC-074:** The Web Interface SHALL display clear warnings and require confirmation before starting live trading.

**REQ-FNC-075:** WHILE trading is active (paper or live), the Web Interface SHALL display:
- Current positions
- Real-time trade activity log
- Real-time performance metrics
- P&L updates

**REQ-FNC-076:** The Web Interface SHALL provide notification driver configuration:
- Selection between Telegram and Pushover
- Credential input fields
- Test notification button
- Event type configuration (which events trigger notifications)

**REQ-FNC-077:** WHEN live trading is active with configured notifications, the Web Interface SHALL send alerts for:
- Trade executions
- Risk limit breaches
- Strategy state changes
- System errors

#### 7.8.7 Settings and Configuration

**REQ-FNC-078:** The Web Interface SHALL provide a Settings page for:
- Default exchange preferences
- Default date range configurations
- Display preferences (themes, chart settings)
- Risk management parameter configuration
- Notification preferences

**REQ-FNC-079:** The Web Interface SHALL persist settings across sessions.

**REQ-FNC-080:** The Web Interface SHALL apply setting changes immediately or after user confirmation.

**REQ-FNC-081:** The Web Interface SHALL provide a reset to defaults option for all settings.

### 7.9 Notification System

**REQ-FNC-082:** The system SHALL support Telegram notifications:
- Bot token configuration
- Chat ID configuration
- Message formatting
- Emoji support for visual alerts

**REQ-FNC-083:** The system SHALL support Pushover notifications:
- User key configuration
- API token configuration
- Priority levels
- Sound selection

**REQ-FNC-084:** The system SHALL send notifications for:
- Trade executions (entry and exit)
- Risk alerts (drawdown, daily loss)
- System errors and connection issues
- Strategy state changes
- Performance milestones

**REQ-FNC-085:** The system SHALL allow granular notification configuration:
- Enable/disable per event type
- Priority levels per event type
- Custom message templates

### 7.10 Error Handling and Logging

**REQ-FNC-086:** The system SHALL capture errors centrally and log them without stopping the reactive pipeline.

**REQ-FNC-087:** The system SHALL log with structured logging including:
- Timestamp (Timestamps64)
- Log level (DEBUG, INFO, WARN, ERROR, CRITICAL)
- Component identifier
- Message
- Stack trace for errors

**REQ-FNC-088:** The system SHALL provide log filtering and search capabilities in the Web Interface.

**REQ-FNC-089:** The system SHALL save system state on critical exceptions to enable clean recovery.

**REQ-FNC-090:** The system SHALL trigger automatic diagnostics when metrics indicate system anomalies.

### 7.11 Data Persistence

**REQ-FNC-091:** The system SHALL persist:
- Strategy configurations
- Backtest results
- Optimization history
- Trade history
- Performance metrics
- User settings
- Imported candle data

**REQ-FNC-092:** The system SHALL use appropriate storage mechanisms:
- Time-series database for OHLCV data
- Relational database for configuration and metadata
- File system for backtest exports and reports

**REQ-FNC-093:** The system SHALL provide data retention policies:
- Configurable history retention periods
- Automatic archival of old data
- Data compression for historical records

---

## 8. NON-FUNCTIONAL REQUIREMENTS

### 8.1 Performance

**REQ-NFN-001:** The system SHALL support minimum 1000 strategies in simultaneous backtesting.

**REQ-NFN-002:** The system SHALL process minimum 10,000 ticks per second in live mode.

**REQ-NFN-003:** The system SHALL use maximum 4GB RAM in standard configuration.

**REQ-NFN-004:** A backtest over 5 years of daily data SHALL complete in less than 10 seconds.

**REQ-NFN-005:** The Web Interface SHALL respond to user actions within 200ms.

**REQ-NFN-006:** Large CSV file imports SHALL not block the Web Interface during processing.

### 8.2 Scalability

**REQ-NFN-007:** The system SHALL support distributed deployment for parallel backtesting across multiple machines.

**REQ-NFN-008:** The system SHALL support horizontal scaling to manage additional brokers and markets simultaneously.

**REQ-NFN-009:** The system SHALL handle strategy portfolios with 100+ concurrent strategies.

### 8.3 Maintainability

**REQ-NFN-010:** The codebase SHALL achieve minimum 80% test coverage with unit tests.

**REQ-NFN-011:** The system SHALL provide complete API documentation with usage examples.

**REQ-NFN-012:** The system SHALL use semantic versioning (MAJOR.MINOR.PATCH).

**REQ-NFN-013:** Code SHALL follow Julia style guidelines and best practices.

**REQ-NFN-014:** The system SHALL provide example implementations for all major features.

### 8.4 Compatibility

**REQ-NFN-015:** The system SHALL run on Julia 1.9+ on Linux, macOS, and Windows.

**REQ-NFN-016:** The system SHALL support a pluggable exchange architecture allowing integration with:
- Binance (mandatory, via Binance.jl as reference implementation)
- Kraken (optional)
- Coinbase (optional)
- Interactive Brokers (optional)
- Other exchanges through standardized interface implementation

**REQ-NFN-017:** The Web Interface SHALL support modern browsers:
- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+

### 8.5 Usability

**REQ-NFN-018:** The Web Interface SHALL provide intuitive navigation without requiring documentation for basic tasks.

**REQ-NFN-019:** Error messages SHALL be clear, actionable, and user-friendly.

**REQ-NFN-020:** The system SHALL provide contextual help and tooltips in the Web Interface.

**REQ-NFN-021:** The Web Interface SHALL support responsive design for tablet and desktop viewing.

### 8.6 Security

**REQ-NFN-022:** API credentials SHALL never be logged in plain text.

**REQ-NFN-023:** The system SHALL use HTTPS for all external communications.

**REQ-NFN-024:** The Web Interface SHALL implement CSRF protection.

**REQ-NFN-025:** Session tokens SHALL expire after configurable timeout periods.

**REQ-NFN-026:** The system SHALL rate-limit API requests to prevent abuse.

---

## 9. REQUIREMENTS TRACEABILITY MATRIX

### 9.1 Requirements Summary by Category

| Category | Count | Priority |
|----------|-------|----------|
| Ubiquitous | 16 | CRITICAL |
| Event-Driven | 30 | HIGH |
| Unwanted | 18 | HIGH |
| State-Driven | 15 | MEDIUM |
| Optional | 20 | LOW |
| Complex | 14 | MEDIUM |
| Functional | 93 | HIGH |
| Non-Functional | 26 | MEDIUM |
| **TOTAL** | **232** | - |

### 9.2 Requirements by Module

| Module | Requirements | Priority |
|--------|--------------|----------|
| Core Trading System | REQ-FNC-001 to REQ-FNC-016 | CRITICAL |
| Technical Indicators | REQ-FNC-017 to REQ-FNC-020 | HIGH |
| Time Resampling | REQ-FNC-021 to REQ-FNC-025 | HIGH |
| Performance Analytics | REQ-FNC-026 to REQ-FNC-035 | HIGH |
| Risk Management | REQ-FNC-036 to REQ-FNC-039 | CRITICAL |
| Exchange Integration | REQ-FNC-040 to REQ-FNC-047 | HIGH |
| Reactive Pipeline | REQ-FNC-048 to REQ-FNC-051 | CRITICAL |
| Backtesting Engine | REQ-FNC-052 to REQ-FNC-056 | HIGH |
| Web Interface | REQ-FNC-057 to REQ-FNC-081 | MEDIUM |
| Notification System | REQ-FNC-082 to REQ-FNC-085 | MEDIUM |
| Error Handling | REQ-FNC-086 to REQ-FNC-090 | HIGH |
| Data Persistence | REQ-FNC-091 to REQ-FNC-093 | MEDIUM |

### 9.3 Dependencies Between Requirements

#### Critical Dependencies

1. **REQ-UBI-013** (Reactive architecture) is prerequisite for:
   - REQ-FNC-048 to REQ-FNC-051 (Reactive pipeline)
   - All event-driven requirements (REQ-EVT-*)

2. **REQ-FNC-001 to REQ-FNC-006** (Strategy framework) are prerequisites for:
   - REQ-FNC-007 to REQ-FNC-009 (Example strategies)
   - REQ-FNC-055 (Parameter optimization)
   - REQ-FNC-059 to REQ-FNC-060 (Web strategy management)

3. **REQ-FNC-040 to REQ-FNC-044** (Exchange integration abstraction) are prerequisites for:
   - REQ-FNC-045 to REQ-FNC-047 (Binance example implementation)
   - REQ-FNC-073 to REQ-FNC-077 (Live trading interface)
   - REQ-CPX-013 to REQ-CPX-014 (Complex exchange scenarios)
   - Future exchange integrations (Kraken, Coinbase, etc.)

4. **REQ-FNC-026 to REQ-FNC-035** (Analytics) are prerequisites for:
   - All reporting and dashboard features
   - REQ-FNC-066 to REQ-FNC-068 (Backtest results display)
   - REQ-FNC-075 (Live trading metrics display)

5. **REQ-FNC-021 to REQ-FNC-025** (OnlineResamplers.jl) are prerequisites for:
   - Multi-timeframe strategy development
   - Efficient OHLCV data aggregation
   - REQ-FNC-017 to REQ-FNC-020 (Technical indicators on resampled data)

---

## 10. GLOSSARY

### General Terms

- **Reactive Programming**: Programming paradigm focused on asynchronous data streams and event propagation
- **Observable**: Data stream that can be subscribed to for receiving updates
- **Pipeline**: Chain of processing steps from data ingestion to execution
- **Streaming Computation**: Incremental calculation on data as it arrives, without reprocessing history

### Trading Terms

- **Tick**: Single update of market data (price, volume, timestamp)
- **Candle/OHLCV**: Aggregated market data showing Open, High, Low, Close prices and Volume over a time period
- **Signal**: Trading decision indicator (BUY, SELL, HOLD) generated by a strategy
- **Slippage**: Difference between expected execution price and actual execution price
- **Drawdown**: Peak-to-trough decline in portfolio value, expressed as percentage
- **Equity Curve**: Graph showing portfolio value evolution over time
- **Fill Ratio**: Proportion of an order that gets executed
- **Commission**: Transaction fee charged by broker or exchange
- **Position**: Current holding in a specific asset
- **Exposure**: Total capital at risk in open positions
- **P&L (Profit and Loss)**: Net profit or loss from trading activity

### Strategy Terms

- **Backtesting**: Testing a strategy on historical data to evaluate performance
- **Paper Trading**: Simulated trading with real-time data but virtual capital
- **Live Trading**: Real trading with actual capital on an exchange
- **Walk-Forward Analysis**: Optimization technique with rolling training and testing windows
- **Parameter Optimization**: Process of finding optimal strategy configuration
- **Grid Search**: Exhaustive search over specified parameter ranges
- **Bayesian Optimization**: Intelligent search using probabilistic models to find optimal parameters
- **Golden Cross**: Technical pattern when fast MA crosses above slow MA (bullish signal)
- **Death Cross**: Technical pattern when fast MA crosses below slow MA (bearish signal)

### Performance Metrics

- **Sharpe Ratio**: Risk-adjusted return metric (return/risk ratio)
- **PSR (Probabilistic Sharpe Ratio)**: Statistical confidence that Sharpe Ratio is positive
- **Sortino Ratio**: Modified Sharpe Ratio considering only downside volatility
- **Calmar Ratio**: Return/maximum drawdown ratio
- **Alpha**: Excess return compared to benchmark
- **Beta**: Sensitivity to market movements
- **Information Ratio**: Alpha/tracking error ratio
- **Treynor Ratio**: Return/beta ratio
- **VaR (Value at Risk)**: Maximum expected loss at given confidence level
- **MAE (Maximum Adverse Excursion)**: Worst unrealized loss during a winning trade
- **MFE (Maximum Favorable Excursion)**: Best unrealized profit during a losing trade
- **Win Rate**: Percentage of profitable trades
- **Profit Factor**: Gross profit/gross loss ratio
- **Expectancy**: Average expected profit per trade

### Technical Terms

- **WebSocket**: Protocol for full-duplex real-time communication
- **REST API**: HTTP-based interface for programmatic interaction
- **Order Book**: List of buy and sell orders at different price levels
- **Liquidity**: Ease of buying/selling without significant price impact
- **Rate Limit**: Maximum number of API requests allowed in a time period
- **Testnet**: Sandbox environment for testing without real capital
- **Exchange**: Trading platform (e.g., Binance, Kraken, Coinbase)
- **Exchange Connector**: Pluggable module implementing exchange-specific integration
- **Symbol/Trading Pair**: Asset pair being traded (e.g., BTC/USDT)
- **Abstract Exchange Interface**: Standardized API that all exchange connectors must implement

### Technology Stack Terms

- **Julia**: High-performance programming language for technical computing
- **Rocket.jl**: Reactive programming library for Julia
- **OnlineStatsBase.jl**: Library for streaming/online statistical calculations
- **Fastback.jl**: Trading engine for account and order management
- **Binance.jl**: Julia interface to Binance exchange API
- **OnlineTechnicalIndicators.jl**: Streaming technical indicator calculations
- **OnlinePortfolioAnalytics.jl**: Portfolio performance metrics library
- **OnlineResamplers.jl**: Streaming time-based data aggregation and resampling (https://github.com/femtotrader/OnlineResamplers.jl)
- **Timestamps64**: Precise timestamp representation
- **Genie.jl**: Web framework for Julia
- **LightweightCharts.jl**: Lightweight charting library integration

### Web Interface Terms

- **Dashboard**: Main overview page showing key metrics
- **Notification Driver**: Service for sending alerts (Telegram, Pushover)
- **CSRF Protection**: Security against cross-site request forgery attacks
- **Session Token**: Authentication token for maintaining user sessions
- **Responsive Design**: Interface that adapts to different screen sizes

---

## 11. DOCUMENT APPROVAL

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2025-11-13 | Requirements Team | Initial requirements document with merged specs from reactive trading system, web interface, and CSV data source. Added OnlineResamplers.jl for time-based resampling. |

### Approval Signatures

| Role | Name | Signature | Date |
|------|------|-----------|------|
| **Product Owner** | _______________ | _______________ | ________ |
| **Technical Lead** | _______________ | _______________ | ________ |
| **QA Lead** | _______________ | _______________ | ________ |
| **Stakeholder** | _______________ | _______________ | ________ |

---

## 12. APPENDIX: EARS PATTERN EXAMPLES

### Ubiquitous Example
> REQ-UBI-001: The system SHALL process market data updates with a maximum latency of 100ms.

### Event-Driven Example
> REQ-EVT-006: WHEN CSV data is loaded THEN the system SHALL parse it into MarketData objects with proper timestamp conversion.

### Unwanted Example
> REQ-UNW-008: IF cumulative loss exceeds 5% of initial capital in one day THEN the system SHALL close all positions and stop trading.

### State-Driven Example
> REQ-STA-001: WHILE the system is in "backtesting" mode THEN the system SHALL use historical data and simulate execution without broker connection.

### Optional Example
> REQ-OPT-001: WHERE margin trading is configured THEN the system SHALL support trading on margin with capital requirement verification.

### Complex Example
> REQ-CPX-001: WHILE the system is in "live trading" mode, WHEN an entry signal is generated, IF market conditions are favorable (spread < 0.1%, liquidity > threshold) THEN the system SHALL place a limit order at the best available price.

---

**END OF DOCUMENT**

---

**Document Control Information:**
- **Classification:** Internal Use
- **Distribution:** Development Team, Stakeholders, QA Team
- **Next Review Date:** 2026-02-12
- **Maintenance:** Requirements Team maintains this living document
- **Related Documents:**
  - Reactive Trading System Technical Specification
  - Genie.jl Web Interface Design Document
  - CSV Data Source Implementation Guide
  - Exchange Connector Interface Specification
  - Binance.jl Integration Guide (Reference Implementation)
