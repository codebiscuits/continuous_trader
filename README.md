# Continuous Trader
This is a cryptocurrency trading system that implements automated algorithmic trading strategies on margin trading pairs (primarily on Binance). Here's what it does and how it works:
## What It Does
The system:
- Analyzes multiple cryptocurrency trading pairs using technical indicators and strategies
- Generates trading forecasts by combining signals from multiple strategies
- Backtests strategies to evaluate performance before live trading
- Executes automated trades on Binance margin accounts
- Manages portfolio allocation across multiple trading pairs with leverage
- Tracks performance and maintains detailed records of trades and positions
## How It Works
### Core Architecture
The system has three main components:
1. Trader Class (in components.py)
    - Manages the overall trading account and portfolio
    - Handles backtesting across all trading pairs
    - Executes trades via Binance API
    - Manages leverage, borrowing, and position sizing
    - Keeps records of all trading activity
2. Coin Class (in components.py)
   - Represents individual trading pairs (e.g., BTCUSDT, ETHUSDT)
   - Downloads and updates OHLC (candlestick) data
   - Manages multiple trading strategies for each pair
   - Combines individual strategy forecasts into a unified forecast
   - Calculates performance-weighted dynamic forecasts
3. Strategy Modules (in strategies.py - referenced but not shown)
   - Individual trading strategies like:
     - StochRSIReversal - Mean reversion based on Stochastic RSI
     - ChanBreak - Channel breakout strategy
     - IchiTrend - Ichimoku-based trend following
     - HmaRoc - Hull Moving Average rate of change
     - Various others
### Trading Flow
1. Data Collection
   - Fetches 5-minute OHLC data from Binance
   - Aggregates to hourly data
   - Calculates volume-weighted moving averages (VWMA)
   - Computes dynamic volatility metrics
2. Strategy Execution
   - Each trading pair runs multiple strategy variants (different timeframes/parameters)
   - Strategies generate forecasts (-1 to +1, where negative = short, positive = long)
   - Individual forecasts are backtested to calculate rolling Sharpe ratios
3. Forecast Combination
   - Raw forecasts from all strategies are combined
   - Performance weighting: Strategies with better recent performance get higher weight
   - Dynamic weighting: Reduces exposure when strategies perform poorly
   - Final forecast determines position size for each pair
4. Portfolio Management
   - Three weighting schemes available:
     - flat: Equal weight to all pairs
     - lin: Linear weighting (top performers get more)
     - perf: Performance-based dynamic weighting
   - Applies target leverage (default 2.5x)
   - Maintains minimum transaction thresholds to reduce costs
5. Trade Execution
   - Calculates position differences between current and target
   - Borrows USDT or base assets as needed for leverage
   - Executes market orders via Binance API
   - Repays unnecessary loans
   - Logs all trades with slippage and fee tracking
## How to Use It
### Configuration
Edit ct.py to configure:
``` python
# Select trading pairs
markets = ['BTCUSDT', 'ETHUSDT', 'SOLUSDT', ...]  # Loaded from records/pairs_{year}.json

# Select strategies to use
strategies = [
    'srsirev',     # Stochastic RSI reversal
    'chanbreak',   # Channel breakout
    'ichitrend',   # Ichimoku trend
    'hmaroc'       # Hull MA rate of change
]

# Set parameters
leverage = 2.5                    # Target leverage multiplier
dyn_weight_lb = '1 week'          # Lookback for dynamic weighting
fc_weighting = False              # Weight individual forecasts by performance
port_weights = 'flat'             # Portfolio weighting: 'flat', 'lin', or 'perf'
```

### Running the System
#### For backtesting only (safe mode):
``` python
python ct.py
```

#### For live trading:
- The system checks for /pi_3.txt file to enable live mode
- Set in_production = True or run on a machine with that file
- Configure API keys in keys.py (Binance API key and secret)
### Key Functions
Backtest strategies:
``` python
trader.run_backtests(
    window='1 year',      # Backtest period
    show_stats=True,      # Print performance metrics
    plot_rtns=False,      # Plot returns
    plot_pnls=True        # Plot cumulative PnL
)
```

Execute trades:
``` python
trader.run_trading()  # Only trades if in_production=True
```
## Output
The system provides:
- **Performance metrics**: Sharpe ratio, total returns, drawdowns
- **Position summaries**: Current holdings vs targets
- **Trade logs**: Detailed records in `records/trades.json`
- **Session records**: Portfolio state in `records/session.json`
- **Charts**: Optional visualization of backtests and forecasts

## Safety Features
- **Buffer mechanism**: Only trades if position change exceeds minimum threshold (1%)
- **Minimum transaction size**: Avoids tiny trades that waste on fees ($10 minimum)
- **Dry-run mode**: By default, simulates trades without executing
- **BNB fee optimization**: Maintains BNB balance for reduced trading fees
- **Debt management**: Automatically repays unnecessary loans

## Important Notes
⚠️ **This is a margin trading system with leverage** - it can borrow assets and amplify both gains and losses. Use with caution and understand the risks.

The system is designed for automated operation (runs on Raspberry Pi hourly via cron) but can be run manually for testing and analysis.

