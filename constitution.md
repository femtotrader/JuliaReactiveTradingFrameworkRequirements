<!--
  SYNC IMPACT REPORT
  ===================
  Version change: 1.7.1 → 1.8.0
  Bump rationale: MINOR - Added new Time Management principle (IX) for Clock abstraction

  Modified principles:
    - None modified

  Added sections:
    - Principle IX: Time Management and Clock Abstraction
    - Technology Stack: Added Clock types entry
    - Module Organization: Added Clock.jl to Core module

  Removed sections: None

  Templates requiring updates:
    - .specify/templates/plan-template.md - ✅ Compatible (no changes needed)
    - .specify/templates/spec-template.md - ✅ Compatible (no changes needed)
    - .specify/templates/tasks-template.md - ✅ Compatible (no changes needed)

  Follow-up TODOs: None
-->

# Reactive Trading System Constitution

## Product Vision

A modular algorithmic trading platform built with Julia that combines reactive programming
with online statistics for real-time market analysis and automated trading execution.

### Core Capabilities

- **Automated Trading**: Execute algorithmic strategies in paper trading, testnet, and
  live trading modes
- **Real-time Analytics**: Stream processing of market data with online statistical analysis
- **Risk Management**: Configurable position limits, drawdown controls, and daily loss
  thresholds
- **Performance Tracking**: Comprehensive metrics including Sharpe Ratio, PSR, Sortino
  Ratio, Alpha, Beta, and rolling statistics
- **Exchange Integration**: Native Binance support via WebSocket and REST API with
  testnet environment
- **Strategy Framework**: Extensible architecture for developing and testing custom
  trading strategies
- **LLM Integration**: Natural language access via Model Context Protocol (MCP) for
  running backtests, querying results, and managing data

### Target Users

Algorithmic traders and quantitative developers who need a robust, modular platform for
developing, testing, and deploying automated trading strategies with professional-grade
risk management and performance analytics.

## Core Principles

### I. Reactive Architecture

The system MUST implement a reactive programming model using Rocket.jl observables as
the central communication backbone.

- All market data, trading signals, and system events MUST flow through observable
  pipelines
- Components MUST process data asynchronously without blocking other pipeline stages
- Error handling MUST be centralized; individual component failures MUST NOT stop
  the entire reactive pipeline
- Signal propagation MUST maintain temporal ordering using Timestamps64 for
  nanosecond precision

**Rationale**: Reactive architecture enables decoupled, scalable trading systems where
data producers and consumers operate independently, supporting real-time market
responsiveness.

### II. Strategy Interface Contract

All trading strategies MUST implement the `AbstractTradingStrategy` interface
inheriting from `OnlineStatsBase.OnlineStat`.

- Strategies MUST implement `OnlineStatsBase._fit!(strategy, data::MarketData)` for
  streaming data processing (public `fit!` is provided automatically by OnlineStatsBase)
- Strategies MUST store their output in a `value::StrategyValue` field, accessible via
  `OnlineStatsBase.value(strategy)` which returns the StrategyValue containing:
  - `signal::TradingSignal`: The most recent trading signal
  - `metrics::StrategyMetrics`: Performance and trading metrics
- Signal access MUST use the canonical pattern: `value(strategy).signal`
- The `_fit!` method MUST NOT return any value (OnlineStatsBase convention)
- The `_fit!` method MUST only generate signals; metrics MUST be updated externally
  by the backtest/execution engine (not inside `_fit!`)
- Trading signals MUST be one of: `SignalType.BUY`, `SignalType.SELL`, or `SignalType.HOLD`
- Each signal MUST include: action, quantity, confidence level (0.0 to 1.0), and timestamp
- Configurable parameters MUST be stored in a `params::Dict{String, Any}` field

**Rationale**: Following OnlineStatsBase conventions enables strategy composition with
ecosystem tools (Series, Group, merge!) and maintains consistency with Julia statistics
packages. Separating signal generation from metrics tracking ensures accurate performance
measurement based on actual execution rather than signal generation.

### III. Streaming-First Processing

All data processing MUST use streaming/online algorithms that process data
incrementally without requiring full dataset storage.

- Technical indicators MUST use OnlineTechnicalIndicators.jl for streaming computation
- Statistical calculations MUST use OnlineStatsBase.jl compatible implementations
- Memory usage MUST remain bounded regardless of data stream length
- State updates MUST be O(1) per observation (constant time)

**Rationale**: Streaming processing is essential for real-time trading where latency
matters and unbounded data streams cannot be stored in memory.

### IV. Portfolio Analytics Integration

