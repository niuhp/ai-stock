# AI Stock - ETF量化交易系统需求文档（简化版）

## 6. 核心数据表设计

### 6.1 ETF基础数据表

```sql
-- ETF基本信息表
CREATE TABLE tb_etf_info (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    etf_code VARCHAR(10) NOT NULL UNIQUE COMMENT 'ETF代码',
    etf_name VARCHAR(100) NOT NULL COMMENT 'ETF名称',
    etf_type VARCHAR(50) COMMENT 'ETF类型',
    exchange VARCHAR(10) COMMENT '交易所',
    tracking_index VARCHAR(100) COMMENT '跟踪指数',
    fund_company VARCHAR(100) COMMENT '基金公司',
    listing_date DATE COMMENT '上市日期',
    status TINYINT DEFAULT 1 COMMENT '状态',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_code (etf_code),
    INDEX idx_type (etf_type)
);

-- ETF K线数据表（InfluxDB）
MEASUREMENT: etf_kline
TAGS: etf_code, period
FIELDS: open, high, low, close, volume, amount
TIME: timestamp

-- ETF实时行情表（Redis）
KEY: etf:realtime:{code}
FIELDS: price, change_rate, volume, amount, iopv, nav, update_time
```

### 6.2 策略相关表

```sql
-- 策略配置表
CREATE TABLE tb_strategy (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    strategy_name VARCHAR(100) NOT NULL COMMENT '策略名称',
    strategy_type VARCHAR(50) COMMENT '策略类型',
    strategy_code TEXT COMMENT '策略代码',
    parameters JSON COMMENT '策略参数',
    description TEXT COMMENT '策略描述',
    creator_id BIGINT COMMENT '创建人ID',
    status TINYINT DEFAULT 1 COMMENT '状态',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 回测记录表
CREATE TABLE tb_backtest (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    strategy_id BIGINT COMMENT '策略ID',
    etf_code VARCHAR(10) COMMENT 'ETF代码',
    start_date DATE COMMENT '开始日期',
    end_date DATE COMMENT '结束日期',
    initial_capital DECIMAL(15,2) COMMENT '初始资金',
    final_capital DECIMAL(15,2) COMMENT '最终资金',
    total_return DECIMAL(10,4) COMMENT '总收益率',
    annual_return DECIMAL(10,4) COMMENT '年化收益率',
    max_drawdown DECIMAL(10,4) COMMENT '最大回撤',
    sharpe_ratio DECIMAL(10,4) COMMENT '夏普比率',
    win_rate DECIMAL(5,2) COMMENT '胜率',
    trade_count INT COMMENT '交易次数',
    status TINYINT COMMENT '状态',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 6.3 交易账户表

```sql
-- 账户表
CREATE TABLE tb_account (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL COMMENT '用户ID',
    account_name VARCHAR(100) COMMENT '账户名称',
    account_type TINYINT COMMENT '账户类型:1-模拟,2-实盘',
    initial_capital DECIMAL(15,2) COMMENT '初始资金',
    available_cash DECIMAL(15,2) COMMENT '可用资金',
    market_value DECIMAL(15,2) COMMENT '持仓市值',
    total_assets DECIMAL(15,2) COMMENT '总资产',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 持仓表
CREATE TABLE tb_position (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_id BIGINT NOT NULL COMMENT '账户ID',
    etf_code VARCHAR(10) NOT NULL COMMENT 'ETF代码',
    shares INT COMMENT '持仓数量',
    avg_cost DECIMAL(10,4) COMMENT '平均成本',
    current_price DECIMAL(10,4) COMMENT '当前价格',
    market_value DECIMAL(15,2) COMMENT '市值',
    profit_loss DECIMAL(15,2) COMMENT '盈亏',
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 交易记录表
CREATE TABLE tb_trade (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_id BIGINT NOT NULL COMMENT '账户ID',
    strategy_id BIGINT COMMENT '策略ID',
    etf_code VARCHAR(10) NOT NULL COMMENT 'ETF代码',
    trade_type TINYINT COMMENT '交易类型:1-买入,2-卖出',
    shares INT COMMENT '交易数量',
    price DECIMAL(10,4) COMMENT '成交价格',
    amount DECIMAL(15,2) COMMENT '交易金额',
    commission DECIMAL(10,2) COMMENT '手续费',
    trade_time DATETIME COMMENT '交易时间',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

##7. 开发计划

### 7.1 项目阶段（简化版）

#### 第一阶段：基础框架（3周）
- Week 1: 项目搭建、用户模块、数据库设计
- Week 2: ETF数据采集模块
- Week 3: 基础API和权限管理

#### 第二阶段：策略引擎（5周）
- Week 4-5: 策略框架搭建，技术指标计算
- Week 6-7: 回测引擎开发
- Week 8: 策略模板实现（均线、MACD等）

#### 第三阶段：交易管理（3周）
- Week 9: 账户和持仓管理
- Week 10: 交易记录和绩效计算
- Week 11: 实时监控和预警

#### 第四阶段：可视化和优化（3周）
- Week 12: 数据可视化（K线图、净值曲线）
- Week 13: 报告生成
- Week 14: 性能优化和测试

#### 第五阶段：上线准备（2周）
- Week 15: 联调测试、Bug修复
- Week 16: 部署上线、文档完善

### 7.2 人员配置（最小团队）

- 后端开发：2人
- Python量化开发：1人
- 前端开发：1人
- 测试：1人
- 项目经理：1人（兼产品）

### 7.3 核心里程碑

| 里程碑 | 时间 | 交付物 |
|--------|------|--------|
| M1: 数据采集完成 | Week 3 | 稳定的ETF数据采集 |
| M2: 回测引擎完成 | Week 7 | 可用的回测系统 |
| M3: 交易管理完成 | Week 11 | 完整的交易管理 |
| M4: MVP完成 | Week 14 | 功能完整的MVP |
| M5: 正式上线 | Week 16 | 生产环境运行 |

---

## 8. 附录

### 8.1 ETF类型分类

- **宽基指数ETF**: 沪深300ETF、中证500ETF、创业板ETF
- **行业主题ETF**: 芯片ETF、医药ETF、新能源ETF
- **债券ETF**: 国债ETF、信用债ETF
- **商品ETF**: 黄金ETF、原油ETF
- **跨境ETF**: 恒生ETF、纳指ETF

### 8.2 常用技术指标

- **趋势类**: MA、EMA、MACD
- **震荡类**: RSI、KDJ、CCI
- **波动类**: ATR、BOLL
- **成交量类**: OBV、VOL

### 8.3 参考资料

- Backtrader官方文档: https://www.backtrader.com/
- TA-Lib文档: https://github.com/mrjbq7/ta-lib
- AKShare数据接口: https://akshare.akfamily.xyz/

---

**文档版本**: v2.0
**最后更新**: 2025-11-19
**状态**: ETF量化交易系统需求（简化版）

