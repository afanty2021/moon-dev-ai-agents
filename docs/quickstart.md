# 快速开始指南

欢迎使用 Moon Dev AI Agents！这个指南将帮助您在5分钟内快速上手这个强大的AI驱动交易系统。

## 🚀 快速安装

### 前提条件
- Python 3.8+
- Git
- Conda (推荐) 或 pip

### 1. 克隆项目
```bash
git clone https://github.com/your-username/moon-dev-ai-agents.git
cd moon-dev-ai-agents
```

### 2. 设置环境
```bash
# 使用现有的 conda 环境 (推荐)
conda activate tflow

# 或创建新环境
conda create -n moon-dev python=3.9 -y
conda activate moon-dev
```

### 3. 安装依赖
```bash
pip install -r requirements.txt
```

### 4. 配置环境变量
```bash
# 复制环境变量模板
cp .env_example .env

# 编辑 .env 文件，添加你的API密钥
nano .env
```

必需的环境变量：
```bash
# AI模型密钥
ANTHROPIC_KEY=your_claude_api_key_here
OPENAI_KEY=your_openai_api_key_here

# 交易API密钥
BIRDEYE_API_KEY=your_birdeye_api_key_here
MOONDEV_API_KEY=your_moondev_api_key_here
```

## 🎯 首次运行

### 运行单个代理
```bash
# 运行交易代理
python src/agents/trading_agent.py

# 运行风险代理
python src/agents/risk_agent.py

# 运行策略代理
python src/agents/strategy_agent.py
```

### 运行主系统
```bash
# 运行完整的编排系统
python src/main.py
```

## 🤖 基础AI对话

### 使用聊天代理
```python
from src.agents.chat_agent import ChatAgent

# 创建聊天代理
chat = ChatAgent()

# 发送消息
response = chat.send_message("分析比特币的短期走势")
print(response)
```

### 使用ModelFactory直接
```python
from src.models.model_factory import ModelFactory

# 创建模型工厂
factory = ModelFactory()

# 获取Claude模型
claude = factory.get_model('claude')

# 生成响应
response = claude.generate_response(
    system_prompt="你是专业的金融分析师",
    user_content="现在是否应该买入以太坊？"
)
print(response)
```

## 📊 市场数据分析

### 获取代币信息
```python
from src.nice_funcs import token_overview, token_price

# 获取代币概览
overview = token_overview("So11111111111111111111111111111111111111112")  # SOL
print(f"当前价格: ${overview['price']}")
print(f"24h变化: {overview['price_change_24h']}%")

# 获取当前价格
price = token_price("So11111111111111111111111111111111111111112")
print(f"SOL价格: ${price}")
```

### 获取市场数据
```python
from src.data.ohlcv_collector import OHLCVCollector

# 创建数据收集器
collector = OHLCVCollector()

# 获取OHLCV数据
ohlcv = collector.get_ohlcv_data(
    token="So11111111111111111111111111111111111111112",
    timeframe='1H',
    days_back=7
)

print(f"获取到 {len(ohlcv)} 条K线数据")
print(ohlcv.head())
```

## 🔄 群体AI决策

### 使用SwarmAgent
```python
from src.agents.swarm_agent import SwarmAgent

# 创建群体代理
swarm = SwarmAgent()

# 查询多个AI模型
result = swarm.query(
    prompt="分析当前加密货币市场状况，给出投资建议",
    system_prompt="你是专业的投资顾问"
)

print(f"共识建议: {result['consensus_summary']}")
print(f"置信度: {result['confidence']:.2f}")

# 查看各模型投票
print("投票结果:")
for model_response in result['individual_responses']:
    print(f"{model_response['model']}: {model_response['action']}")
```

## ⚡ 快速交易示例

### 简单交易信号
```python
from src.agents.trading_agent import TradingAgent

# 创建交易代理
agent = TradingAgent()

# 分析市场
result = agent.analyze_market_data("BTC")

if result['action'] == 'BUY':
    print(f"建议买入，置信度: {result['confidence']}%")
    print(f"理由: {result['reasoning']}")
    # 注意：这里是演示，实际交易需要验证
else:
    print("当前不建议交易")
```

### 风险检查
```python
from src.agents.risk_agent import RiskAgent

# 创建风险代理
risk_agent = RiskAgent()

# 检查当前风险状态
risk_status = risk_agent.check_pnl_limits()

if risk_status['limit_breached']:
    print("⚠️ 风险限制已触发")
    print(f"风险类型: {risk_status['limit_type']}")
else:
    print("✅ 风险状态正常")
```