Performance metrics MUST be calculated using OnlinePortfolioAnalytics.jl in real-time.

- The system MUST compute: Sharpe Ratio, Sortino Ratio, Maximum Drawdown, PSR
- The system MUST compute: Alpha, Beta, Information Ratio, Treynor Ratio
- Metrics MUST update incrementally with each new observation
- Drawdown calculations MUST support both absolute and percentage modes
- Equity curves and metric collectors MUST be configurable (sampling period, mode)

**Rationale**: Real-time portfolio analytics enable traders to monitor strategy
performance and risk without post-processing delays.

### V. Exchange Integration Standards

Exchange integrations MUST provide unified interfaces for data retrieval and
order execution.

- Data sources MUST implement `AbstractDataSource` interface
- Execution engines MUST implement `AbstractExecutionEngine` interface
- Account management MUST use Fastback.jl for position tracking and P&L
- Supported exchanges: Binance Spot (via Binance.jl), Kraken
- Credentials MUST NOT be stored in code; environment variables or secure
  configuration required
- Order execution MUST support: paper trading, backtesting, and live trading modes

**Rationale**: Unified interfaces enable strategy portability across exchanges and
execution modes without code changes.

### VI. Web Interface Separation

The Genie.jl web interface MUST remain decoupled from the core trading system.

- Web interface MUST communicate with trading system via defined service APIs
- Background operations (optimization, trading) MUST NOT block web request handling
- Real-time updates MUST use WebSocket connections for live data
- Chart rendering MUST use LightweightCharts.jl for OHLCV visualization
- Notification drivers (Telegram, Pushover) MUST be pluggable and configurable
- User settings and candle data MUST persist across sessions

**Rationale**: Separation enables headless operation, API access, and alternative
frontends while maintaining a responsive user interface.

### VII. Testing and Quality Assurance

All components MUST be independently testable with clear boundaries.

- Strategies MUST be testable with synthetic data streams
- Backtesting MUST produce reproducible results given the same seed and data
- Example strategies (Random Trading, MA Crossover) MUST serve as reference
  implementations demonstrating correct interface usage
- Integration tests MUST verify end-to-end reactive pipeline behavior
- Walk-forward analysis MUST be supported for strategy optimization validation

**Rationale**: Trading systems require high reliability; comprehensive testing
prevents costly errors in live trading.

### VIII. MCP Integration Standards

The system MUST expose functionality via Model Context Protocol (MCP) for
LLM-based natural language interaction.

- MCP tools MUST wrap existing service layer functions (no duplicate business logic)
- Tool handlers MUST be stateless; the LLM client maintains conversation context
- Long-running operations (backtests, imports) MUST return immediately with a job ID
  and provide status polling via separate tools
- Backtest results MUST be returned in TOON format to minimize token consumption
- Result granularity MUST be configurable (compact summary vs full data)
- MCP server MUST operate independently of the Genie.jl web interface
- Tool schemas MUST include clear descriptions and parameter documentation
- Live trading execution via MCP MUST be prohibited (safety constraint)

**Rationale**: MCP enables natural language access to trading system functionality
while maintaining separation from the web interface and ensuring safe operation
through stateless, read-mostly tool design.

### IX. Time Management and Clock Abstraction

The system MUST use a unified Clock abstraction for time management across all
trading modes (live, paper, backtest).

- All time-dependent components MUST use the `AbstractClock` interface for time queries
- `RealTimeClock` MUST be used for live and paper trading modes, returning actual
  system UTC time via `now(Timestamp64, UTC)` from Timestamps64
- `SimulationClock` MUST be used for backtesting, with time advanced by the backtest
  engine as it processes historical data
- All time values MUST be in UTC for consistency across exchanges and markets
- Risk management daily resets MUST use `today_utc(clock)` to correctly track trading
  days in both real-time and simulated modes
- The backtest engine MUST call `advance!(clock, timestamp)` for each market data point
  to ensure all time-dependent calculations use the simulated date
- Components that depend on time (RiskManager, MetricsCollector) MUST accept a clock
  parameter and default to `DEFAULT_CLOCK` (RealTimeClock) when not provided

**Rationale**: A shared Clock abstraction ensures that time-dependent logic (daily
P&L resets, drawdown calculations, timestamp comparisons) works correctly in both
real-time and backtesting modes. Without this abstraction, components using `today()`
would see the actual system date during backtests, causing incorrect daily limit
tracking across multi-year historical simulations.

## Technology Stack

The following technology choices are NON-NEGOTIABLE for this project:

