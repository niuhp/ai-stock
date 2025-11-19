# AI Stock - ETF量化交易系统需求文档

## 文档版本
- **版本号**: v2.0
- **编写日期**: 2025-11-19
- **编写人**: AI Stock Team
- **最后更新**: 2025-11-19
- **版本说明**: 调整项目范围，前期专注于ETF量化交易

---

## 1. 项目概述

### 1.1 项目背景
ETF（交易型开放式指数基金）作为一种重要的投资工具，具有交易便捷、费用低廉、风险分散等优势。中国ETF市场规模持续增长，品种日益丰富。本项目旨在开发一个专注于ETF的量化交易系统，通过量化策略和AI技术，帮助投资者实现ETF的智能化交易。

### 1.2 项目目标
- 构建稳定的ETF数据采集和处理平台
- 开发多种ETF量化交易策略
- 提供ETF回测和实盘交易支持
- 实现策略自动化执行和监控
- 打造可扩展的量化交易框架

### 1.3 目标用户
- **个人投资者**: 需要量化策略辅助ETF投资
- **量化交易员**: 需要策略开发和回测平台
- **投资机构**: 需要批量ETF交易和管理工具

### 1.4 项目范围
**一期范围（ETF量化交易）**：
- ✅ ETF数据采集与存储（场内ETF）
- ✅ 量化策略开发框架
- ✅ 策略回测引擎
- ✅ 实时行情监控
- ✅ 信号生成与推送
- ✅ 交易记录管理
- ✅ 绩效分析与评估
- ❌ 不包含：实际交易执行、资金托管（需券商接口）

**后期扩展方向**：
- 个股量化交易
- 期权策略
- 跨市场套利
- 高频交易策略

---

## 2. 业务需求

### 2.1 核心业务流程

#### 2.1.1 数据采集流程
```
外部数据源 → ETF数据采集 → 数据清洗验证 → 数据存储 → 数据更新同步
```

#### 2.1.2 策略执行流程
```
实时行情 → 策略计算 → 信号生成 → 风险检查 → 信号推送 → 交易记录
```

#### 2.1.3 回测流程
```
选择策略 → 配置参数 → 加载历史数据 → 执行回测 → 生成回测报告 → 策略优化
```

#### 2.1.4 监控流程
```
实时监控 → 策略运行状态 → 持仓盈亏 → 风险指标 → 异常告警
```

### 2.2 业务规则
1. **交易时间规则**: 
   - 交易时间：工作日 9:30-11:30, 13:00-15:00
   - 集合竞价：9:15-9:25 (可接收数据，不产生信号)
   - 盘后数据更新：15:30-17:00

2. **数据更新规则**: 
   - 实时行情：延迟不超过3秒
   - 分钟K线：每分钟更新
   - 日K线：收盘后15分钟内更新
   - ETF净值：每日公布后更新
   - ETF成分股：定期更新（季度调整）

3. **策略信号规则**:
   - 开仓信号：满足策略条件 且 可用资金充足 且 未达到最大持仓限制
   - 平仓信号：满足止盈/止损条件 或 策略退出条件
   - 信号过滤：避免频繁交易（最小持仓时间限制）
   - 信号确认：连续N个周期满足条件（可配置）

4. **风险控制规则**:
   - 单只ETF持仓限制：不超过总资金的40%
   - 总持仓限制：不超过总资金的80%（保留20%现金）
   - 单日最大亏损：不超过总资金的3%
   - 单只ETF止损：下跌超过8%自动止损
   - 强制风险提示：投资有风险，策略仅供参考

5. **回测规则**:
   - 交易成本：手续费万分之2.5，印花税千分之1（仅卖出）
   - 滑点设置：默认0.1%
   - 最小交易单位：100股（1手）
   - 现金管理：不计算现金利息

---

## 3. 功能需求

### 3.1 用户管理模块

#### 3.1.1 用户注册与登录
- **FR-UM-001**: 支持手机号、邮箱注册
- **FR-UM-002**: 支持密码登录
- **FR-UM-003**: 用户信息完善（风险等级、初始资金等）

#### 3.1.2 账户管理
- **FR-UM-004**: 用户基本信息修改
- **FR-UM-005**: 密码修改、找回
- **FR-UM-006**: 账户注销
- **FR-UM-007**: 策略偏好设置（风险等级、回测参数等）

### 3.2 ETF数据管理模块

#### 3.2.1 数据采集
- **FR-DM-001**: 采集场内ETF基本信息（代码、名称、类型、跟踪指数、基金公司等）
- **FR-DM-002**: 采集ETF实时行情（最新价、涨跌幅、成交量、成交额、IOPV等）
- **FR-DM-003**: 采集K线数据（1分钟、5分钟、15分钟、30分钟、60分钟、日K、周K、月K）
- **FR-DM-004**: 采集ETF净值数据（单位净值、累计净值）
- **FR-DM-005**: 采集ETF规模数据（基金规模、份额变化）
- **FR-DM-006**: 采集ETF成分股信息（成分股列表、权重）
- **FR-DM-007**: 采集ETF申赎数据（申购赎回清单）
- **FR-DM-008**: 采集指数数据（跟踪指数行情、指数成分）

#### 3.2.2 数据管理
- **FR-DM-009**: 数据去重、清洗、验证
- **FR-DM-010**: 异常数据检测与修复
- **FR-DM-011**: 数据完整性检查
- **FR-DM-012**: 数据备份与恢复
- **FR-DM-013**: 数据更新日志记录
- **FR-DM-014**: 数据查询接口（按代码、类型、日期范围查询）

### 3.3 量化策略模块

#### 3.3.1 技术指标策略
- **FR-QS-001**: 均线策略（单均线、双均线、多均线）
- **FR-QS-002**: MACD策略（金叉死叉、零轴突破）
- **FR-QS-003**: KDJ策略（超买超卖、金叉死叉）
- **FR-QS-004**: RSI策略（超买超卖、背离）
- **FR-QS-005**: 布林带策略（突破、回归）
- **FR-QS-006**: ATR波动率策略

