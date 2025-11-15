# CLAUDE.md - Strategies Repository Guide

## Repository Overview

This repository contains a collection of **TradingView Pine Script trading strategies** designed for algorithmic trading across various markets (crypto, forex, stocks). All strategies are written in **Pine Script v5** and focus on technical analysis with robust risk management.

**Repository Purpose**: Development and maintenance of algorithmic trading strategies with emphasis on:
- Non-repainting signals
- Multi-timeframe analysis
- Advanced risk management (TP/SL levels)
- Backtesting capabilities
- Webhook integration for automated trading

---

## Codebase Structure

### Core Strategy Files

```
Strategies/
├── ALGOX v13.pine              # Advanced strategy with ALMA ribbons, ATR-based exits
├── SAIYAN_OCC_STRATEGY_R5.41.pine  # Supply/demand zones with support/resistance
├── XXX Claude.txt              # Hybrid HTF strategy (improved non-repainting)
├── XXX_MODDED.txt              # Base version of XXX strategy
├── XXX_MODDED_original.txt     # Original backup version
├── XXX Claude_original.txt     # Original backup version
├── XXX cgpt.txt                # GPT-assisted variation
├── XXX cgpt - Copy.txt         # Backup copy
├── RR DANGOV v3.3.pine         # Multi-indicator comprehensive strategy
├── HARMONIC V.5 .pine          # Harmonic/Gold Digger EMA-based strategy
├── GOLD DIGGER .pine           # Simple breakout strategy
└── ESTIMATIONS_10M.PINE        # Estimation-based strategy
```

### Strategy Categorization

| Strategy | Type | Complexity | Main Indicators |
|----------|------|------------|----------------|
| **ALGOX v13** | Trend Following | High | ALMA, ATR, Renko, EMA |
| **SAIYAN OCC** | S/R Zones | High | Pivot Points, S/D Zones, Keltner Channels |
| **XXX Hybrid** | Multi-Timeframe | High | ALMA, HTF Signals, Trend Filters |
| **RR DANGOV** | Multi-Strategy | Very High | Guppy, VWAP, ORB, Supertrend, CPR, WaveTrend |
| **HARMONIC/Gold Digger** | EMA Energy | Medium | EMAs (9-20 periods), Supertrend, PSAR |

---

## Key Technical Components

### 1. Moving Averages & Smoothing
- **ALMA** (Arnaud Legoux Moving Average): Primary smoothing method
- **TEMA/DEMA**: Triple/Double Exponential Moving Averages
- **HullMA**: Hull Moving Average for low-lag signals
- **Guppy EMAs**: Multiple EMA ribbons (fast: 3-23, slow: 25-70)

### 2. Volatility & Range Indicators
- **ATR** (Average True Range): Used for dynamic TP/SL levels
- **Keltner Channels**: Multi-level channels for overbought/oversold
- **Range Filter**: C-125 smoothed range filtering
- **Supertrend**: Volatility-based trend indicator

### 3. Support/Resistance Systems
- **Supply/Demand Zones**: Automated zone detection with box drawing
- **Pivot Points**: Daily/Weekly/Monthly pivots with S/R levels
- **CPR** (Central Pivot Range): TC, Pivot, BC levels
- **ORB** (Opening Range Breakout): Configurable time-based ranges

### 4. Signal Generation
- **Crossover/Crossunder**: MA crosses for entry/exit
- **VWAP**: Volume-weighted average price for institutional levels
- **WaveTrend Oscillator**: Momentum-based oversold/overbought signals
- **Stochastic**: Traditional overbought/oversold momentum

### 5. Risk Management Features
- **Multi-Level Take Profit**: TP1, TP2, TP3 with percentage-based exits
- **ATR-Based Stops**: Dynamic stop loss calculation
- **Trailing Stops**: Optional trailing stop functionality
- **Position Sizing**: USD-based or SL-based quantity calculation
- **Max Daily Trades**: Prevents overtrading
- **Max Daily Profit**: Automatic pause after target reached

