# AI Stock 系统设计原则

## 核心设计理念

**🎯 战略**: 框架支持全A股，实施分步推进
**📐 原则**: 高内聚、低耦合、易扩展、可演进

---

## 1. 标的类型抽象设计

### 1.1 设计思想

不同标的类型（股票、ETF、可转债、期权）具有**共性**和**特性**：

**共性（85%）**:
- 基本信息（代码、名称、市场）
- 行情数据（价格、成交量）
- K线数据（OHLC）
- 交易属性（交易时间、涨跌停限制）

**特性（15%）**:
- ETF: 净值、折溢价、成分股、申赎
- 股票: 财务数据、股东信息、分红
- 可转债: 转股价、溢价率、剩余期限
- 期权: 行权价、到期日、Greeks

### 1.2 设计方案

采用**模板方法模式 + 策略模式**:

```java
// 1. 抽象基类（模板方法）
public abstract class Security {
    protected String code;
    protected String name;
    protected SecurityType type;
    protected Market market;
    
    // 通用方法（共性）
    public abstract Quote getRealtimeQuote();
    public abstract List<Kline> getKlineData(Period period);
    public abstract TradingInfo getTradingInfo();
    
    // 扩展方法（特性）- 子类选择性实现
    public Object getExtraData() {
        return null; // 默认无扩展数据
    }
}

// 2. 具体实现
public class ETF extends Security {
    @Override
    public Object getExtraData() {
        return new ETFExtraData(
            nav,              // 净值
            premium,          // 溢价率
            components,       // 成分股
            subscription      // 申赎信息
        );
    }
}

public class Stock extends Security {
    @Override
    public Object getExtraData() {
        return new StockExtraData(
            financialData,    // 财务数据
            shareholders,     // 股东信息
            dividends         // 分红信息
        );
    }
}

// 3. 统一接口
@RestController
@RequestMapping("/api/v1/securities")
public class SecurityController {
    
    @GetMapping("/{code}")
    public SecurityVO getSecurity(@PathVariable String code) {
        Security security = securityService.getSecurity(code);
        return convertToVO(security); // 通用转换
    }
    
    @GetMapping("/{code}/extra")
    public Object getExtraData(@PathVariable String code) {
        Security security = securityService.getSecurity(code);
        return security.getExtraData(); // 扩展数据
    }
}
```

---

## 2. 数据模型设计原则

### 2.1 单一职责原则

**核心表**（通用，所有标的共用）:
```sql
-- 标的基本信息表
CREATE TABLE tb_security_info (
    id BIGINT PRIMARY KEY,
    security_code VARCHAR(10) NOT NULL,
    security_name VARCHAR(100) NOT NULL,
    security_type VARCHAR(20) NOT NULL,  -- STOCK/ETF/BOND/OPTION
    market VARCHAR(10) NOT NULL,          -- SH/SZ/BJ
    listing_date DATE,
    status TINYINT DEFAULT 1,
    create_time DATETIME,
    update_time DATETIME,
    UNIQUE KEY uk_code (security_code)
);

-- K线数据表（InfluxDB - 时序数据库）
MEASUREMENT: security_kline
TAGS: code, type, period
FIELDS: open, high, low, close, volume, amount
TIME: timestamp

-- 实时行情表（Redis - 内存数据库）
KEY: quote:realtime:{code}
HASH FIELDS: price, change, volume, amount, time
```

**扩展表**（按标的类型，一对一关联）:
```sql
-- ETF扩展表
CREATE TABLE tb_etf_extra (
    security_id BIGINT PRIMARY KEY,
    fund_company VARCHAR(100),
    tracking_index VARCHAR(100),
    unit_nav DECIMAL(10,4),
    iopv DECIMAL(10,4),
    fund_scale DECIMAL(15,2),
    -- ...
    FOREIGN KEY (security_id) REFERENCES tb_security_info(id)
);

-- 股票扩展表
CREATE TABLE tb_stock_extra (
    security_id BIGINT PRIMARY KEY,
    industry VARCHAR(50),
    sector VARCHAR(50),
    total_shares BIGINT,
    float_shares BIGINT,
    -- ...
    FOREIGN KEY (security_id) REFERENCES tb_security_info(id)
);

-- 财务数据表（仅股票）
CREATE TABLE tb_stock_financial (
    id BIGINT PRIMARY KEY,
    security_id BIGINT,
    report_date DATE,
    total_revenue DECIMAL(20,2),
    net_profit DECIMAL(20,2),
    -- ...
    FOREIGN KEY (security_id) REFERENCES tb_security_info(id)
);
```

