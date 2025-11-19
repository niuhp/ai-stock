# AI Stock - 通用数据模型设计

## 设计原则

1. **标的类型无关**: 使用通用字段设计，支持ETF、股票、期权等多种标的
2. **扩展性优先**: 预留扩展字段，方便后续功能迭代
3. **一期ETF实现**: 一期只填充ETF相关数据，二期扩展个股
4. **统一命名**: 使用 `instrument` 代替 `stock/etf`，表示金融工具

---

## 1. 核心数据表

### 1.1 金融工具基本信息表 (tb_instrument_info)

**设计说明**: 通用的金融工具表，支持ETF、股票、期权等

```sql
CREATE TABLE tb_instrument_info (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    instrument_code VARCHAR(20) NOT NULL UNIQUE COMMENT '标的代码',
    instrument_name VARCHAR(100) NOT NULL COMMENT '标的名称',
    instrument_type VARCHAR(20) NOT NULL COMMENT '标的类型: ETF, STOCK, OPTION, INDEX',
    exchange VARCHAR(10) COMMENT '交易所: SH, SZ',
    
    -- ETF专属字段（一期使用）
    fund_company VARCHAR(100) COMMENT '基金公司',
    tracking_index VARCHAR(100) COMMENT '跟踪指数',
    index_code VARCHAR(20) COMMENT '指数代码',
    etf_type VARCHAR(50) COMMENT 'ETF类型: 股票型、债券型、商品型',
    
    -- 股票专属字段（二期使用）
    industry VARCHAR(50) COMMENT '所属行业',
    sector VARCHAR(50) COMMENT '所属板块',
    market_cap DECIMAL(20,2) COMMENT '总市值',
    is_st TINYINT DEFAULT 0 COMMENT '是否ST股',
    
    -- 通用字段
    listing_date DATE COMMENT '上市日期',
    status TINYINT DEFAULT 1 COMMENT '状态: 0-停牌, 1-正常, 2-退市',
    attributes JSON COMMENT '扩展属性（JSON格式）',
    
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_code (instrument_code),
    INDEX idx_type (instrument_type),
    INDEX idx_exchange (exchange)
) COMMENT='金融工具基本信息表（通用设计）';
```

**一期数据示例**:
```sql
-- ETF数据
INSERT INTO tb_instrument_info VALUES 
(1, '510300', '沪深300ETF', 'ETF', 'SH', '华泰柏瑞', '沪深300', '000300', '股票型', NULL, NULL, NULL, 0, '2012-05-28', 1, NULL, NOW(), NOW());

-- 二期可以插入股票数据
-- (2, '600000', '浦发银行', 'STOCK', 'SH', NULL, NULL, NULL, NULL, '银行', '金融', 5000.00, 0, '1999-11-10', 1, NULL, NOW(), NOW());
```

### 1.2 K线数据表 (InfluxDB)

**设计说明**: 使用InfluxDB存储时序数据，天然支持多标的类型

```
MEASUREMENT: kline_data

TAGS:
  - instrument_code: 标的代码
  - instrument_type: 标的类型 (ETF/STOCK/OPTION)
  - period: 周期 (1m/5m/15m/30m/60m/1d/1w/1M)
  - exchange: 交易所

FIELDS:
  - open: FLOAT (开盘价)
  - high: FLOAT (最高价)
  - low: FLOAT (最低价)
  - close: FLOAT (收盘价)
  - volume: INTEGER (成交量)
  - amount: FLOAT (成交额)
  - change_rate: FLOAT (涨跌幅)

TIME: timestamp

EXAMPLE:
kline_data,instrument_code=510300,instrument_type=ETF,period=1d,exchange=SH 
  open=4.123,high=4.156,low=4.098,close=4.145,volume=123456789,amount=509876543.21,change_rate=0.0053
  1700000000000000000
```

### 1.3 实时行情表 (Redis)

**设计说明**: Redis存储实时行情，使用统一的key命名

```
KEY: realtime:{instrument_type}:{instrument_code}
TYPE: Hash

FIELDS:
  current_price: 当前价
  change_rate: 涨跌幅
  change_amount: 涨跌额
  volume: 成交量
  amount: 成交额
  open_price: 开盘价
  high_price: 最高价
  low_price: 最低价
  pre_close: 昨收价
  bid_price: 买一价
  ask_price: 卖一价
  turnover_rate: 换手率
  update_time: 更新时间
  
  -- ETF专属
  iopv: 实时净值
  nav: 单位净值
  
  -- 股票专属（二期）
  pe_ratio: 市盈率
  pb_ratio: 市净率

EXAMPLE:
HSET realtime:ETF:510300 current_price 4.145 change_rate 0.0053 volume 123456789 ...
HSET realtime:STOCK:600000 current_price 12.34 pe_ratio 5.6 pb_ratio 0.8 ...
```