---

## Development Workflows

### Testing a Strategy Modification

```pinescript
//@version=5
strategy("Test Strategy", overlay=true)

// 1. Always include backtesting date range
i_startTime = input.time(timestamp("01 Jan 2023"), "Start Date")
i_endTime = input.time(timestamp("31 Dec 2099"), "End Date")
tradeDateIsAllowed = time >= i_startTime and time <= i_endTime

// 2. Define risk parameters clearly
stopLoss = input.float(1.0, "Stop Loss %") / 100
takeProfit = input.float(2.0, "Take Profit %") / 100

// 3. Use non-repainting security calls
rp_security(_symbol, _res, _src) =>
    request.security(_symbol, _res, _src[barstate.isrealtime ? 1 : 0])

// 4. Always validate conditions
if buy_signal and tradeDateIsAllowed
    strategy.entry("Long", strategy.long)
```

### Adding a New Indicator

1. **Define in Inputs Section**: Use `input.` functions with proper grouping
2. **Calculate Indicator**: Place calculations in dedicated section
3. **Create Conditions**: Use boolean variables for clarity
4. **Integrate Filters**: Add to existing filter logic (AND/OR)
5. **Test Thoroughly**: Backtest across multiple timeframes and markets

### Version Control Best Practices

```bash
# Before making significant changes
cp "strategy.pine" "strategy_backup_$(date +%Y%m%d).pine"

# Test changes on copy first
# Commit with descriptive messages
git add .
git commit -m "Add EMA 200 filter to ALGOX strategy"
git push -u origin claude/[branch-name]
```

---

## Key Conventions for AI Assistants

### Code Style Guidelines

1. **Indentation**: Use 4 spaces (not tabs) for consistency
2. **Variable Naming**:
   - Lowercase with underscores: `long_signal`, `atr_value`
   - Descriptive names: `ema_200` not `e200`
   - Constants in UPPERCASE: `VERSION = 'v13'`

3. **Function Naming**:
   - Prefix with `f_`: `f_array_add_pop()`, `f_supply_demand()`
   - Descriptive action verbs: `f_check_overlapping()`, `f_extend_box_endpoint()`

4. **Grouping**:
   ```pinescript
   // === INPUTS ===
   // Input parameters here
   // === /INPUTS ===

   // === CALCULATIONS ===
   // Indicator calculations
   // === /CALCULATIONS ===

   // === STRATEGY ===
   // Entry/exit logic
   // === /STRATEGY ===
   ```

### Non-Repainting Principles

**CRITICAL**: All HTF (Higher TimeFrame) requests must avoid repainting

```pinescript
// ❌ BAD - Will repaint
htf_close = request.security(syminfo.tickerid, "60", close)

// ✅ GOOD - Non-repainting
htf_close = request.security(syminfo.tickerid, "60", close,
    lookahead=barmerge.lookahead_on, gaps=barmerge.gaps_off)

// ✅ GOOD - With offset for historical data
rp_security(_symbol, _res, _src) =>
    request.security(_symbol, _res, _src[barstate.isrealtime ? 1 : 0])
```

### Alert & Webhook Integration

All strategies support webhook alerts for automated trading:

```pinescript
// Alert message format for API integration
i_alert_txt_entry_long = input.text_area(defval="", title="Long Entry Message")

// Webhook JSON format (example)
'[{"C":"CURRENCY","MODE":"HEDGE","LEV":"10","TYPE":"ISOLATED","TS":"BTCUSDT","Q":"100","TT":"BUY","TP":"2.5","SL":"1.0","OT":"MARKET","AT":"BINANCE_PERPETUAL"}]'
```

### Common Patterns to Maintain

1. **Entry Conditions**:
   ```pinescript
   leTrigger = ta.crossover(closeSeriesAlt, openSeriesAlt)
   seTrigger = ta.crossunder(closeSeriesAlt, openSeriesAlt)
   ```