#### 3.3.2 趋势跟踪策略
- **FR-QS-007**: 动量策略（价格动量、成交量动量）
- **FR-QS-008**: 突破策略（价格突破、区间突破）
- **FR-QS-009**: 趋势线策略
- **FR-QS-010**: 通道策略（唐奇安通道、肯特纳通道）

#### 3.3.3 均值回归策略
- **FR-QS-011**: 配对交易策略（ETF与指数）
- **FR-QS-012**: 统计套利策略
- **FR-QS-013**: 折溢价策略（ETF折溢价套利）

#### 3.3.4 多因子策略
- **FR-QS-014**: 动量因子策略
- **FR-QS-015**: 波动率因子策略
- **FR-QS-016**: 成交量因子策略
- **FR-QS-017**: 多因子组合策略

#### 3.3.5 机器学习策略
- **FR-QS-018**: 分类预测策略（涨跌预测）
- **FR-QS-019**: 回归预测策略（价格预测）
- **FR-QS-020**: 强化学习策略
- **FR-QS-021**: 集成学习策略

#### 3.3.6 策略管理
- **FR-QS-022**: 策略创建与配置
- **FR-QS-023**: 策略参数优化
- **FR-QS-024**: 策略组合管理
- **FR-QS-025**: 策略版本管理
- **FR-QS-026**: 策略性能监控

### 3.4 回测引擎模块

#### 3.4.1 回测执行
- **FR-BT-001**: 选择策略和ETF标的
- **FR-BT-002**: 配置回测参数（时间范围、初始资金、手续费率等）
- **FR-BT-003**: 历史数据加载与验证
- **FR-BT-004**: 回测执行引擎（逐Bar/逐Tick回放）
- **FR-BT-005**: 交易成本计算（手续费、印花税、滑点）
- **FR-BT-006**: 持仓管理（开仓、平仓、持仓更新）
- **FR-BT-007**: 现金管理（可用资金、冻结资金）

#### 3.4.2 回测报告
- **FR-BT-008**: 收益指标（总收益、年化收益、月度收益）
- **FR-BT-009**: 风险指标（最大回撤、波动率、夏普比率、索提诺比率）
- **FR-BT-010**: 交易统计（交易次数、胜率、盈亏比、平均持仓时间）
- **FR-BT-011**: 收益曲线图表（净值曲线、回撤曲线）
- **FR-BT-012**: 交易记录明细
- **FR-BT-013**: 持仓分布分析
- **FR-BT-014**: 报告导出（PDF、Excel）

#### 3.4.3 策略优化
- **FR-BT-015**: 参数网格搜索
- **FR-BT-016**: 遗传算法优化
- **FR-BT-017**: 走势分析（WFA Walk Forward Analysis）
- **FR-BT-018**: 蒙特卡洛模拟
- **FR-BT-019**: 多策略组合回测
- **FR-BT-020**: 过拟合检测

### 3.5 持仓管理模块

#### 3.5.1 账户管理
- **FR-PM-001**: 账户创建（实盘/模拟）
- **FR-PM-002**: 账户资金管理（初始资金、可用资金、冻结资金）
- **FR-PM-003**: 多账户管理
- **FR-PM-004**: 账户权益计算（总资产、总盈亏）

#### 3.5.2 持仓管理
- **FR-PM-005**: 持仓查询（当前持仓列表）
- **FR-PM-006**: 持仓详情（成本、市值、盈亏、盈亏比例）
- **FR-PM-007**: 持仓历史记录
- **FR-PM-008**: 持仓统计分析

#### 3.5.3 交易记录
- **FR-PM-009**: 交易记录查询（买入、卖出记录）
- **FR-PM-010**: 交易详情（时间、价格、数量、手续费）
- **FR-PM-011**: 交易统计（日/周/月交易汇总）
- **FR-PM-012**: 成交回报处理

#### 3.5.4 绩效分析
- **FR-PM-013**: 收益分析（日收益、累计收益、年化收益）
- **FR-PM-014**: 风险指标（最大回撤、波动率、夏普比率）
- **FR-PM-015**: 净值曲线展示
- **FR-PM-016**: 与基准对比（沪深300、中证500等）
- **FR-PM-017**: 归因分析

### 3.6 实时监控模块

#### 3.6.1 行情监控
- **FR-MN-001**: ETF实时行情监控
- **FR-MN-002**: 大盘指数监控（沪深300、中证500、创业板指等）
- **FR-MN-003**: 持仓ETF实时盈亏
- **FR-MN-004**: 实时净值更新（账户总资产、可用资金）

#### 3.6.2 策略监控
- **FR-MN-005**: 策略运行状态监控
- **FR-MN-006**: 策略信号实时推送
- **FR-MN-007**: 策略绩效实时统计
- **FR-MN-008**: 策略异常检测与告警

#### 3.6.3 预警设置
- **FR-MN-009**: 价格预警（突破、跌破关键价位）
- **FR-MN-010**: 持仓预警（止损、止盈触发）
- **FR-MN-011**: 风控预警（持仓超限、回撤超限、单日亏损超限）
- **FR-MN-012**: 系统预警（数据异常、策略错误）

### 3.7 数据可视化模块

#### 3.7.1 图表展示
- **FR-VZ-001**: K线图展示（支持多周期切换）
- **FR-VZ-002**: 分时图展示
- **FR-VZ-003**: 技术指标叠加展示
- **FR-VZ-004**: 净值曲线图（账户、策略）
- **FR-VZ-005**: 回撤曲线图
- **FR-VZ-006**: 收益分布图（日收益、月收益）
- **FR-VZ-007**: 交易信号标注（买入卖出点）
- **FR-VZ-008**: 持仓分布图（饼图、柱状图）

#### 3.7.2 报告生成
- **FR-VZ-009**: 回测报告生成
- **FR-VZ-010**: 交易日报生成
- **FR-VZ-011**: 周度、月度绩效报告
- **FR-VZ-012**: 策略对比报告
- **FR-VZ-013**: 报告导出（PDF、Excel、HTML）

### 3.8 系统管理模块