| Component | Technology | Purpose |
|-----------|------------|---------|
| Language | Julia | High-performance numerical computing |
| Reactive Framework | Rocket.jl | Observable-based reactive programming |
| Online Statistics | OnlineStatsBase.jl | Streaming statistical interface |
| Technical Indicators | OnlineTechnicalIndicators.jl | Streaming TA calculations |
| Portfolio Analytics | OnlinePortfolioAnalytics.jl | Real-time performance metrics |
| Account Management | Fastback.jl | Position tracking, P&L, order execution |
| Exchange Integration | Binance.jl | Binance Spot market access |
| Timestamps | Timestamps64 | Nanosecond precision time handling |
| Web Framework | Genie.jl | Web interface and API |
| Charting | LightweightCharts.jl | Interactive OHLCV visualization |
| LLM Integration | ModelContextProtocol.jl | MCP server for natural language access |

**Core Types**:

| Type | Module | Purpose |
|------|--------|---------|
| `AbstractClock` | Core | Base type for clock implementations |
| `RealTimeClock` | Core | Real UTC time for live/paper trading |
| `SimulationClock` | Core | Simulated time for backtesting |

**Constraints**:
- Julia version: 1.12+ required
- All dependencies MUST be registered Julia packages or documented local packages
- Thread safety MUST be considered for all shared state

## Development Workflow

### Module Organization

The codebase follows this module structure:

```text
src/
├── Core/                      # Level 0: Foundational types (no dependencies)
│   ├── Core.jl                # Module definition
│   ├── Clock.jl               # AbstractClock, RealTimeClock, SimulationClock
│   ├── MarketData.jl          # TradingPair, MarketData, TradingSignal, market_timestamp
│   ├── ErrorHandling.jl       # Error types, CircuitBreaker
│   └── ReactiveCore.jl        # Rocket.jl reactive operators
├── Strategies/                # Level 1: Strategy framework (depends on Core)
│   ├── Strategies.jl          # Module definition
│   ├── AbstractStrategy.jl    # AbstractTradingStrategy interface
│   ├── StrategyValue.jl       # OnlineStatsBase value() output
│   ├── StrategyMetrics.jl     # Performance & trading metrics
│   ├── StrategyLoader.jl      # User strategy discovery
│   └── builtin/               # Builtin strategies
│       ├── RandomStrategy/
│       ├── MACrossover/
│       └── BuyAndHold/
├── DataSources/               # Level 1: Data sources (depends on Core)
│   ├── DataSources.jl         # Module definition
│   ├── AbstractDataSource.jl
│   ├── BinanceDataSource.jl
│   ├── CSVDataSource.jl
│   └── SimulatedDataSource.jl
├── Logging/                   # Level 1: Strategy logging (depends on Core)
│   ├── StrategyLogging.jl     # Module definition (renamed to avoid stdlib conflict)
│   ├── LogEntry.jl
│   ├── StrategyLogger.jl
│   └── WebSocketLogger.jl
├── Execution/                 # Level 2: Trade execution (depends on Core, Strategies)
│   ├── Execution.jl           # Module definition
│   ├── AbstractEngine.jl
│   ├── PaperTradingEngine.jl
│   └── LiveTradingEngine.jl
├── Backtesting/               # Level 3: Backtesting (depends on all above)
│   ├── Backtesting.jl         # Module definition
│   └── BacktestEngine.jl
├── MCP/                       # Level 3: LLM Integration (depends on all above)
│   ├── MCP.jl                 # Module definition
│   ├── ResponseHelpers.jl     # success_response, error_response utilities
│   └── tools/                 # MCP tool handlers
│       ├── BacktestTools.jl   # run_backtest, get_backtest_status, get_backtest_results, etc.
│       ├── StrategyTools.jl   # list_strategies, get_strategy_params
│       ├── DataTools.jl       # list_datasets, import_candles, get_import_status
│       ├── ExportTools.jl     # export_backtest (JSON/TOON)
│       └── QueryTools.jl      # search_backtests
├── risk/                      # Risk management (not yet modularized)
│   └── RiskManager.jl         # Uses Clock for daily reset tracking
├── portfolio/                 # Portfolio management (not yet modularized)
├── cli/                       # CLI commands (not yet modularized)
└── ReactiveTradingSystem.jl   # Root module

bin/
└── mcp_server.jl              # MCP server entry point (stdio transport)

tests/
├── strategies_test.jl         # Strategy unit tests
├── metrics_test.jl            # Metrics calculation tests
├── integration_test.jl        # End-to-end pipeline tests
└── mcp_test.jl                # MCP tool handler tests
```

### Common Commands