## 📈 监控你的投资

### 获取持仓状态
```python
from src.nice_funcs import get_position

# 获取当前持仓
position = get_position("BTC-USDT")

if position:
    print(f"持仓数量: {position['position_amount']}")
    print(f"当前价格: ${position['mark_price']}")
    print(f"盈亏: ${position['pnl']}")
    print(f"盈亏百分比: {position['pnl_percentage']}%")
else:
    print("当前无持仓")
```

### 监控盈亏
```python
# 监控特定代币的盈亏
agent = TradingAgent()
is_closed = agent.monitor_position_pnl("BTC", check_interval=5)

if is_closed:
    print("持仓已自动平仓")
else:
    print("继续监控中...")
```

## 🎛️ 配置你的系统

### 基本配置编辑
编辑 `src/config.py` 来调整系统参数：

```python
# 交易配置
USD_SIZE = 100  # 单次交易金额
MAX_POSITION_PERCENTAGE = 30  # 最大仓位百分比
LEVERAGE = 1  # 杠杆倍数

# 风险配置
MAX_LOSS_USD = 25  # 最大损失限制
MAX_GAIN_USD = 25  # 最大收益限制
STOP_LOSS_PERCENTAGE = 5.0  # 止损百分比

# AI配置
AI_MODEL = "claude-3-5-haiku-20240307"  # 默认AI模型
AI_TEMPERATURE = 0.7  # 创意程度
AI_MAX_TOKENS = 1024  # 最大token数
```

### 使用环境变量
```python
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 使用环境变量
api_key = os.getenv('ANTHROPIC_KEY')
max_risk = float(os.getenv('MAX_RISK_PERCENTAGE', '2.0'))
```

## 🔧 常见任务

### 1. 创建自定义策略
```python
from src.strategies.base_strategy import BaseStrategy

class MyStrategy(BaseStrategy):
    name = "my_custom_strategy"
    description = "我的自定义交易策略"

    def generate_signals(self, token, market_data):
        # 实现你的策略逻辑
        return {
            'action': 'BUY',
            'confidence': 0.8,
            'reasoning': '基于技术指标的买入信号'
        }

# 使用策略
strategy = MyStrategy()
signals = strategy.generate_signals('BTC', market_data)
```

### 2. 批量分析多个代币
```python
tokens = [
    "So11111111111111111111111111111111111111112",  # SOL
    "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",  # USDC
    "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB"   # Ray
]

agent = TradingAgent()

for token in tokens:
    result = agent.analyze_market_data(token)
    print(f"{token}: {result['action']} (置信度: {result['confidence']}%)")
```

### 3. 获取情绪分析
```python
from src.agents.sentiment_agent import SentimentAgent

sentiment = SentimentAgent()

# 分析Twitter情绪
sentiment_score = sentiment.analyze_sentiment("比特币今天大涨！")
print(f"情绪得分: {sentiment_score}")

# 运行完整情绪分析
sentiment.run()  # 这会分析最新推文并生成报告
```

## 🚨 重要提醒

### ⚠️ 风险警告
- 这是实验性项目，投资有风险
- 在使用真实资金前请充分测试
- 始终设置适当的止损和风险限制
- 不要投入超过你承受损失的资金

### 🛡️ 安全建议
- 妥善保管API密钥
- 使用测试环境进行实验
- 定期备份重要数据
- 监控系统日志和交易记录

### 📚 继续学习
- 阅读详细的[文档](README.md)
- 观看[视频教程](https://youtube.com/@moondev)
- 加入[Discord社区](https://discord.gg/moondev)
- 查看GitHub[Issues](https://github.com/afanty2021/moon-dev-ai-agents/issues)

## 🆘 获取帮助

如果遇到问题：

1. 查看[故障排除指南](troubleshooting.md)
2. 搜索现有的GitHub Issues
3. 在Discord社区提问
4. 创建新的Issue描述问题

---

## 🎉 恭喜！

你已经完成了 Moon Dev AI Agents 的快速开始！现在你可以：

- ✅ 运行AI代理进行市场分析
- ✅ 使用群体智能获得交易建议
- ✅ 配置和自定义交易策略
- ✅ 监控风险和投资表现

继续探索完整的[文档](README.md)来了解更多高级功能和使用技巧！

**下一步**: 阅读[系统架构](architecture.md)了解系统设计，或者查看[API参考](api-reference.md)学习更多API用法。