#### 3.8.1 后台管理
- **FR-SY-001**: 管理员账户管理
- **FR-SY-002**: 用户管理（查询、禁用、删除）
- **FR-SY-003**: 数据源配置管理
- **FR-SY-004**: 策略模板管理
- **FR-SY-005**: 系统参数配置
- **FR-SY-006**: 任务调度管理（数据采集、策略执行）
- **FR-SY-007**: 日志管理（操作日志、错误日志、交易日志）

#### 3.8.2 统计分析
- **FR-SY-008**: 用户统计（活跃用户、策略使用情况）
- **FR-SY-009**: 策略统计（回测次数、实盘使用率）
- **FR-SY-010**: 系统性能监控（CPU、内存、数据库）
- **FR-SY-011**: API调用统计
- **FR-SY-012**: 数据质量监控（数据完整性、及时性）

---

## 4. 非功能需求

### 4.1 性能需求
- **NFR-PF-001**: 支持1000+并发用户
- **NFR-PF-002**: API响应时间 < 200ms（P95）
- **NFR-PF-003**: 实时行情延迟 < 2秒
- **NFR-PF-004**: 数据库查询响应时间 < 50ms（P95）
- **NFR-PF-005**: 策略计算单周期 < 100ms
- **NFR-PF-006**: 支持500+只ETF同时监控
- **NFR-PF-007**: 回测速度：1年历史数据 < 10秒

### 4.2 可用性需求
- **NFR-AV-001**: 系统可用性 ≥ 99.9%
- **NFR-AV-002**: 支持故障自动切换
- **NFR-AV-003**: 数据库主从复制，读写分离
- **NFR-AV-004**: 关键服务支持降级
- **NFR-AV-005**: 定期数据备份（每日全量 + 实时增量）

### 4.3 可扩展性需求
- **NFR-SC-001**: 支持微服务架构，模块独立部署
- **NFR-SC-002**: 支持横向扩展（增加服务器节点）
- **NFR-SC-003**: 支持插件式策略接入
- **NFR-SC-004**: 支持多种数据源切换
- **NFR-SC-005**: 预留扩展：个股量化、期权策略、跨市场套利
- **NFR-SC-006**: 策略框架支持自定义策略开发

### 4.4 安全性需求
- **NFR-SE-001**: 用户密码加密存储（BCrypt）
- **NFR-SE-002**: 支持HTTPS加密传输
- **NFR-SE-003**: API接口鉴权（JWT Token）
- **NFR-SE-004**: 敏感数据脱敏（手机号、身份证等）
- **NFR-SE-005**: SQL注入、XSS攻击防护
- **NFR-SE-006**: 接口限流防刷（令牌桶算法）
- **NFR-SE-007**: 操作审计日志记录
- **NFR-SE-008**: 数据访问权限控制

### 4.5 可维护性需求
- **NFR-MT-001**: 代码注释覆盖率 ≥ 30%
- **NFR-MT-002**: 单元测试覆盖率 ≥ 70%
- **NFR-MT-003**: 集成测试覆盖核心流程
- **NFR-MT-004**: 统一的日志规范和监控告警
- **NFR-MT-005**: 完善的API文档（Swagger）
- **NFR-MT-006**: 系统架构文档和开发手册

### 4.6 兼容性需求
- **NFR-CP-001**: 支持主流浏览器（Chrome、Firefox、Safari、Edge）
- **NFR-CP-002**: 支持移动端H5访问
- **NFR-CP-003**: 提供移动端SDK（iOS、Android）
- **NFR-CP-004**: 提供开放API供第三方集成

### 4.7 合规性需求
- **NFR-CM-001**: 遵守《证券法》相关规定
- **NFR-CM-002**: 明确风险提示，不承诺收益
- **NFR-CM-003**: 用户协议和免责声明
- **NFR-CM-004**: 遵守《网络安全法》和《数据安全法》
- **NFR-CM-005**: 用户隐私保护（GDPR/个人信息保护法）

---

## 5. 系统架构设计

### 5.1 总体架构

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端层                              │
│   Web端  │  移动端  │  API接口                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      API网关层                               │
│   身份认证  │  限流熔断  │  路由转发  │  日志监控            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                     业务服务层                               │
│  用户服务  │  数据服务  │  策略服务  │  回测服务            │
│  交易服务  │  监控服务  │  通知服务  │  报告服务            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    量化引擎层                                │
│  数据采集  │  策略引擎  │  回测引擎  │  信号引擎            │
│  风控引擎  │  绩效计算  │  指标计算  │  优化引擎            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    基础设施层                                │
│  MySQL  │  Redis  │  RabbitMQ  │  InfluxDB  │  监控告警      │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 核心服务模块

#### 5.2.1 用户服务 (user-service)
- 用户注册、登录、认证
- 用户信息管理
- 权限管理
- 账户管理

#### 5.2.2 数据服务 (data-service)
- ETF数据采集
- 行情数据存储
- 数据查询接口
- 数据同步服务

#### 5.2.3 策略服务 (strategy-service)
- 策略管理（CRUD）
- 策略执行调度
- 信号生成
- 策略监控

#### 5.2.4 回测服务 (backtest-service)
- 回测任务管理
- 回测引擎执行
- 回测报告生成
- 参数优化

#### 5.2.5 交易服务 (trade-service)
- 账户管理
- 持仓管理
- 交易记录
- 绩效分析

#### 5.2.6 监控服务 (monitor-service)
- 实时行情监控
- 策略运行监控
- 预警通知
- WebSocket推送

#### 5.2.7 报告服务 (report-service)
- 回测报告生成
- 交易报告生成
- 报告导出
- 历史报告查询

#### 5.2.8 通知服务 (notification-service)
- 消息推送（站内信、邮件）
- 信号通知
- 预警通知
- 消息记录查询

### 5.3 技术选型

