# API 参考文档

## 概述

Moon Dev AI Agents 提供了完整的 API 接口，支持交易执行、数据分析、风险管理和 AI 模型集成。本文档详细介绍了所有可用的 API 端点和方法。

## 目录

1. [核心交易 API](#核心交易-api)
2. [数据获取 API](#数据获取-api)
3. [风险管理 API](#风险管理-api)
4. [AI 模型集成 API](#ai-模型集成-api)
5. [配置管理 API](#配置管理-api)
6. [工具函数 API](#工具函数-api)
7. [错误处理](#错误处理)

## 核心交易 API

### ExchangeManager

`ExchangeManager` 是所有交易操作的核心管理器，支持多个交易所的统一接口。

#### 初始化
```python
from src.exchange_manager import ExchangeManager

# 创建交易所管理器实例
em = ExchangeManager()
```

#### 市场买入

```python
def market_buy(symbol: str, amount: float, leverage: int = None) -> dict:
    """
    执行市价买入订单

    Args:
        symbol: 交易对符号 (例如: 'BTC-USDT')
        amount: 买入金额 (USD)
        leverage: 杠杆倍数 (可选)

    Returns:
        dict: {
            'success': bool,
            'order_id': str,
            'executed_price': float,
            'executed_amount': float,
            'fee': float,
            'message': str
        }

    Raises:
        InsufficientBalanceError: 余额不足
        ExchangeError: 交易所错误
        NetworkError: 网络连接错误
    """
```

**使用示例**:
```python
result = em.market_buy('SOL-USDT', 100, leverage=5)
if result['success']:
    print(f"买入成功: {result['executed_amount']} SOL @ ${result['executed_price']}")
else:
    print(f"买入失败: {result['message']}")
```

#### 市场卖出

```python
def market_sell(symbol: str, amount: float, leverage: int = None) -> dict:
    """
    执行市价卖出订单

    Args:
        symbol: 交易对符号
        amount: 卖出数量
        leverage: 杠杆倍数 (可选)

    Returns:
        dict: 与 market_buy 相同的结构
    """
```

#### AI 辅助入场

```python
def ai_entry(symbol: str, amount: float, strategy: str = 'default') -> dict:
    """
    AI 辅助的智能入场

    Args:
        symbol: 交易对符号
        amount: 交易金额
        strategy: 使用的策略名称

    Returns:
        dict: {
            'success': bool,
            'action': 'BUY' | 'SELL' | 'HOLD',
            'confidence': float,  # 0-1
            'reasoning': str,
            'entry_price': float,
            'position_size': float
        }
    """
```

#### 批量平仓

```python
def chunk_kill(symbol: str, max_size: float = None, slippage: int = 199) -> dict:
    """
    分批平仓以减少市场冲击

    Args:
        symbol: 交易对符号
        max_size: 单笔最大交易金额
        slippage: 滑点容忍度 (基点, 199=1.99%)

    Returns:
        dict: {
            'success': bool,
            'total_closed': float,
            'average_price': float,
            'total_fees': float,
            'transactions': list
        }
    """
```

### TradingAgent

`TradingAgent` 是主要的交易决策和执行代理。

#### 初始化
```python
from src.agents.trading_agent import TradingAgent

# 创建交易代理实例
agent = TradingAgent()
```

#### 市场数据分析

```python
def analyze_market_data(token: str, market_data: dict = None) -> dict:
    """
    分析市场数据并生成交易建议

    Args:
        token: 代币地址或符号
        market_data: 市场数据字典 (可选，会自动获取)

    Returns:
        dict: {
            'token': str,
            'action': 'BUY' | 'SELL' | 'HOLD',
            'confidence': int,  # 0-100
            'reasoning': str,
            'entry_price': float,
            'position_size': float,
            'stop_loss': float,
            'take_profit': float,
            'leverage': int,
            'risk_reward_ratio': float
        }
    """
```

**使用示例**:
```python
# 分析单个代币
result = agent.analyze_market_data('9BB6NFEcjBCtnNLFko2FqVQBq8HHM13kCyYcdQbgpump')

if result['action'] == 'BUY':
    print(f"建议买入 {result['token']}")
    print(f"置信度: {result['confidence']}%")
    print(f"理由: {result['reasoning']}")

    # 执行交易
    agent.execute_trade(
        action=result['action'],
        token=result['token'],
        amount=result['position_size'],
        leverage=result['leverage']
    )
```

#### 执行交易

```python
def execute_trade(action: str, token: str, amount: float, leverage: int = None) -> bool:
    """
    执行交易操作

    Args:
        action: 交易动作 ('BUY', 'SELL', 'HOLD')
        token: 代币标识符
        amount: 交易金额
        leverage: 杠杆倍数

    Returns:
        bool: 执行是否成功
    """
```

#### 持仓监控

```python
def get_position_status(token: str) -> dict:
    """
    获取指定代币的持仓状态

    Args:
        token: 代币标识符

    Returns:
        dict: {
            'position_amount': float,
            'entry_price': float,
            'mark_price': float,
            'pnl': float,
            'pnl_percentage': float,
            'is_long': bool,
            'leverage': int,
            'margin_used': float,
            'margin_available': float,
            'liquidation_price': float
        }
    """
```

#### 盈亏监控

```python
def monitor_position_pnl(token: str, check_interval: int = 5) -> bool:
    """
    监控持仓盈亏并自动执行止损/止盈

    Args:
        token: 监控的代币
        check_interval: 检查间隔 (秒)

    Returns:
        bool: 是否已平仓
    """
```

## 数据获取 API

### OHLCV 数据

#### OHLCVCollector

```python
from src.data.ohlcv_collector import OHLCVCollector

collector = OHLCVCollector()
```

#### 获取历史数据

```python
def get_ohlcv_data(
    token: str,
    timeframe: str = '1H',
    days_back: int = 7,
    limit: int = 1000
) -> pd.DataFrame:
    """
    获取 OHLCV (开高低收成交量) 数据

    Args:
        token: 代币地址
        timeframe: 时间框架 ('1m', '5m', '15m', '1H', '4H', '1D')
        days_back: 回溯天数
        limit: 最大数据点数量

    Returns:
        pd.DataFrame: 包含列 ['timestamp', 'open', 'high', 'low', 'close', 'volume']
    """
```

**使用示例**:
```python
# 获取 SOL 的小时数据
ohlcv = collector.get_ohlcv_data('So11111111111111111111111111111111111111112', '1H', 7)

# 计算移动平均线
ohlcv['sma_20'] = ohlcv['close'].rolling(window=20).mean()
ohlcv['sma_50'] = ohlcv['close'].rolling(window=50).mean()

# 获取最新价格
latest_price = ohlcv.iloc[-1]['close']
print(f"当前 SOL 价格: ${latest_price}")
```

### 实时数据

#### 获取代币概览

```python
from src.nice_funcs import token_overview

def token_overview(token_address: str) -> dict:
    """
    获取代币的完整概览信息

    Args:
        token_address: 代币合约地址

    Returns:
        dict: {
            'symbol': str,
            'name': str,
            'price': float,
            'price_change_24h': float,
            'volume_24h': float,
            'market_cap': float,
            'liquidity': float,
            'holders': int,
            'total_supply': float,
            'circulating_supply': float,
            'created_at': str
        }
    """
```

#### 获取当前价格

```python
from src.nice_funcs import token_price

def token_price(token_address: str) -> float:
    """
    获取代币当前价格

    Args:
        token_address: 代币合约地址

    Returns:
        float: 当前价格
    """
```

#### 获取持仓信息

```python
from src.nice_funcs import get_position

def get_position(symbol: str = None) -> dict:
    """
    获取当前持仓信息

    Args:
        symbol: 交易对符号 (可选)

    Returns:
        dict: 持仓详情或所有持仓列表
    """
```

### 市场数据 API

#### MoonDev API

```python
from src.agents.api import MoonDevAPI

api = MoonDevAPI()
```

##### 获取清算数据

```python
def get_liquidation_data(timeframe: str = '1h') -> dict:
    """
    获取清算数据

    Args:
        timeframe: 时间框架

    Returns:
        dict: {
            'total_liquidated': float,
            'long_liquidations': float,
            'short_liquidations': float,
            'largest_liquidation': dict,
            'liquidations': list
        }
    """
```

##### 获取资金费率

```python
def get_funding_data() -> dict:
    """
    获取资金费率数据

    Returns:
        dict: {
            'funding_rate': float,
            'predicted_funding_rate': float,
            'funding_interval': str,
            'next_funding_time': str
        }
    """
```

##### 获取持仓兴趣

```python
def get_oi_data(symbol: str = None) -> dict:
    """
    获取持仓兴趣数据

    Args:
        symbol: 交易对符号 (可选)

    Returns:
        dict: 持仓兴趣数据
    """
```

## 风险管理 API

### RiskAgent

```python
from src.agents.risk_agent import RiskAgent

risk_agent = RiskAgent()
```

#### 检查盈亏限制

```python
def check_pnl_limits() -> dict:
    """
    检查是否达到盈亏限制

    Returns:
        dict: {
            'limit_breached': bool,
            'limit_type': 'MAX_LOSS' | 'MAX_GAIN',
            'current_pnl': float,
            'limit_value': float,
            'should_close_all': bool,
            'ai_recommendation': str
        }
    """
```

#### 风险评估

```python
def assess_risk(position_size: float, leverage: int, symbol: str) -> dict:
    """
    评估交易风险

    Args:
        position_size: 仓位大小
        leverage: 杠杆倍数
        symbol: 交易对

    Returns:
        dict: {
            'risk_score': float,  # 0-1
            'max_loss': float,
            'liquidation_price': float,
            'recommended_leverage': int,
            'risk_level': 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL'
        }
    """
```

#### 紧急平仓

```python
def emergency_close_all(reason: str = "Risk limit breach") -> bool:
    """
    紧急平掉所有持仓

    Args:
        reason: 平仓原因

    Returns:
        bool: 是否成功平仓
    """
```

## AI 模型集成 API

### ModelFactory

```python
from src.models.model_factory import ModelFactory

factory = ModelFactory()
```

#### 获取模型实例

```python
def get_model(model_type: str, model_name: str = None) -> BaseModel:
    """
    获取指定类型的 AI 模型实例

    Args:
        model_type: 模型类型 ('claude', 'openai', 'groq', 'gemini', 'deepseek', 'ollama')
        model_name: 特定模型名称 (可选)

    Returns:
        BaseModel: 模型实例

    Raises:
        ModelNotFoundError: 模型不存在
        ConfigurationError: 配置错误
    """
```

**支持的模型类型**:

| 模型类型 | 默认模型 | 特点 | 适用场景 |
|---------|---------|------|---------|
| claude | claude-3-5-haiku-latest | 平衡性能和成本 | 通用分析、决策 |
| openai | gpt-4o | 最新GPT模型 | 复杂推理、创意 |
| groq | mixtral-8x7b-32768 | 极快速度 | 实时分析、高频交易 |
| gemini | gemini-2.5-flash | Google最新模型 | 多模态处理 |
| deepseek | deepseek-reasoner | 增强推理 | 复杂逻辑、策略 |
| ollama | llama3.2 | 本地部署 | 隐私敏感场景 |

#### 生成响应

```python
def generate_response(
    system_prompt: str,
    user_content: str,
    temperature: float = 0.7,
    max_tokens: int = 1024
) -> str:
    """
    生成 AI 响应

    Args:
        system_prompt: 系统提示词
        user_content: 用户输入内容
        temperature: 创意程度 (0-1)
        max_tokens: 最大令牌数

    Returns:
        str: AI 生成的响应
    """
```

**使用示例**:
```python
# 获取 Claude 模型
claude_model = factory.get_model('claude')

# 分析市场
system_prompt = "你是一个专业的加密货币交易分析师。"
user_content = f"""
分析以下代币的短期走势:
代币: SOL
当前价格: ${token_price}
24小时变化: {price_change}%
成交量: ${volume}
"""

response = claude_model.generate_response(
    system_prompt=system_prompt,
    user_content=user_content,
    temperature=0.3,  # 低温度确保逻辑性
    max_tokens=500
)

print("AI 分析结果:", response)
```

### SwarmAgent (群体智能)

```python
from src.agents.swarm_agent import SwarmAgent

swarm = SwarmAgent()
```

#### 群体查询

```python
def query(
    prompt: str,
    system_prompt: str = None,
    models: list = None
) -> dict:
    """
    向多个 AI 模型发起查询并计算共识

    Args:
        prompt: 查询内容
        system_prompt: 系统提示词 (可选)
        models: 指定模型列表 (可选，默认使用6个模型)

    Returns:
        dict: {
            'consensus_summary': str,
            'consensus_action': 'BUY' | 'SELL' | 'HOLD',
            'confidence': float,
            'individual_responses': [
                {
                    'model': str,
                    'response': str,
                    'action': str,
                    'confidence': float
                }
            ],
            'vote_counts': {
                'BUY': int,
                'SELL': int,
                'HOLD': int
            }
        }
    """
```

**使用示例**:
```python
# 群体决策是否买入 BTC
result = swarm.query(
    prompt="现在是否应该买入比特币？考虑当前市场状况和技术指标。",
    system_prompt="你是专业的加密货币交易顾问，请给出明确的买入、卖出或持有建议。"
)

print(f"共识建议: {result['consensus_action']}")
print(f"置信度: {result['confidence']:.2f}")
print(f"共识总结: {result['consensus_summary']}")

# 查看各模型的投票
print("投票分布:", result['vote_counts'])
```

## 配置管理 API

### 配置访问

```python
from src.config import *

# 读取配置值
print(f"交易金额: ${usd_size}")
print(f"杠杆倍数: {LEVERAGE}")
print(f"监控代币数量: {len(MONITORED_TOKENS)}")
```

### 动态配置更新

```python
from src.config import update_config

def update_config(key: str, value: any) -> bool:
    """
    动态更新配置值

    Args:
        key: 配置键
        value: 新值

    Returns:
        bool: 更新是否成功
    """
```

### 配置验证

```python
from src.config import validate_config

def validate_config() -> dict:
    """
    验证配置的有效性

    Returns:
        dict: {
            'valid': bool,
            'errors': list,
            'warnings': list
        }
    """
```

## 工具函数 API

### 技术指标

```python
from src.nice_funcs import calculate_indicators

def calculate_indicators(df: pd.DataFrame, indicators: list = None) -> pd.DataFrame:
    """
    计算技术指标

    Args:
        df: OHLCV 数据 DataFrame
        indicators: 要计算的指标列表

    Returns:
        pd.DataFrame: 包含技术指标的 DataFrame
    """
```

**支持的指标**:
- RSI (相对强弱指数)
- MACD (移动平均收敛散度)
- Bollinger Bands (布林带)
- Moving Averages (移动平均线)
- Stochastic (随机指标)
- ADX (平均方向指数)

### 仓位计算

```python
from src.nice_funcs import calculate_position_size

def calculate_position_size(
    account_balance: float,
    risk_percentage: float,
    entry_price: float,
    stop_loss_price: float
) -> float:
    """
    计算推荐的仓位大小

    Args:
        account_balance: 账户余额
        risk_percentage: 风险百分比
        entry_price: 入场价格
        stop_loss_price: 止损价格

    Returns:
        float: 推荐仓位大小
    """
```

### PnL 计算

```python
from src.nice_funcs import calculate_pnl

def calculate_pnl(
    entry_price: float,
    exit_price: float,
    position_size: float,
    is_long: bool = True,
    leverage: int = 1
) -> dict:
    """
    计算盈亏

    Args:
        entry_price: 入场价格
        exit_price: 出场价格
        position_size: 仓位大小
        is_long: 是否多头
        leverage: 杠杆倍数

    Returns:
        dict: {
            'pnl_amount': float,
            'pnl_percentage': float,
            'roi': float,
            'fees': float
        }
    """
```

## 错误处理

### 异常类型

```python
# 自定义异常类
class TradingError(Exception):
    """交易相关错误的基类"""
    pass

class InsufficientBalanceError(TradingError):
    """余额不足错误"""
    pass

class ExchangeError(TradingError):
    """交易所错误"""
    pass

class NetworkError(TradingError):
    """网络连接错误"""
    pass

class ModelError(Exception):
    """AI模型相关错误"""
    pass

class ConfigurationError(Exception):
    """配置错误"""
    pass
```

### 错误处理模式

```python
# 推荐的错误处理模式
def safe_trading_operation(symbol: str, amount: float) -> bool:
    """
    安全的交易操作，包含完整错误处理
    """
    try:
        # 预检查
        balance = get_account_balance()
        if balance < amount:
            raise InsufficientBalanceError(f"余额不足: ${balance} < ${amount}")

        # 执行交易
        result = market_buy(symbol, amount)

        if not result['success']:
            raise ExchangeError(f"交易失败: {result['message']}")

        print(f"交易成功: {result}")
        return True

    except InsufficientBalanceError as e:
        print(f"❌ 余额错误: {e}")
        return False

    except ExchangeError as e:
        print(f"❌ 交易所错误: {e}")
        return False

    except NetworkError as e:
        print(f"❌ 网络错误: {e}")
        # 可以实现重试逻辑
        return False

    except Exception as e:
        print(f"❌ 未知错误: {e}")
        # 记录错误日志
        logger.error(f"未知错误在交易操作中: {e}", exc_info=True)
        return False
```

### 重试机制

```python
import time
import random
from functools import wraps

def retry(max_attempts: int = 3, base_delay: float = 1.0):
    """
    重试装饰器

    Args:
        max_attempts: 最大重试次数
        base_delay: 基础延迟时间
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except (NetworkError, ExchangeError) as e:
                    if attempt == max_attempts - 1:
                        raise

                    # 指数退避 + 随机抖动
                    delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
                    print(f"尝试 {attempt + 1} 失败，{delay:.1f}秒后重试...")
                    time.sleep(delay)

        return wrapper
    return decorator

# 使用示例
@retry(max_attempts=3)
def reliable_api_call(url: str) -> dict:
    """可靠的 API 调用"""
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

## 最佳实践

### 1. 资源管理
```python
# 使用上下文管理器
with ExchangeManager() as em:
    result = em.market_buy('SOL-USDT', 100)
    # 自动清理资源
```

### 2. 并发控制
```python
import asyncio
import aiohttp

async def async_multiple_trades(trades: list):
    """异步执行多个交易"""
    async with aiohttp.ClientSession() as session:
        tasks = [execute_trade_async(session, trade) for trade in trades]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        return results
```

### 3. 缓存策略
```python
from functools import lru_cache
import time

@lru_cache(maxsize=128)
def cached_token_price(token: str, max_age: int = 60):
    """带时间限制的缓存价格"""
    timestamp, price = _cached_prices.get(token, (0, None))
    if time.time() - timestamp > max_age:
        # 重新获取价格
        price = token_price(token)
        _cached_prices[token] = (time.time(), price)
    return price
```

### 4. 监控和日志
```python
import logging
from datetime import datetime

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('trading.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger('trading')

def log_trade(action: str, symbol: str, amount: float, result: dict):
    """记录交易日志"""
    logger.info(f"交易执行: {action} {symbol} ${amount}")
    if result['success']:
        logger.info(f"✅ 成功: {result}")
    else:
        logger.error(f"❌ 失败: {result['message']}")
```

这个 API 参考文档提供了完整的接口说明和使用示例，帮助开发者充分利用 Moon Dev AI Agents 的强大功能。