### 1.4 策略配置表 (tb_strategy)

```sql
CREATE TABLE tb_strategy (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    strategy_name VARCHAR(100) NOT NULL COMMENT '策略名称',
    strategy_type VARCHAR(50) COMMENT '策略类型',
    strategy_code TEXT COMMENT '策略代码',
    parameters JSON COMMENT '策略参数',
    
    -- 支持的标的类型（通用设计）
    supported_instrument_types VARCHAR(100) COMMENT '支持的标的类型: ETF,STOCK,OPTION',
    default_instrument_type VARCHAR(20) COMMENT '默认标的类型',
    
    description TEXT COMMENT '策略描述',
    creator_id BIGINT COMMENT '创建人ID',
    status TINYINT DEFAULT 1 COMMENT '状态: 0-停用, 1-启用',
    
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_type (strategy_type),
    INDEX idx_creator (creator_id)
) COMMENT='策略配置表（支持多标的类型）';
```

### 1.5 回测记录表 (tb_backtest)

```sql
CREATE TABLE tb_backtest (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    strategy_id BIGINT COMMENT '策略ID',
    
    -- 标的信息（通用）
    instrument_codes JSON COMMENT '标的代码列表（JSON数组）',
    instrument_type VARCHAR(20) COMMENT '标的类型: ETF, STOCK, OPTION',
    
    -- 回测参数
    start_date DATE COMMENT '开始日期',
    end_date DATE COMMENT '结束日期',
    initial_capital DECIMAL(15,2) COMMENT '初始资金',
    commission_rate DECIMAL(6,4) COMMENT '手续费率',
    slippage_rate DECIMAL(6,4) COMMENT '滑点率',
    
    -- 回测结果
    final_capital DECIMAL(15,2) COMMENT '最终资金',
    total_return DECIMAL(10,4) COMMENT '总收益率',
    annual_return DECIMAL(10,4) COMMENT '年化收益率',
    max_drawdown DECIMAL(10,4) COMMENT '最大回撤',
    sharpe_ratio DECIMAL(10,4) COMMENT '夏普比率',
    win_rate DECIMAL(5,2) COMMENT '胜率',
    trade_count INT COMMENT '交易次数',
    
    status TINYINT COMMENT '状态: 0-运行中, 1-已完成, 2-失败',
    error_message TEXT COMMENT '错误信息',
    
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_strategy (strategy_id),
    INDEX idx_type (instrument_type),
    INDEX idx_date (start_date, end_date)
) COMMENT='回测记录表（支持多标的）';
```

### 1.6 账户表 (tb_account)

```sql
CREATE TABLE tb_account (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL COMMENT '用户ID',
    account_name VARCHAR(100) COMMENT '账户名称',
    account_type TINYINT COMMENT '账户类型: 1-模拟, 2-实盘',
    
    -- 资金信息
    initial_capital DECIMAL(15,2) COMMENT '初始资金',
    available_cash DECIMAL(15,2) COMMENT '可用资金',
    frozen_cash DECIMAL(15,2) COMMENT '冻结资金',
    market_value DECIMAL(15,2) COMMENT '持仓市值',
    total_assets DECIMAL(15,2) COMMENT '总资产',
    total_profit DECIMAL(15,2) COMMENT '总盈亏',
    total_return_rate DECIMAL(10,4) COMMENT '总收益率',
    
    status TINYINT DEFAULT 1 COMMENT '状态: 0-停用, 1-正常',
    
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_user (user_id)
) COMMENT='账户表（支持混合持仓）';
```

### 1.7 持仓表 (tb_position)