| 类别 | 技术选型 | 说明 |
|------|---------|------|
| 开发语言 | Java 17 | LTS版本，性能和特性优秀 |
| 后端框架 | Spring Boot 3.x | 主流微服务框架 |
| 微服务治理 | Spring Cloud Alibaba | 服务注册、配置中心、网关 |
| ORM框架 | MyBatis Plus | 简化数据库操作 |
| 关系数据库 | MySQL 8.0 | 存储业务数据、用户数据 |
| 时序数据库 | InfluxDB / TimescaleDB | 存储K线、行情等时序数据 |
| 缓存 | Redis | 缓存热点数据、会话 |
| 消息队列 | RabbitMQ / Kafka | 异步处理、削峰填谷 |
| 搜索引擎 | Elasticsearch | 日志检索、股票搜索 |
| AI/ML | Python (Flask/FastAPI) | TensorFlow/PyTorch模型服务 |
| 任务调度 | XXL-Job | 定时任务管理 |
| API文档 | Swagger / Knife4j | 接口文档自动生成 |
| 日志 | SLF4J + Logback | 日志记录 |
| 监控 | Prometheus + Grafana | 系统监控告警 |
| 链路追踪 | SkyWalking / Zipkin | 分布式追踪 |
| 对象存储 | MinIO / 阿里云OSS | 文件、报告存储 |
| 容器化 | Docker + K8s | 容器编排部署 |

---

## 6. 数据模型设计

### 6.1 用户相关表

#### 6.1.1 用户表 (tb_user)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| username | VARCHAR(50) | 用户名 |
| password | VARCHAR(128) | 密码（加密） |
| phone | VARCHAR(20) | 手机号 |
| email | VARCHAR(100) | 邮箱 |
| real_name | VARCHAR(50) | 真实姓名 |
| id_card | VARCHAR(20) | 身份证号（加密） |
| avatar | VARCHAR(255) | 头像URL |
| status | TINYINT | 状态：0-禁用，1-正常 |
| vip_level | TINYINT | 会员等级 |
| vip_expire_time | DATETIME | 会员过期时间 |
| risk_preference | TINYINT | 风险偏好：1-保守，2-稳健，3-平衡，4-进取，5-激进 |
| investment_amount | DECIMAL(15,2) | 可投资金额 |
| create_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |
| deleted | TINYINT | 逻辑删除标记 |

#### 6.1.2 用户偏好表 (tb_user_preference)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| user_id | BIGINT | 用户ID |
| preferred_industries | VARCHAR(500) | 偏好行业（JSON数组） |
| preferred_concepts | VARCHAR(500) | 偏好概念（JSON数组） |
| preferred_market_cap | TINYINT | 偏好市值：1-大盘，2-中盘，3-小盘 |
| min_price | DECIMAL(10,2) | 最低股价 |
| max_price | DECIMAL(10,2) | 最高股价 |
| create_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |

### 6.2 股票相关表

#### 6.2.1 股票基本信息表 (tb_stock_info)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| stock_code | VARCHAR(10) | 股票代码 |
| stock_name | VARCHAR(50) | 股票名称 |
| exchange | VARCHAR(10) | 交易所：SH-上交所，SZ-深交所 |
| industry | VARCHAR(50) | 所属行业 |
| sector | VARCHAR(50) | 所属板块 |
| market_cap | DECIMAL(20,2) | 总市值 |
| listing_date | DATE | 上市日期 |
| is_st | TINYINT | 是否ST股：0-否，1-是 |
| status | TINYINT | 状态：0-停牌，1-正常，2-退市 |
| create_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |

#### 6.2.2 股票日K线表 (tb_stock_kline_day)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| stock_code | VARCHAR(10) | 股票代码 |
| trade_date | DATE | 交易日期 |
| open_price | DECIMAL(10,2) | 开盘价 |
| close_price | DECIMAL(10,2) | 收盘价 |
| high_price | DECIMAL(10,2) | 最高价 |
| low_price | DECIMAL(10,2) | 最低价 |
| volume | BIGINT | 成交量（手） |
| amount | DECIMAL(20,2) | 成交额（元） |
| change_rate | DECIMAL(10,4) | 涨跌幅 |
| turnover_rate | DECIMAL(10,4) | 换手率 |
| create_time | DATETIME | 创建时间 |

*注：其他周期K线表结构类似*

#### 6.2.3 股票实时行情表 (tb_stock_realtime)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| stock_code | VARCHAR(10) | 股票代码（主键） |
| current_price | DECIMAL(10,2) | 当前价 |
| change_rate | DECIMAL(10,4) | 涨跌幅 |
| change_amount | DECIMAL(10,2) | 涨跌额 |
| volume | BIGINT | 成交量 |
| amount | DECIMAL(20,2) | 成交额 |
| bid_price | DECIMAL(10,2) | 买一价 |
| ask_price | DECIMAL(10,2) | 卖一价 |
| high_price | DECIMAL(10,2) | 最高价 |
| low_price | DECIMAL(10,2) | 最低价 |
| open_price | DECIMAL(10,2) | 开盘价 |
| pre_close | DECIMAL(10,2) | 昨收价 |
| turnover_rate | DECIMAL(10,4) | 换手率 |
| update_time | DATETIME | 更新时间 |

#### 6.2.4 股票财务数据表 (tb_stock_financial)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| stock_code | VARCHAR(10) | 股票代码 |
| report_date | DATE | 报告期 |
| total_revenue | DECIMAL(20,2) | 营业总收入 |
| net_profit | DECIMAL(20,2) | 净利润 |
| total_assets | DECIMAL(20,2) | 总资产 |
| total_liabilities | DECIMAL(20,2) | 总负债 |
| net_assets | DECIMAL(20,2) | 净资产 |
| operating_cash_flow | DECIMAL(20,2) | 经营现金流 |
| eps | DECIMAL(10,4) | 每股收益 |
| roe | DECIMAL(10,4) | 净资产收益率 |
| pe_ratio | DECIMAL(10,2) | 市盈率 |
| pb_ratio | DECIMAL(10,2) | 市净率 |
| debt_ratio | DECIMAL(10,4) | 资产负债率 |
| gross_margin | DECIMAL(10,4) | 毛利率 |
| create_time | DATETIME | 创建时间 |

### 6.3 AI分析相关表

