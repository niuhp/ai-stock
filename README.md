# AI Stock - A股量化交易系统

## 项目简介

AI Stock 是一个面向A股市场的量化交易系统，支持股票、ETF、期权等多种标的交易。通过量化策略和数据分析技术，为投资者提供策略回测、实时监控和交易管理等功能。

**📌 一期范围**: 专注于场内ETF量化交易
**🚀 后期扩展**: 个股量化、期权策略、跨市场套利等

### 核心特性

- 📊 **数据管理**: 实时行情、历史K线、基本面数据采集（一期：ETF数据）
- 🎯 **策略开发**: 内置多种量化策略模板,支持自定义策略开发
- 🔄 **回测引擎**: 高性能回测引擎,支持多周期、多标的回测
- 💰 **持仓管理**: 模拟/实盘账户管理,持仓追踪,绩效分析
- 📈 **数据可视化**: K线图、净值曲线、回测报告等
- ⚡ **实时监控**: 策略运行监控、信号推送、风控预警
- 🔧 **扩展性强**: 架构支持股票、ETF、期权等多种标的

## 技术栈

### 后端技术
- **开发语言**: Java 17 + Python 3.10+
- **后端框架**: Spring Boot 3.x
- **微服务治理**: Spring Cloud Alibaba
- **ORM框架**: MyBatis Plus
- **数据库**: MySQL 8.0 (业务数据) + InfluxDB (时序数据)
- **缓存**: Redis
- **消息队列**: RabbitMQ
- **任务调度**: XXL-Job

### 量化技术
- **量化框架**: Backtrader / Zipline
- **技术指标库**: TA-Lib / pandas-ta
- **数据分析**: Pandas / NumPy
- **机器学习**: Scikit-learn (可选)

### 数据源
- **主要数据源**: AKShare
- **备用数据源**: Tushare Pro、东方财富API

## 项目文档

- 📋 [需求文档](./docs/requirements.md) - 系统功能需求和业务规则
- 🏗️ [架构文档](./docs/architecture.md) - 系统架构设计和技术选型
- 👨‍💻 [开发指南](./docs/development-guide.md) - 开发环境搭建和开发规范

## 快速开始

### 环境要求

- JDK 17+
- Python 3.10+
- MySQL 8.0+
- Redis 7.0+
- InfluxDB 2.7+

### 安装步骤

```bash
# 1. 克隆项目
git clone https://github.com/your-repo/ai-stock.git
cd ai-stock

# 2. 配置数据库
# 创建MySQL数据库: ai_stock
# 导入初始化SQL: docs/sql/init.sql

# 3. 配置Redis和InfluxDB
# 修改 application.yml 中的配置

# 4. 安装Python依赖
cd quant-engine
pip install -r requirements.txt

# 5. 启动后端服务
cd ../backend
./mvnw spring-boot:run

# 6. 启动前端服务 (如有)
cd ../frontend
npm install
npm run dev
```

## 项目结构

```
ai-stock/
├── backend/              # Java后端服务
│   ├── user-service/     # 用户服务
│   ├── data-service/     # 数据服务
│   ├── strategy-service/ # 策略服务
│   ├── backtest-service/ # 回测服务
│   ├── trade-service/    # 交易服务
│   └── monitor-service/  # 监控服务
├── quant-engine/         # Python量化引擎
│   ├── strategies/       # 策略库
│   ├── backtest/         # 回测引擎
│   └── indicators/       # 技术指标
├── frontend/             # 前端项目
├── docs/                 # 文档
└── sql/                  # 数据库脚本
```

## 功能模块

### 1. 数据管理（一期：ETF）
- **一期实现**:
  - 场内ETF基本信息采集
  - ETF实时行情订阅与推送
  - 多周期K线数据存储
  - ETF净值数据同步
- **二期扩展**:
  - 个股数据（基本信息、行情、财务数据）
  - 指数数据
  - 市场情绪数据

### 2. 量化策略（通用框架）
- 技术指标策略(均线、MACD、RSI等)
- 趋势跟踪策略(动量、突破等)
- 均值回归策略(配对交易、统计套利)
- 自定义策略开发框架
- **支持多标的**: 策略框架支持股票、ETF、期权等

### 3. 回测系统（通用引擎）
- 历史数据回测
- 参数优化(网格搜索、遗传算法)
- 回测报告生成(收益、风险、交易统计)
- 过拟合检测
- **多标的回测**: 支持单标的、多标的、投资组合回测

### 4. 交易管理（通用架构）
- 模拟/实盘账户管理
- 持仓追踪与成本计算
- 交易记录管理
- 绩效分析与归因
- **多标的支持**: 账户可同时持有ETF、股票等

### 5. 实时监控（通用监控）
- 策略运行状态监控
- 交易信号实时推送
- 风控预警(止损、回撤、持仓超限)
- WebSocket实时行情推送

## 开发计划

### 一期：ETF量化交易（16周）
- ✅ Phase 1: 项目搭建与基础框架（通用架构）
- 🔄 Phase 2: ETF数据采集模块
- 📅 Phase 3: 策略引擎与回测系统（通用框架）
- 📅 Phase 4: 交易管理与监控（通用模块）
- 📅 Phase 5: 前端界面与可视化

### 二期：扩展到个股（8-10周）
- 📅 个股数据采集（实时行情、财务数据）
- 📅 个股策略模板（基本面量化、因子选股等）
- 📅 投资组合管理
- 📅 风控模块增强

### 三期：高级功能（按需）
- 📅 期权策略
- 📅 跨市场套利
- 📅 高频交易策略
- 📅 AI/ML策略

## 贡献指南

欢迎贡献代码、提交Issue或提供建议！

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 风险提示

⚠️ **重要提示**: 本系统仅供学习和研究使用,不构成任何投资建议。

- 股市有风险,投资需谨慎
- 历史回测结果不代表未来收益
- 请根据自身风险承受能力谨慎投资
- 使用本系统产生的任何投资损失,开发者不承担责任

## 许可证

本项目采用 [MIT License](./LICENSE) 开源许可证。

## 联系方式

- Issue: https://github.com/your-repo/ai-stock/issues
- Email: your-email@example.com

---

**Star ⭐ 本项目,获取最新更新!**