### 2.2 开闭原则

**新增标的类型，只需**:
1. 新增扩展表（不影响核心表）
2. 实现对应的Service类
3. 注册到Spring容器

**无需修改**:
- 核心数据模型
- 通用查询接口
- 回测引擎
- 交易管理模块

---

## 3. 策略框架设计

### 3.1 策略抽象

```python
from abc import ABC, abstractmethod
from typing import List

class BaseStrategy(ABC):
    """策略基类 - 适用于所有标的类型"""
    
    def __init__(self, security_type: SecurityType):
        self.security_type = security_type
        self.positions = {}
    
    @abstractmethod
    def on_data(self, data: MarketData):
        """行情数据回调 - 核心逻辑"""
        pass
    
    @abstractmethod
    def should_buy(self, code: str, data: MarketData) -> bool:
        """买入信号判断"""
        pass
    
    @abstractmethod
    def should_sell(self, code: str, data: MarketData) -> bool:
        """卖出信号判断"""
        pass
    
    def get_supported_types(self) -> List[SecurityType]:
        """支持的标的类型"""
        return [self.security_type]

# 具体策略 - 可复用
class MACrossStrategy(BaseStrategy):
    """双均线策略 - 通用策略，适用股票、ETF等"""
    
    def __init__(self, fast_period=5, slow_period=20):
        super().__init__(SecurityType.ALL)  # 支持所有标的
        self.fast_period = fast_period
        self.slow_period = slow_period
    
    def should_buy(self, code: str, data: MarketData) -> bool:
        ma_fast = data.sma(self.fast_period)
        ma_slow = data.sma(self.slow_period)
        return ma_fast > ma_slow and ma_fast[-2] <= ma_slow[-2]  # 金叉
    
    def should_sell(self, code: str, data: MarketData) -> bool:
        ma_fast = data.sma(self.fast_period)
        ma_slow = data.sma(self.slow_period)
        return ma_fast < ma_slow and ma_fast[-2] >= ma_slow[-2]  # 死叉
```

### 3.2 策略注册机制

```python
# 策略工厂
class StrategyFactory:
    _strategies = {}
    
    @classmethod
    def register(cls, name: str, strategy_class):
        """注册策略"""
        cls._strategies[name] = strategy_class
    
    @classmethod
    def create(cls, name: str, **kwargs):
        """创建策略实例"""
        if name not in cls._strategies:
            raise ValueError(f"Strategy {name} not found")
        return cls._strategies[name](**kwargs)
    
    @classmethod
    def list_strategies(cls, security_type: SecurityType = None):
        """列出可用策略"""
        if security_type is None:
            return list(cls._strategies.keys())
        # 过滤支持该标的类型的策略
        return [
            name for name, strategy_class in cls._strategies.items()
            if security_type in strategy_class().get_supported_types()
        ]

# 使用示例
StrategyFactory.register('ma_cross', MACrossStrategy)
StrategyFactory.register('rsi', RSIStrategy)
StrategyFactory.register('etf_rotation', ETFRotationStrategy)  # ETF专用

# 创建策略
strategy = StrategyFactory.create('ma_cross', fast_period=5, slow_period=20)
```

---

## 4. 回测引擎设计

### 4.1 标的无关设计