#### 6.3.1 股票AI评分表 (tb_stock_ai_score)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| stock_code | VARCHAR(10) | 股票代码 |
| score_date | DATE | 评分日期 |
| total_score | DECIMAL(5,2) | 总分（0-100） |
| technical_score | DECIMAL(5,2) | 技术面得分 |
| fundamental_score | DECIMAL(5,2) | 基本面得分 |
| capital_score | DECIMAL(5,2) | 资金面得分 |
| sentiment_score | DECIMAL(5,2) | 情绪面得分 |
| trend_direction | TINYINT | 趋势方向：1-上升，0-震荡，-1-下降 |
| risk_level | TINYINT | 风险等级：1-5 |
| confidence_level | DECIMAL(5,2) | 置信度 |
| model_version | VARCHAR(20) | 模型版本 |
| create_time | DATETIME | 创建时间 |

#### 6.3.2 买卖点推荐表 (tb_recommendation)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| stock_code | VARCHAR(10) | 股票代码 |
| recommend_type | TINYINT | 推荐类型：1-买入，2-卖出 |
| recommend_level | TINYINT | 推荐级别：1-强烈推荐，2-推荐，3-观察 |
| recommend_price | DECIMAL(10,2) | 推荐价格 |
| target_price | DECIMAL(10,2) | 目标价格 |
| stop_loss_price | DECIMAL(10,2) | 止损价格 |
| expected_return | DECIMAL(10,4) | 预期收益率 |
| risk_level | TINYINT | 风险等级 |
| reason | TEXT | 推荐理由 |
| valid_days | INT | 有效天数 |
| expire_time | DATETIME | 过期时间 |
| status | TINYINT | 状态：0-待确认，1-生效，2-已执行，3-已过期 |
| create_time | DATETIME | 创建时间 |

### 6.4 投资组合相关表

#### 6.4.1 投资组合表 (tb_portfolio)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| user_id | BIGINT | 用户ID |
| portfolio_name | VARCHAR(100) | 组合名称 |
| initial_amount | DECIMAL(15,2) | 初始金额 |
| current_amount | DECIMAL(15,2) | 当前金额 |
| available_cash | DECIMAL(15,2) | 可用现金 |
| total_profit | DECIMAL(15,2) | 总收益 |
| total_return_rate | DECIMAL(10,4) | 总收益率 |
| create_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |

#### 6.4.2 持仓明细表 (tb_position)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| portfolio_id | BIGINT | 组合ID |
| stock_code | VARCHAR(10) | 股票代码 |
| shares | INT | 持仓数量 |
| available_shares | INT | 可用数量 |
| avg_cost | DECIMAL(10,2) | 平均成本 |
| current_price | DECIMAL(10,2) | 当前价格 |
| market_value | DECIMAL(15,2) | 市值 |
| profit_loss | DECIMAL(15,2) | 浮动盈亏 |
| profit_loss_rate | DECIMAL(10,4) | 盈亏比例 |
| create_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |

#### 6.4.3 交易记录表 (tb_transaction)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| portfolio_id | BIGINT | 组合ID |
| stock_code | VARCHAR(10) | 股票代码 |
| transaction_type | TINYINT | 交易类型：1-买入，2-卖出 |
| shares | INT | 交易数量 |
| price | DECIMAL(10,2) | 成交价格 |
| amount | DECIMAL(15,2) | 交易金额 |
| commission | DECIMAL(10,2) | 手续费 |
| transaction_time | DATETIME | 交易时间 |
| create_time | DATETIME | 创建时间 |

### 6.5 监控预警相关表

#### 6.5.1 自选股表 (tb_watchlist)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| user_id | BIGINT | 用户ID |
| stock_code | VARCHAR(10) | 股票代码 |
| add_price | DECIMAL(10,2) | 添加时价格 |
| add_reason | VARCHAR(255) | 添加原因 |
| create_time | DATETIME | 创建时间 |

#### 6.5.2 预警规则表 (tb_alert_rule)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| user_id | BIGINT | 用户ID |
| stock_code | VARCHAR(10) | 股票代码 |
| alert_type | TINYINT | 预警类型：1-价格，2-涨跌幅，3-成交量，4-AI评分 |
| condition_type | TINYINT | 条件类型：1-大于，2-小于，3-等于 |
| threshold_value | DECIMAL(15,2) | 阈值 |
| is_enabled | TINYINT | 是否启用 |
| create_time | DATETIME | 创建时间 |

#### 6.5.3 预警记录表 (tb_alert_log)
| 字段名 | 类型 | 说明 |
|-------|------|------|
| id | BIGINT | 主键 |
| user_id | BIGINT | 用户ID |
| rule_id | BIGINT | 规则ID |
| stock_code | VARCHAR(10) | 股票代码 |
| alert_content | TEXT | 预警内容 |
| alert_time | DATETIME | 预警时间 |
| is_read | TINYINT | 是否已读 |
| create_time | DATETIME | 创建时间 |

---

## 7. API设计

### 7.1 API设计规范

#### 7.1.1 基本规范
- **协议**: HTTPS
- **请求格式**: JSON
- **响应格式**: JSON
- **字符编码**: UTF-8
- **RESTful风格**: 使用标准HTTP方法（GET、POST、PUT、DELETE）
- **版本控制**: `/api/v1/...`

#### 7.1.2 统一响应格式
```json
{
  "code": 200,
  "message": "success",
  "data": {},
  "timestamp": 1699999999999
}
```

#### 7.1.3 状态码定义
| 状态码 | 说明 |
|-------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 禁止访问 |
| 404 | 资源不存在 |
| 429 | 请求过于频繁 |
| 500 | 服务器内部错误 |

### 7.2 核心API接口

#### 7.2.1 用户相关API

**1. 用户注册**
```
POST /api/v1/users/register
Request Body:
{
  "phone": "13800138000",
  "password": "password123",
  "verifyCode": "123456"
}
```

**2. 用户登录**
```
POST /api/v1/users/login
Request Body:
{
  "username": "user123",
  "password": "password123"
}
Response:
{
  "code": 200,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "userInfo": {
      "id": 1,
      "username": "user123",
      "vipLevel": 2
    }
  }
}
```

**3. 获取用户信息**
```
GET /api/v1/users/profile
Headers: Authorization: Bearer {token}
```