```julia
# Project Setup
using Pkg
Pkg.activate(".")
Pkg.instantiate()

# Running Tests
Pkg.test()                              # Run all tests
include("test/strategies_test.jl")      # Run specific test file

# Development Workflow
julia --project=.                       # Start REPL with project
include("src/TradingSystem.jl")
using .TradingSystem
```

### RTS CLI Tool

```bash
./bin/rts strategy list                 # List all strategies (builtin + user)
./bin/rts strategy new MyStrategyName   # Create new user strategy scaffold
./bin/rts --help                        # Show CLI help
```

### MCP Server

```bash
# Start MCP server (used by Claude Desktop)
julia --project=. bin/mcp_server.jl
```

### Implementation Standards

1. **Incremental Development**: Features MUST be implemented in testable increments
   following user story priorities (P1 before P2 before P3)

2. **Interface-First Design**: Define abstract types and method signatures before
   implementing concrete types

3. **Documentation**: Public functions MUST include docstrings with:
   - Brief description
   - Arguments and return types
   - Example usage for non-trivial functions

4. **Error Handling**:
   - Use Julia's exception system for exceptional conditions
   - Reactive pipelines MUST catch and log errors without propagating crashes
   - User-facing errors MUST provide actionable messages

5. **DateTime Formatting Standard**:
   - All datetime display MUST use ISO 8601 format: `yyyy-mm-ddTHH:MM:SS.sss`
   - The 'T' separator MUST be used between date and time (not space)
   - Millisecond precision MUST be included for trading data
   - Julia format string: `dateformat"yyyy-mm-ddTHH:MM:SS.sss"`
   - This format ensures cross-platform compatibility (JavaScript Date parsing)

6. **Time Management**:
   - All time-dependent components MUST use Clock abstraction
   - Never use `today()` or `now()` directly; use `today_utc(clock)` or `now_utc(clock)`
   - Components MUST accept optional `clock` parameter defaulting to `DEFAULT_CLOCK`
   - Backtesting code MUST create `SimulationClock` and advance it with data timestamps

### Enum Definition Pattern

All enums MUST be wrapped in a module to avoid naming collisions:

```julia
module EnumName
    @enum T VALUE1 VALUE2 VALUE3
end
```

**Usage**:
- Qualified access: `EnumName.VALUE1`
- Type annotation: `field::EnumName.T`

**Rationale**: Module-wrapped enums prevent naming collisions (e.g., `BUY` vs `BUY_SIDE`),
provide namespace clarity, and enable better IDE autocomplete.

### File Naming Conventions

- **PascalCase** for module files: `ReactiveCore.jl`, `PortfolioManager.jl`
- **lowercase** for type files in Core: `Clock.jl` (contains types, not a module)
- **snake_case** for test files: `strategies_test.jl`, `integration_test.jl`
- **lowercase** for example files: `random_strategy.jl`, `backtest_example.jl`

### Requirements Syntax (EARS)

This project uses **EARS (Easy Approach to Requirements Syntax)** for writing requirements.
All requirements in spec.md files MUST follow one of the five EARS patterns:

| Pattern | Syntax | Use Case |
|---------|--------|----------|
| Ubiquitous | The system SHALL \<action\> | Always-active behavior |
| Event-driven | WHEN \<trigger\>, the system SHALL \<action\> | Response to events |
| State-driven | WHILE \<state\>, the system SHALL \<action\> | Behavior during a state |
| Unwanted | IF \<condition\>, THEN the system SHALL \<action\> | Error handling |
| Optional | WHERE \<feature\>, the system SHALL \<action\> | Conditional features |

**Keywords**:
- **SHALL** - Mandatory requirement (MUST be implemented)
- **SHOULD** - Recommended (implement unless justified reason not to)
- **MAY** - Optional (implement at discretion)

See **Appendix B** for detailed examples and guidance.

### Code Review Gates

Before merging any PR, verify:

- [ ] New strategies implement full `AbstractTradingStrategy` contract
- [ ] Streaming algorithms maintain O(1) update complexity
- [ ] No credentials or secrets in code
- [ ] Reactive pipeline integration tested
- [ ] Web interface changes do not couple to trading core
- [ ] MCP tools wrap service layer (no duplicate business logic)
- [ ] Time-dependent code uses Clock abstraction (not raw `today()`/`now()`)

## Governance

This constitution establishes the foundational rules for the Reactive Trading System.
All development decisions MUST align with these principles.

### Amendment Process

1. Propose amendment via documented rationale
2. Assess impact on existing code and dependent templates
3. Update constitution version following semantic versioning:
   - MAJOR: Principle removal or backward-incompatible redefinition
   - MINOR: New principle or material guidance expansion
   - PATCH: Clarifications, wording improvements