2. **Multi-Level Exits**:
   ```pinescript
   strategy.exit("LXTP1", qty_percent=50, limit=tp1Line, stop=slLine)
   strategy.exit("LXTP2", qty_percent=30, limit=tp2Line, stop=slLine)
   strategy.exit("LXTP3", qty_percent=20, limit=tp3Line, stop=slLine)
   ```

3. **Visual Indicators**:
   ```pinescript
   plotshape(longEntry, style=shape.labelup, location=location.belowbar,
       color=color.green, text="BUY")
   ```

---

## Testing & Validation Checklist

### Before Committing Changes

- [ ] **Syntax Check**: Code compiles without errors
- [ ] **Backtest**: Run on multiple timeframes (1m, 5m, 15m, 1h, 1D)
- [ ] **Verify Signals**: Check for repainting on replay mode
- [ ] **Risk Management**: TP/SL levels calculate correctly
- [ ] **Performance Metrics**: Win rate, profit factor, max drawdown reasonable
- [ ] **Visual Check**: Labels, lines, and shapes display correctly
- [ ] **Alert Test**: Webhook messages format correctly (if applicable)

### Performance Benchmarks

Good strategy characteristics:
- **Win Rate**: 40-65% (varies by strategy type)
- **Profit Factor**: > 1.5
- **Max Drawdown**: < 20%
- **Risk/Reward**: Minimum 1:1.5 ratio
- **Sharpe Ratio**: > 1.0 (if available)

---

## Common Modifications

### Adding a New Filter

```pinescript
// 1. Add input
useTrendFilter = input.bool(true, "Use Trend Filter", group="FILTERS")
trendEMA = input.int(200, "Trend EMA Period", group="FILTERS")

// 2. Calculate
ema_trend = ta.ema(close, trendEMA)
bullishTrend = close > ema_trend

// 3. Apply to entries
if buy_signal and (not useTrendFilter or bullishTrend)
    strategy.entry("Long", strategy.long)
```

### Adjusting Risk Levels

```pinescript
// Modify these inputs for different risk profiles
i_lxLvlTP1 = input.float(1.0, "TP1 %")  // Conservative: 0.5-1.0%
i_lxLvlTP2 = input.float(2.0, "TP2 %")  // Moderate: 1.5-2.5%
i_lxLvlTP3 = input.float(3.0, "TP3 %")  // Aggressive: 3.0-5.0%
i_lxLvlSL  = input.float(0.5, "SL %")   // Tight: 0.3-0.5%, Loose: 1.0-2.0%
```

### Adding New Entry Signal

```pinescript
// Pattern: Create boolean conditions
newSignal = ta.crossover(indicator1, indicator2) and filter_condition

// Add to entry logic
longEntry = (existing_signal or newSignal) and filters_ok
```

---

## Troubleshooting Guide

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **Repainting Signals** | Incorrect `request.security()` usage | Use `lookahead_on` with offset or `barstate.isconfirmed` |
| **Too Many Alerts** | No bar confirmation | Add `barstate.isconfirmed` check |
| **Incorrect TP/SL** | Wrong calculation order | Calculate AFTER entry price is set |
| **Strategy Not Trading** | Filters too restrictive | Check `tradeDateIsAllowed`, time filters, trend filters |
| **Visual Clutter** | Too many plots | Use `display.none` or `na` conditionally |
| **Performance Lag** | Heavy calculations | Optimize loops, use `varip` for intrabar data |

### Debug Techniques

```pinescript
// 1. Use plot for debugging values
plot(debug_value, "Debug", display=display.data_window)

// 2. Create debug table
var table debugTable = table.new(position.top_right, 2, 5)
if barstate.islast
    table.cell(debugTable, 0, 0, "Signal", text_color=color.white)
    table.cell(debugTable, 1, 0, str.tostring(buy_signal))

// 3. Log to labels (sparingly)
if barstate.islast
    label.new(bar_index, close, "Debug: " + str.tostring(value))
```