**4. 更新用户偏好**
```
PUT /api/v1/users/preference
Headers: Authorization: Bearer {token}
Request Body:
{
  "riskPreference": 3,
  "preferredIndustries": ["电子", "医药"],
  "investmentAmount": 100000
}
```

#### 7.2.2 股票数据API

**1. 获取股票列表**
```
GET /api/v1/stocks?page=1&size=20&industry=电子&keyword=芯片
```

**2. 获取股票详情**
```
GET /api/v1/stocks/{stockCode}
Response:
{
  "code": 200,
  "data": {
    "stockCode": "000001",
    "stockName": "平安银行",
    "currentPrice": 12.50,
    "changeRate": 2.35,
    "volume": 123456789,
    "marketCap": 242500000000
  }
}
```

**3. 获取K线数据**
```
GET /api/v1/stocks/{stockCode}/kline?period=day&startDate=2024-01-01&endDate=2024-12-31
```

**4. 获取实时行情**
```
GET /api/v1/stocks/{stockCode}/realtime
WebSocket: ws://api.domain.com/v1/stocks/realtime?stockCodes=000001,600000
```

**5. 获取财务数据**
```
GET /api/v1/stocks/{stockCode}/financial?reportDate=2024-09-30
```

#### 7.2.3 AI分析API

**1. 获取股票AI评分**
```
GET /api/v1/analysis/{stockCode}/score
Response:
{
  "code": 200,
  "data": {
    "stockCode": "000001",
    "totalScore": 78.5,
    "technicalScore": 82,
    "fundamentalScore": 75,
    "capitalScore": 80,
    "sentimentScore": 77,
    "trendDirection": 1,
    "riskLevel": 2,
    "confidenceLevel": 0.85
  }
}
```

**2. 获取技术分析**
```
GET /api/v1/analysis/{stockCode}/technical
Response:
{
  "code": 200,
  "data": {
    "trend": "上升",
    "supportPrice": 12.00,
    "resistancePrice": 13.50,
    "indicators": {
      "macd": {"value": 0.25, "signal": "金叉"},
      "kdj": {"k": 65, "d": 58, "j": 72, "signal": "看多"},
      "rsi": {"value": 62, "signal": "强势"}
    }
  }
}
```

**3. 获取基本面分析**
```
GET /api/v1/analysis/{stockCode}/fundamental
```

**4. 批量分析**
```
POST /api/v1/analysis/batch
Request Body:
{
  "stockCodes": ["000001", "600000", "000858"]
}
```

#### 7.2.4 推荐API

**1. 获取买入推荐**
```
GET /api/v1/recommendations/buy?page=1&size=10
Headers: Authorization: Bearer {token}
Response:
{
  "code": 200,
  "data": {
    "total": 25,
    "recommendations": [
      {
        "stockCode": "000001",
        "stockName": "平安银行",
        "recommendLevel": 1,
        "recommendPrice": 12.50,
        "targetPrice": 15.00,
        "expectedReturn": 0.20,
        "riskLevel": 2,
        "reason": "技术面强势突破，资金持续流入...",
        "expireTime": "2024-12-20 15:00:00"
      }
    ]
  }
}
```

**2. 获取卖出推荐**
```
GET /api/v1/recommendations/sell?portfolioId=1
Headers: Authorization: Bearer {token}
```

**3. 获取个性化推荐**
```
GET /api/v1/recommendations/personalized
Headers: Authorization: Bearer {token}
```

#### 7.2.5 投资组合API

**1. 创建投资组合**
```
POST /api/v1/portfolios
Headers: Authorization: Bearer {token}
Request Body:
{
  "portfolioName": "价值投资组合",
  "initialAmount": 100000
}
```

**2. 获取组合列表**
```
GET /api/v1/portfolios
Headers: Authorization: Bearer {token}
```

**3. 获取组合详情**
```
GET /api/v1/portfolios/{portfolioId}
Headers: Authorization: Bearer {token}
Response:
{
  "code": 200,
  "data": {
    "id": 1,
    "portfolioName": "价值投资组合",
    "initialAmount": 100000,
    "currentAmount": 115230,
    "totalProfit": 15230,
    "totalReturnRate": 0.1523,
    "positions": [
      {
        "stockCode": "000001",
        "stockName": "平安银行",
        "shares": 1000,
        "avgCost": 11.50,
        "currentPrice": 12.50,
        "marketValue": 12500,
        "profitLoss": 1000,
        "profitLossRate": 0.0870
      }
    ]
  }
}
```

**4. 添加持仓**
```
POST /api/v1/portfolios/{portfolioId}/positions
Headers: Authorization: Bearer {token}
Request Body:
{
  "stockCode": "000001",
  "shares": 1000,
  "price": 12.50
}
```

**5. 获取组合分析**
```
GET /api/v1/portfolios/{portfolioId}/analysis
Headers: Authorization: Bearer {token}
```

#### 7.2.6 监控预警API

**1. 添加自选股**
```
POST /api/v1/watchlist
Headers: Authorization: Bearer {token}
Request Body:
{
  "stockCode": "000001",
  "addReason": "技术突破"
}
```

**2. 获取自选股列表**
```
GET /api/v1/watchlist
Headers: Authorization: Bearer {token}
```

**3. 创建预警规则**
```
POST /api/v1/alerts/rules
Headers: Authorization: Bearer {token}
Request Body:
{
  "stockCode": "000001",
  "alertType": 1,
  "conditionType": 1,
  "thresholdValue": 15.00
}
```

**4. 获取预警记录**
```
GET /api/v1/alerts/logs?page=1&size=20&isRead=0
Headers: Authorization: Bearer {token}
```

#### 7.2.7 报告API

**1. 生成个股分析报告**
```
POST /api/v1/reports/stock
Headers: Authorization: Bearer {token}
Request Body:
{
  "stockCode": "000001",
  "reportType": "comprehensive"
}
```

**2. 生成组合报告**
```
POST /api/v1/reports/portfolio
Headers: Authorization: Bearer {token}
Request Body:
{
  "portfolioId": 1,
  "reportPeriod": "monthly"
}
```