4. Update dependent templates if principle changes affect their structure
5. Document changes in Sync Impact Report

### Compliance

- All PRs MUST reference relevant principles when touching core architecture
- Constitution violations MUST be justified in Complexity Tracking (plan.md)
- Quarterly review recommended to assess principle relevance

### Precedence

In case of conflict:
1. This Constitution (highest)
2. Feature Specifications (spec.md)
3. Implementation Plans (plan.md)
4. Task Lists (tasks.md)

## Appendix A: Kiro Document Reference

This project was initially developed using Kiro.dev before migrating to Github Spec Kit.
The original Kiro steering documents are preserved in `.kiro/` for historical reference:

| Document | Location | Content |
|----------|----------|---------|
| Product Overview | `.kiro/steering/product.md` | Project vision and capabilities |
| Technical Stack | `.kiro/steering/tech.md` | Dependencies and commands |
| Project Structure | `.kiro/steering/structure.md` | Module organization |
| Feature Specs | `.kiro/specs/` | Detailed requirements per feature |

These documents informed the principles in this constitution. When updating principles,
consult the original Kiro documents for additional context on design rationale.

## Appendix B: EARS Quick Reference

**EARS (Easy Approach to Requirements Syntax)** is a structured syntax for writing
unambiguous, testable requirements. Each pattern addresses a specific behavioral context.

### Pattern 1: Ubiquitous (Always Active)

**Syntax**: `The <system> SHALL <action>.`

No precondition - the behavior is always active.

**Examples for this project**:
- The system SHALL use Timestamps64 for all time values.
- The system SHALL maintain O(1) memory per observation.
- The system SHALL log all trading signals to the audit trail.
- The system SHALL use Clock abstraction for all time-dependent operations.

### Pattern 2: Event-Driven (Triggered by Event)

**Syntax**: `WHEN <trigger>, the <system> SHALL <action>.`

Behavior occurs in response to a specific event.

**Examples for this project**:
- WHEN market data arrives, the system SHALL invoke `_fit!` on active strategies.
- WHEN a strategy generates a BUY signal, the system SHALL validate against risk limits.
- WHEN the user requests a backtest, the system SHALL replay historical data.
- WHEN the backtest processes a data point, the system SHALL advance the SimulationClock.

### Pattern 3: State-Driven (Active During State)

**Syntax**: `WHILE <state>, the <system> SHALL <action>.`

Behavior is active only while the system is in a specific state.

**Examples for this project**:
- WHILE in paper trading mode, the system SHALL simulate order fills.
- WHILE connected to Binance WebSocket, the system SHALL process ticker updates.
- WHILE a drawdown exceeds 10%, the system SHALL reduce position sizes.
- WHILE in backtesting mode, the system SHALL use SimulationClock for time queries.

### Pattern 4: Unwanted Behavior (Error Handling)

**Syntax**: `IF <unwanted condition>, THEN the <system> SHALL <action>.`

Defines how the system handles errors, exceptions, or edge cases.

**Examples for this project**:
- IF the WebSocket connection drops, THEN the system SHALL attempt reconnection with
  exponential backoff.
- IF an order execution fails, THEN the system SHALL log the error and emit an alert.
- IF daily loss exceeds the limit, THEN the system SHALL halt trading until next day.

### Pattern 5: Optional Feature (Conditional)

**Syntax**: `WHERE <feature is included>, the <system> SHALL <action>.`

Behavior applies only when an optional feature is enabled.

**Examples for this project**:
- WHERE Telegram notifications are enabled, the system SHALL send trade alerts.
- WHERE walk-forward optimization is configured, the system SHALL partition data into
  training and validation windows.
- WHERE Kraken exchange is selected, the system SHALL use Kraken API endpoints.

### Combining Patterns

Complex requirements may combine patterns:

- WHILE in live trading mode, WHEN a risk limit is breached, the system SHALL
  immediately close all positions.
- WHERE alerts are enabled, IF maximum drawdown exceeds threshold, THEN the system
  SHALL send emergency notification.

### Keywords Reference

| Keyword | Meaning | Compliance |
|---------|---------|------------|
| SHALL | Mandatory | Must be implemented |
| SHALL NOT | Prohibition | Must not be implemented |
| SHOULD | Recommended | Implement unless justified |
| SHOULD NOT | Discouraged | Avoid unless justified |
| MAY | Optional | Implementer's discretion |

**Version**: 1.8.0 | **Ratified**: 2025-12-15 | **Last Amended**: 2025-12-31
