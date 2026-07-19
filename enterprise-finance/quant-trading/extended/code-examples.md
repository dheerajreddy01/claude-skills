# Quant Trading Code Examples

> This document contains the full set of code examples for the quant-trading skill

## Technical Indicator Toolbox

```python
# Common technical indicators

def sma(prices, period):
    """Simple Moving Average"""
    return prices.rolling(period).mean()

def ema(prices, period):
    """Exponential Moving Average"""
    return prices.ewm(span=period, adjust=False).mean()

def rsi(prices, period=14):
    """Relative Strength Index (0-100)"""
    delta = prices.diff()
    gain = delta.where(delta > 0, 0).rolling(period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(period).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

def bollinger_bands(prices, period=20, std_dev=2):
    """Bollinger Bands"""
    middle = sma(prices, period)
    std = prices.rolling(period).std()
    upper = middle + std_dev * std
    lower = middle - std_dev * std
    return upper, middle, lower

def macd(prices, fast=12, slow=26, signal=9):
    """MACD Indicator"""
    ema_fast = ema(prices, fast)
    ema_slow = ema(prices, slow)
    macd_line = ema_fast - ema_slow
    signal_line = ema(macd_line, signal)
    histogram = macd_line - signal_line
    return macd_line, signal_line, histogram

def atr(high, low, close, period=14):
    """Average True Range (volatility)"""
    tr1 = high - low
    tr2 = abs(high - close.shift())
    tr3 = abs(low - close.shift())
    tr = pd.concat([tr1, tr2, tr3], axis=1).max(axis=1)
    return tr.rolling(period).mean()
```

## Strategy Pattern Library

### Trend Following Strategy

```python
def trend_following_strategy(data, short_ma=20, long_ma=50):
    """
    Dual moving average trend strategy

    Rules:
    - Short MA > long MA and price > short MA → go long
    - Short MA < long MA and price < short MA → go short
    - Otherwise → flat
    """
    data['ma_short'] = data['close'].rolling(short_ma).mean()
    data['ma_long'] = data['close'].rolling(long_ma).mean()

    conditions = [
        (data['ma_short'] > data['ma_long']) & (data['close'] > data['ma_short']),
        (data['ma_short'] < data['ma_long']) & (data['close'] < data['ma_short'])
    ]
    choices = [1, -1]
    data['position'] = np.select(conditions, choices, default=0)

    return data
```

### Mean Reversion Strategy

```python
def mean_reversion_strategy(data, lookback=20, entry_z=2, exit_z=0):
    """
    Mean reversion strategy (Bollinger Bands)

    Rules:
    - Price touches the lower band (z < -2) → go long
    - Price touches the upper band (z > 2) → go short
    - Close position when it reverts to the mean
    """
    data['ma'] = data['close'].rolling(lookback).mean()
    data['std'] = data['close'].rolling(lookback).std()
    data['z_score'] = (data['close'] - data['ma']) / data['std']

    position = 0
    positions = []

    for z in data['z_score']:
        if z < -entry_z and position == 0:
            position = 1  # go long
        elif z > entry_z and position == 0:
            position = -1  # go short
        elif abs(z) < exit_z:
            position = 0  # close position
        positions.append(position)

    data['position'] = positions
    return data
```

### Pairs Trading Strategy

```python
def pairs_trading_strategy(price_a, price_b, lookback=60, entry_z=2, exit_z=0.5):
    """
    Pairs trading strategy

    Assumption: the spread between two highly correlated assets will revert
    """
    # Calculate the spread
    spread = price_a - price_b

    # Calculate the z-score
    spread_mean = spread.rolling(lookback).mean()
    spread_std = spread.rolling(lookback).std()
    z_score = (spread - spread_mean) / spread_std

    # Generate signals
    # z < -2: go long the spread (buy A, sell B)
    # z > 2: go short the spread (sell A, buy B)
    position_a = np.where(z_score < -entry_z, 1,
                 np.where(z_score > entry_z, -1, 0))
    position_b = -position_a

    return position_a, position_b, z_score
```