```sql
CREATE TABLE tb_position (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_id BIGINT NOT NULL COMMENT '账户ID',
    
    -- 标的信息（通用）
    instrument_code VARCHAR(20) NOT NULL COMMENT '标的代码',
    instrument_type VARCHAR(20) NOT NULL COMMENT '标的类型: ETF, STOCK, OPTION',
    instrument_name VARCHAR(100) COMMENT '标的名称',
    
    -- 持仓信息
    shares INT COMMENT '持仓数量',
    available_shares INT COMMENT '可用数量',
    frozen_shares INT DEFAULT 0 COMMENT '冻结数量',
    avg_cost DECIMAL(10,4) COMMENT '平均成本',
    current_price DECIMAL(10,4) COMMENT '当前价格',
    market_value DECIMAL(15,2) COMMENT '市值',
    profit_loss DECIMAL(15,2) COMMENT '盈亏',
    profit_loss_rate DECIMAL(10,4) COMMENT '盈亏比例',
    
    -- 辅助字段
    first_buy_date DATE COMMENT '首次买入日期',
    last_trade_date DATE COMMENT '最后交易日期',
    
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    UNIQUE KEY uk_account_instrument (account_id, instrument_code),
    INDEX idx_instrument (instrument_code),
    INDEX idx_type (instrument_type)
) COMMENT='持仓表（支持多标的类型）';
```

### 1.8 交易记录表 (tb_trade)

```sql
CREATE TABLE tb_trade (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_id BIGINT NOT NULL COMMENT '账户ID',
    strategy_id BIGINT COMMENT '策略ID',
    
    -- 标的信息（通用）
    instrument_code VARCHAR(20) NOT NULL COMMENT '标的代码',
    instrument_type VARCHAR(20) NOT NULL COMMENT '标的类型: ETF, STOCK, OPTION',
    instrument_name VARCHAR(100) COMMENT '标的名称',
    
    -- 交易信息
    trade_type TINYINT COMMENT '交易类型: 1-买入, 2-卖出',
    shares INT COMMENT '交易数量',
    price DECIMAL(10,4) COMMENT '成交价格',
    amount DECIMAL(15,2) COMMENT '交易金额',
    commission DECIMAL(10,2) COMMENT '手续费',
    stamp_tax DECIMAL(10,2) DEFAULT 0 COMMENT '印花税（股票卖出）',
    
    trade_time DATETIME COMMENT '交易时间',
    signal_source VARCHAR(50) COMMENT '信号来源: MANUAL, STRATEGY, SYSTEM',
    
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_account (account_id),
    INDEX idx_instrument (instrument_code),
    INDEX idx_type (instrument_type),
    INDEX idx_time (trade_time)
) COMMENT='交易记录表（支持多标的）';
```

---

## 2. 数据迁移策略

### 一期到二期数据迁移

**无需迁移**: 由于使用了通用设计，一期ETF数据已经存储在通用表中

**二期只需**:
1. 在 `tb_instrument_info` 中插入股票数据，`instrument_type='STOCK'`
2. InfluxDB和Redis使用相同的数据结构，只是标的类型不同
3. 所有业务逻辑通过 `instrument_type` 字段区分处理

### 示例：二期添加股票数据

```sql
-- 插入股票基本信息
INSERT INTO tb_instrument_info 
(instrument_code, instrument_name, instrument_type, exchange, industry, sector, market_cap, listing_date, status)
VALUES
('600000', '浦发银行', 'STOCK', 'SH', '银行', '金融', 500000000000.00, '1999-11-10', 1);

-- 回测时可以混合使用
INSERT INTO tb_backtest (strategy_id, instrument_codes, instrument_type, ...)
VALUES (1, '["510300", "600000"]', 'MIXED', ...);
```

---

## 3. 扩展字段说明

### 3.1 JSON扩展字段 (attributes)

为每个标的类型预留扩展字段，避免频繁加字段：

```json
// ETF扩展属性
{
  "management_fee": 0.005,
  "custodian_fee": 0.001,
  "fund_scale": 5000000000.00,
  "establishment_date": "2012-01-01"
}

// 股票扩展属性（二期）
{
  "total_shares": 10000000000,
  "float_shares": 8000000000,
  "controlling_shareholder": "XX集团",
  "actual_controller": "XX"
}
```

---

## 4. 性能优化建议

1. **分表策略**: 
   - K线数据按年份分表: `kline_data_2024`, `kline_data_2025`
   - 交易记录按月分表: `tb_trade_202401`, `tb_trade_202402`

2. **索引优化**:
   - 为高频查询字段建立联合索引
   - `instrument_code + instrument_type` 联合索引

3. **缓存策略**:
   - 实时行情使用Redis
   - 基本信息使用本地缓存（Caffeine）+ Redis二级缓存

---

**设计优势**:
✅ 一期专注ETF，快速交付
✅ 架构支持多标的，二期无需重构
✅ 数据模型通用，扩展成本低
✅ 性能可优化，支持大规模数据