```python
class BacktestEngine:
    """回测引擎 - 标的类型无关"""
    
    def __init__(self, strategy: BaseStrategy, initial_capital: float):
        self.strategy = strategy
        self.broker = Broker(initial_capital)  # 模拟交易撮合
        self.data_feed = None
    
    def add_data(self, code: str, data: pd.DataFrame):
        """添加数据 - 支持任意标的"""
        self.data_feed.add_security(code, data)
    
    def run(self):
        """执行回测"""
        for timestamp, market_data in self.data_feed:
            # 1. 策略计算信号
            signals = self.strategy.on_data(market_data)
            
            # 2. 执行交易
            for signal in signals:
                if signal.action == 'BUY':
                    self.broker.buy(signal.code, signal.size, signal.price)
                elif signal.action == 'SELL':
                    self.broker.sell(signal.code, signal.size, signal.price)
            
            # 3. 更新账户
            self.broker.update(market_data)
        
        return self.generate_report()
    
    def generate_report(self):
        """生成回测报告"""
        return BacktestReport(
            trades=self.broker.get_trades(),
            positions=self.broker.get_positions(),
            equity_curve=self.broker.get_equity_curve(),
            metrics=self.calculate_metrics()
        )
```

---

## 5. 扩展性检查清单

### ✅ 一期实现（ETF）

| 模块 | 通用设计 | ETF实现 | 扩展性 |
|-----|---------|--------|-------|
| 数据模型 | ✅ tb_security_info | ✅ tb_etf_extra | ⭐⭐⭐⭐⭐ |
| 数据采集 | ✅ DataCollector接口 | ✅ ETFDataCollector | ⭐⭐⭐⭐⭐ |
| 策略框架 | ✅ BaseStrategy | ✅ ETF策略模板 | ⭐⭐⭐⭐⭐ |
| 回测引擎 | ✅ BacktestEngine | ✅ 支持ETF | ⭐⭐⭐⭐⭐ |
| 账户管理 | ✅ 多标的账户 | ✅ ETF持仓 | ⭐⭐⭐⭐⭐ |

### 🚀 二期扩展（个股）

| 功能 | 新增工作 | 改动范围 |
|-----|---------|---------|
| 数据采集 | 实现StockDataCollector | 新增类 |
| 数据表 | tb_stock_extra + tb_stock_financial | 新增表 |
| 策略 | 添加基本面策略 | 新增策略类 |
| 接口 | /api/v1/stocks/* | 新增路由 |
| **核心模块** | **无需修改** | ✅ |

---

## 6. 代码组织建议

```
ai-stock/
├── backend/
│   ├── common/                    # 通用模块
│   │   ├── domain/
│   │   │   ├── Security.java      # 标的抽象类
│   │   │   ├── Quote.java         # 行情接口
│   │   │   └── Kline.java         # K线接口
│   │   └── service/
│   │       └── DataCollector.java # 数据采集接口
│   │
│   ├── data-service/              # 数据服务
│   │   ├── collector/
│   │   │   ├── ETFDataCollector.java     # 一期
│   │   │   └── StockDataCollector.java   # 二期
│   │   └── repository/
│   │       ├── SecurityRepository.java    # 通用
│   │       ├── ETFRepository.java         # ETF专用
│   │       └── StockRepository.java       # 股票专用（二期）
│   │
│   ├── strategy-service/          # 策略服务
│   │   └── strategies/
│   │       ├── common/            # 通用策略（所有标的）
│   │       ├── etf/               # ETF专用策略
│   │       └── stock/             # 股票专用策略（二期）
│   │
│   └── backtest-service/          # 回测服务（通用）
│       ├── engine/                # 回测引擎（标的无关）
│       └── report/                # 报告生成（通用）
```

---

## 7. 总结

### 设计原则总结

1. **抽象优于具体**: 核心模块基于抽象设计，不依赖具体标的
2. **组合优于继承**: 使用策略模式、工厂模式等组合设计
3. **接口隔离**: 通用接口 + 扩展接口，按需实现
4. **依赖倒置**: 高层模块不依赖低层模块，都依赖抽象

### 扩展路径

```
一期（ETF）:
  ✅ 建立通用框架
  ✅ 实现ETF功能
  ✅ 验证架构可行性

二期（个股）:
  → 新增Stock相关实现
  → 核心框架无需改动
  → 增量开发

三期（其他）:
  → 继续扩展标的类型
  → 框架持续演进
```

---

**版本**: v1.0  
**更新日期**: 2025-11-19  
**状态**: 设计原则文档