**3. 下载报告**
```
GET /api/v1/reports/{reportId}/download
Headers: Authorization: Bearer {token}
```

---

## 8. 安全性设计

### 8.1 身份认证
1. **JWT Token机制**: 
   - 登录成功后返回JWT Token
   - Token有效期：7天
   - 支持Token刷新机制
   - Token存储在Redis中，支持主动失效

2. **多因素认证**（可选）:
   - 敏感操作需要短信验证码
   - 支持绑定Google Authenticator

### 8.2 数据安全
1. **数据加密**:
   - 密码使用BCrypt加密存储
   - 敏感信息（身份证、银行卡）使用AES加密
   - HTTPS传输加密

2. **数据脱敏**:
   - 手机号脱敏：138****8000
   - 身份证脱敏：110***********1234
   - 日志中不记录敏感信息

3. **SQL注入防护**:
   - 使用MyBatis预编译语句
   - 参数验证和过滤
   - 禁止拼接SQL

### 8.3 访问控制
1. **RBAC权限模型**:
   - 角色：普通用户、VIP用户、管理员
   - 权限：API访问权限、数据访问权限
   - 资源：股票数据、AI分析、推荐服务

2. **接口限流**:
   - 基于用户的限流：普通用户100次/分钟，VIP用户500次/分钟
   - 基于IP的限流：防止恶意攻击
   - 滑动窗口算法

3. **数据权限**:
   - 用户只能访问自己的数据
   - 管理员可以查看所有数据（脱敏）

### 8.4 防护措施
1. **XSS防护**: 
   - 输入验证和过滤
   - 输出编码
   - CSP（Content Security Policy）

2. **CSRF防护**:
   - Token验证
   - 双重Cookie验证

3. **防爬虫**:
   - User-Agent检测
   - 访问频率限制
   - Captcha验证

---

## 9. 部署方案

### 9.1 部署架构

```
                    ┌──────────────┐
                    │   负载均衡    │
                    │   (Nginx)    │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ Gateway │       │ Gateway │       │ Gateway │
   │  节点1   │       │  节点2   │       │  节点3   │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ 业务服务 │       │ 业务服务 │       │ 业务服务 │
   │  集群    │       │  集群    │       │  集群    │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼───────┐
                    │  数据层集群   │
                    │ MySQL/Redis  │
                    └──────────────┘
```

### 9.2 环境规划

#### 9.2.1 开发环境 (DEV)
- 单机部署
- 使用本地数据库
- 热部署、快速迭代

#### 9.2.2 测试环境 (TEST)
- 与生产环境配置一致
- 独立数据库
- 用于功能测试、性能测试

#### 9.2.3 预生产环境 (STAGING)
- 完全模拟生产环境
- 用于上线前验证
- 灰度发布

#### 9.2.4 生产环境 (PROD)
- 高可用集群部署
- 主从数据库
- 完善的监控告警

### 9.3 服务器规划

| 服务 | 数量 | 配置 | 说明 |
|------|------|------|------|
| 负载均衡 | 2 | 4C8G | Nginx主备 |
| API网关 | 3 | 8C16G | Spring Cloud Gateway |
| 业务服务 | 10+ | 8C16G | 微服务集群，可动态扩容 |
| MySQL主库 | 1 | 16C64G | 主库 |
| MySQL从库 | 2 | 16C64G | 读从库 |
| Redis | 3 | 8C32G | 哨兵模式 |
| RabbitMQ | 3 | 8C16G | 集群模式 |
| Elasticsearch | 3 | 16C32G | 集群模式 |
| AI服务 | 3 | 16C64G (GPU) | Python AI服务 |
| 监控服务 | 2 | 8C16G | Prometheus + Grafana |

### 9.4 持续集成/持续部署 (CI/CD)

#### 9.4.1 CI/CD流程
```
代码提交 → Git Webhook → Jenkins触发 → 
编译打包 → 单元测试 → 构建Docker镜像 → 
推送镜像仓库 → K8s滚动更新 → 健康检查 → 完成
```

#### 9.4.2 发布策略
1. **蓝绿部署**: 减少停机时间
2. **金丝雀发布**: 先发布到少量节点验证
3. **灰度发布**: 按用户比例逐步发布
4. **回滚机制**: 发现问题快速回滚

### 9.5 容灾备份

#### 9.5.1 数据备份
- **数据库**: 每日全量备份 + 实时binlog备份
- **Redis**: RDB + AOF持久化
- **文件**: 对象存储自动备份
- **备份保留**: 全量备份保留30天，增量备份保留7天

#### 9.5.2 容灾方案
- **异地多活**: 多个可用区部署
- **数据库主从切换**: 自动failover
- **服务降级**: 非核心功能降级
- **限流熔断**: 保护核心服务

---

## 10. 监控告警

### 10.1 监控指标

#### 10.1.1 基础监控
- CPU使用率
- 内存使用率
- 磁盘使用率
- 网络流量

#### 10.1.2 应用监控
- QPS（每秒请求数）
- 响应时间（P50、P90、P99）
- 错误率
- 接口可用性

#### 10.1.3 业务监控
- 用户注册数
- 日活跃用户数
- 推荐准确率
- 数据采集成功率
- AI模型预测准确率

#### 10.1.4 数据库监控
- 连接数
- 慢查询
- 锁等待
- 主从延迟

### 10.2 告警策略

| 告警级别 | 触发条件 | 通知方式 |
|---------|---------|---------|
| P0（紧急） | 服务宕机、数据库故障 | 电话 + 短信 + 钉钉 |
| P1（严重） | 错误率>5%、响应时间>3s | 短信 + 钉钉 |
| P2（警告） | CPU>80%、内存>85% | 钉钉 + 邮件 |
| P3（提醒） | 磁盘>70%、慢查询增多 | 邮件 |

### 10.3 日志管理

#### 10.3.1 日志分类
- **访问日志**: 记录所有API请求
- **错误日志**: 记录异常和错误信息
- **业务日志**: 记录关键业务操作
- **审计日志**: 记录敏感操作