---

## API & Integration Notes

### Webhook Alert Format

Most strategies use JSON format for webhook integration:

```json
[{
  "C": "CURRENCY",
  "MODE": "HEDGE",
  "LEV": "10",
  "TYPE": "ISOLATED",
  "TS": "BTCUSDT",
  "Q": "100",
  "TT": "BUY",
  "TP": "2.5",
  "SL": "1.0",
  "OT": "MARKET",
  "AT": "BINANCE_PERPETUAL"
}]
```

**Fields**:
- `C`: Currency/Asset class
- `MODE`: HEDGE or ONE-WAY
- `LEV`: Leverage (1-125)
- `TYPE`: ISOLATED or CROSS
- `TS`: Trading Symbol (e.g., BTCUSDT)
- `Q`: Quantity in quote currency
- `TT`: Trade Type (BUY/SELL)
- `TP`: Take Profit percentage
- `SL`: Stop Loss percentage
- `OT`: Order Type (MARKET/LIMIT)
- `AT`: Account Type

---

## File Naming Conventions

- **Production Files**: `STRATEGY_NAME vX.X.pine`
- **Development Files**: `STRATEGY_NAME_test.pine` or `XXX Claude.txt`
- **Backups**: `STRATEGY_NAME_original.txt` or `_backup_YYYYMMDD.txt`
- **Variations**: `STRATEGY_NAME_MODDED.txt` or `STRATEGY_NAME cgpt.txt`

---

## Important Reminders for AI Assistants

### DO:
✅ Always test on multiple timeframes before committing
✅ Maintain non-repainting logic for production strategies
✅ Document all changes with clear comments
✅ Preserve existing risk management systems
✅ Use input groups for organization
✅ Include tooltips for complex parameters
✅ Test with backtesting date ranges

### DON'T:
❌ Remove existing safety checks (date filters, position size limits)
❌ Break webhook alert message formats
❌ Introduce repainting in HTF analysis
❌ Remove existing visual indicators without discussion
❌ Change default risk parameters without explicit request
❌ Modify core strategy logic without thorough testing
❌ Use deprecated Pine Script v4 syntax

---

## Resources & References

### Pine Script Documentation
- [Pine Script v5 User Manual](https://www.tradingview.com/pine-script-docs/en/v5/index.html)
- [Pine Script Reference](https://www.tradingview.com/pine-script-reference/v5/)
- [Strategy Tester](https://www.tradingview.com/support/solutions/43000481656)

### Key Concepts
- **Repainting**: Signals that change on historical bars
- **Lookahead**: Future data leaking into past (causes unrealistic backtests)
- **Pyramiding**: Multiple entries in same direction
- **Overlay**: Strategy plots on price chart vs separate pane

### Timeframe Syntax
- Intraday: `"1"`, `"5"`, `"15"`, `"60"` (minutes)
- Daily: `"D"` or `"1D"`
- Weekly: `"W"` or `"1W"`
- Monthly: `"M"` or `"1M"`

---

## Version History

- **2024-03**: ALGOX v13 - Added Renko support, ATR-based exits
- **2023-10**: XXX v6.1.23 - Base strategy with SAIYAN OCC integration
- **2024**: XXX Hybrid HTF v111 - Improved non-repainting, re-entry logic
- **v5.41**: SAIYAN OCC - Supply/demand zones with break of structure

---

## Contact & Support

For questions or issues related to:
- **Strategy Logic**: Review inline comments in respective `.pine` files
- **Risk Parameters**: See "Risk Management Features" section
- **Webhook Integration**: See "API & Integration Notes" section
- **Backtesting**: Use TradingView's strategy tester with date range filters

---

**Last Updated**: 2025-11-15
**Pine Script Version**: v5
**Total Strategies**: 11 active files
**Primary Author**: Multiple contributors (traderschatroom88, gu5tavo71, BackQuant, QuantNomad)