## Factor Calculation Examples

```python
# Momentum factor
def momentum_factor(prices, lookback=252):
    """Trailing 12-month return (excluding the most recent month)"""
    return prices.shift(21).pct_change(lookback - 21)

# Value factor
def value_factor(fundamentals):
    """E/P (inverse of P/E)"""
    return fundamentals['earnings'] / fundamentals['price']

# Factor normalization
def normalize_factor(factor):
    """Z-score normalization"""
    return (factor - factor.mean()) / factor.std()
```

## Risk Parity Allocation

```python
def risk_parity_weights(returns, target_risk=0.10):
    """
    Risk parity allocation: each asset contributes equal risk
    """
    volatilities = returns.std() * np.sqrt(252)
    inverse_vol = 1 / volatilities
    weights = inverse_vol / inverse_vol.sum()

    # Scale to target risk
    portfolio_vol = (weights * volatilities).sum()
    weights = weights * (target_risk / portfolio_vol)

    return weights
```

## Stop-Loss Strategy

```python
def trailing_stop(prices, positions, atr_multiplier=2):
    """
    ATR trailing stop
    """
    atr = calculate_atr(prices, period=14)
    stop_loss = []
    highest_since_entry = prices.iloc[0]
    lowest_since_entry = prices.iloc[0]

    for i, (price, pos, atr_val) in enumerate(zip(prices, positions, atr)):
        if pos > 0:  # long
            highest_since_entry = max(highest_since_entry, price)
            stop = highest_since_entry - atr_multiplier * atr_val
            stop_loss.append(stop)
        elif pos < 0:  # short
            lowest_since_entry = min(lowest_since_entry, price)
            stop = lowest_since_entry + atr_multiplier * atr_val
            stop_loss.append(stop)
        else:
            stop_loss.append(None)
            highest_since_entry = price
            lowest_since_entry = price

    return stop_loss
```

## Strategy Decay Detection

```python
def detect_regime_change(pnl_series, lookback=60):
    """
    Detect whether the strategy is decaying
    """
    recent_sharpe = pnl_series.tail(lookback).mean() / pnl_series.tail(lookback).std()
    historical_sharpe = pnl_series.head(-lookback).mean() / pnl_series.head(-lookback).std()

    # Warn if Sharpe ratio has dropped more than 50%
    if recent_sharpe < historical_sharpe * 0.5:
        return "WARNING: Strategy may be decaying"

    return "OK"
```

## Performance Attribution Analysis

```python
def performance_attribution(returns, benchmark_returns, positions):
    """
    Performance attribution analysis
    """
    # Alpha: excess return
    alpha = returns.mean() - benchmark_returns.mean()

    # Selection contribution
    selection = (returns - benchmark_returns).mean()

    # Timing contribution
    timing = (positions * (returns - returns.mean())).mean()

    # Information ratio
    tracking_error = (returns - benchmark_returns).std()
    information_ratio = alpha / tracking_error if tracking_error > 0 else 0

    return {
        'alpha': alpha * 252,  # annualized
        'selection': selection * 252,
        'timing': timing * 252,
        'information_ratio': information_ratio * np.sqrt(252)
    }
```

## Simple Moving Average Crossover Strategy

```python
def sma_crossover_strategy(data, short=10, long=30):
    """
    Short MA crosses above long MA → buy
    Short MA crosses below long MA → sell
    """
    data['SMA_short'] = data['close'].rolling(short).mean()
    data['SMA_long'] = data['close'].rolling(long).mean()

    data['signal'] = 0
    data.loc[data['SMA_short'] > data['SMA_long'], 'signal'] = 1
    data.loc[data['SMA_short'] < data['SMA_long'], 'signal'] = -1

    return data
```