#### 10.3.2 日志收集
```
应用日志 → Filebeat → Kafka → Logstash → Elasticsearch → Kibana
```

---

## 11. 开发计划

### 11.1 项目阶段划分

#### 第一阶段：基础框架搭建（4周）
- **Week 1-2**: 
  - 项目初始化
  - 基础框架搭建（Spring Boot、数据库、Redis等）
  - 开发环境配置
  - 代码规范制定
  
- **Week 3-4**:
  - 用户模块开发（注册、登录、权限）
  - 数据库表设计与创建
  - 通用工具类开发
  - API网关搭建

#### 第二阶段：数据采集与存储（6周）
- **Week 5-7**:
  - 股票数据源接入
  - 数据采集模块开发
  - 数据清洗与验证
  - K线数据存储
  
- **Week 8-10**:
  - 实时行情采集
  - 财务数据采集
  - 技术指标计算
  - 数据定时更新任务

#### 第三阶段：AI分析模块（8周）
- **Week 11-14**:
  - AI模型框架搭建
  - 技术分析模型开发
  - 基本面分析模型开发
  - 模型训练与优化
  
- **Week 15-18**:
  - 资金面分析模型
  - 情绪分析模型
  - AI评分系统
  - 模型效果验证

#### 第四阶段：推荐与组合管理（6周）
- **Week 19-21**:
  - 买卖点推荐算法
  - 个性化推荐引擎
  - 推荐效果评估
  
- **Week 22-24**:
  - 投资组合管理模块
  - 收益计算与风险评估
  - 组合优化算法

#### 第五阶段：监控预警与可视化（4周）
- **Week 25-26**:
  - 实时监控模块
  - 预警规则引擎
  - WebSocket推送
  
- **Week 27-28**:
  - 数据可视化
  - 报告生成模块
  - 前端联调

#### 第六阶段：测试与优化（4周）
- **Week 29-30**:
  - 功能测试
  - 性能测试
  - 安全测试
  
- **Week 31-32**:
  - 性能优化
  - Bug修复
  - 文档完善

#### 第七阶段：上线准备（2周）
- **Week 33-34**:
  - 生产环境部署
  - 灰度发布
  - 监控告警配置
  - 正式上线

### 11.2 人员配置

| 角色 | 人数 | 职责 |
|------|------|------|
| 产品经理 | 1 | 需求分析、产品设计 |
| 项目经理 | 1 | 项目管理、进度把控 |
| 后端开发 | 4-6 | 业务逻辑开发 |
| 前端开发 | 2-3 | Web/移动端开发 |
| AI工程师 | 2-3 | AI模型开发与优化 |
| 测试工程师 | 2 | 功能测试、自动化测试 |
| 运维工程师 | 1-2 | 部署、监控、运维 |
| 数据工程师 | 1-2 | 数据采集、数据处理 |

### 11.3 里程碑

| 里程碑 | 时间 | 交付物 |
|--------|------|--------|
| M1: 基础框架完成 | Week 4 | 可运行的基础系统 |
| M2: 数据采集完成 | Week 10 | 稳定的数据采集系统 |
| M3: AI分析完成 | Week 18 | AI分析引擎 |
| M4: 推荐系统完成 | Week 24 | 完整的推荐系统 |
| M5: 系统集成完成 | Week 28 | 功能完整的系统 |
| M6: 测试完成 | Week 32 | 可上线的系统 |
| M7: 正式上线 | Week 34 | 生产环境运行 |

---

## 12. 风险评估

### 12.1 技术风险

| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|---------|
| 数据源不稳定 | 高 | 高 | 接入多个数据源，互为备份 |
| AI模型准确率不达标 | 中 | 高 | 持续优化模型，设置合理预期 |
| 并发性能问题 | 中 | 中 | 提前性能测试，预留扩容能力 |
| 第三方API限流 | 中 | 中 | 控制调用频率，增加缓存 |

### 12.2 业务风险

| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|---------|
| 合规风险 | 中 | 高 | 咨询法律顾问，明确免责声明 |
| 推荐失误导致用户损失 | 中 | 高 | 风险提示，不承诺收益 |
| 竞品冲击 | 高 | 中 | 持续创新，提升用户体验 |

### 12.3 项目风险

| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|---------|
| 人员流失 | 中 | 中 | 代码规范，文档完善 |
| 需求变更 | 高 | 中 | 敏捷开发，快速响应 |
| 进度延期 | 中 | 中 | 合理排期，留有缓冲 |

---

## 13. 成本预算

### 13.1 硬件成本（年）
- **服务器**: 30台 × 5000元/月 × 12月 = 180万
- **带宽**: 100Mbps × 3000元/月 × 12月 = 36万
- **对象存储**: 10TB × 100元/月 × 12月 = 1.2万
- **合计**: 约217万/年

### 13.2 软件成本（年）
- **数据源**: 50万/年
- **云服务**: 20万/年
- **监控工具**: 5万/年
- **合计**: 约75万/年

### 13.3 人力成本（年）
- **团队**: 15人 × 30万/年 = 450万/年

### 13.4 总成本
- **第一年**: 约750万
- **后续年份**: 约300万/年（运维成本）

---

## 14. 附录

### 14.1 术语表

| 术语 | 说明 |
|------|------|
| A股 | 中国大陆股票市场的人民币普通股票 |
| K线 | 蜡烛图，显示股票价格走势 |
| MACD | 指数平滑异同移动平均线，技术指标 |
| PE | 市盈率，Price-to-Earnings Ratio |
| ROE | 净资产收益率，Return on Equity |
| 北向资金 | 通过沪深港通流入A股的资金 |
| 融资融券 | 证券信用交易 |
| 龙虎榜 | 当日异动股票的成交明细 |

### 14.2 参考文档
- Spring Boot官方文档
- MyBatis Plus文档
- TensorFlow/PyTorch文档
- A股交易规则
- 证券投资分析相关书籍

### 14.3 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|---------|
| v1.0 | 2025-11-14 | AI Stock Team | 初始版本 |

---

**文档结束**

_注：本需求文档为初稿，后续会根据实际开发情况进行调整和完善。_

