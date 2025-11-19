# 代码规范和质量标准

## 概述

本文档详细说明了 Moon Dev AI Agents 项目的代码规范、质量标准和最佳实践。所有贡献者都必须遵循这些标准以确保代码质量和一致性。

## 目录

1. [代码格式化](#代码格式化)
2. [命名规范](#命名规范)
3. [类型提示](#类型提示)
4. [文档字符串](#文档字符串)
5. [错误处理](#错误处理)
6. [测试规范](#测试规范)
7. [性能考虑](#性能考虑)
8. [安全规范](#安全规范)
9. [Git 使用规范](#git-使用规范)
10. [代码审查清单](#代码审查清单)

## 代码格式化

### 自动格式化工具

项目使用以下工具自动格式化代码：

```bash
# 安装开发依赖
pip install black isort flake8 mypy pre-commit

# 设置 pre-commit 钩子
pre-commit install
```

### Black 配置

`pyproject.toml`:
```toml
[tool.black]
line-length = 88
target-version = ['py39']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | _build
  | buck-out
  | build
  | dist
)/
'''

[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["src"]
known_third_party = ["pandas", "numpy", "requests"]
```

### 格式化规则

#### 行长度和缩进

```python
# ✅ 正确：行长度不超过 88 字符
def analyze_market_data(
    token_address: str,
    timeframe: str = '1H',
    days_back: int = 7,
    include_indicators: bool = True
) -> Dict[str, Any]:
    """分析市场数据并生成交易建议。"""
    pass

# ❌ 错误：行过长
def analyze_market_data(token_address: str, timeframe: str = '1H', days_back: int = 7, include_indicators: bool = True) -> Dict[str, Any]:
    pass
```

#### 导入语句

```python
# ✅ 正确：按类型分组，按字母排序
import os
import sys
from typing import Dict, List, Optional

import pandas as pd
import requests
from termcolor import cprint

from src.models.model_factory import ModelFactory
from src.nice_funcs import token_overview

# ❌ 错误：未分组，未排序
from src.nice_funcs import token_overview
import pandas as pd
import requests
import os
from typing import Dict, List, Optional
from src.models.model_factory import ModelFactory
from termcolor import cprint
import sys
```

#### 空行使用

```python
# ✅ 正确：适当的空行分隔
import os

import pandas as pd

from src.config import MAX_POSITION_SIZE


class TradingAgent:
    """交易代理类。"""

    def __init__(self) -> None:
        """初始化交易代理。"""
        self.model = ModelFactory().get_model('claude')
        self.positions = {}

    def analyze(self, token: str) -> Dict[str, Any]:
        """分析代币。"""
        # 获取市场数据
        market_data = self._get_market_data(token)

        # 生成分析结果
        return self._generate_analysis(market_data)

    def _get_market_data(self, token: str) -> Dict[str, Any]:
        """获取市场数据。"""
        pass

    def _generate_analysis(self, data: Dict[str, Any]) -> Dict[str, Any]:
        """生成分析结果。"""
        pass

# ❌ 错误：缺少必要的空行
import os
import pandas as pd
from src.config import MAX_POSITION_SIZE
class TradingAgent:
    def __init__(self) -> None:
        self.model = ModelFactory().get_model('claude')
    def analyze(self, token: str) -> Dict[str, Any]:
        market_data = self._get_market_data(token)
        return self._generate_analysis(market_data)
```

## 命名规范

### 基本规则

| 类型 | 命名风格 | 示例 |
|------|---------|------|
| 类名 | PascalCase | `TradingAgent`, `ModelFactory`, `RiskManager` |
| 函数名 | snake_case | `analyze_market`, `get_token_price`, `calculate_pnl` |
| 变量名 | snake_case | `token_address`, `market_data`, `position_size` |
| 常量 | UPPER_SNAKE_CASE | `MAX_POSITION_SIZE`, `API_TIMEOUT`, `DEFAULT_MODEL` |
| 私有方法 | _snake_case | `_validate_config`, `_parse_response`, `_cleanup` |
| 模块名 | snake_case | `trading_agent.py`, `model_factory.py`, `risk_manager.py` |
| 包名 | lowercase | `src`, `agents`, `models`, `strategies` |

### 具体示例

```python
# ✅ 正确的命名
class TradingAgent:
    MAX_POSITION_SIZE = 1000  # 常量
    API_TIMEOUT = 30

    def __init__(self) -> None:
        self.current_positions = {}  # 实例变量
        self._last_update = None      # 私有变量

    def analyze_market_data(self, token_address: str) -> Dict[str, Any]:
        """分析市场数据。"""
        market_data = self._fetch_market_data(token_address)
        return self._process_market_data(market_data)

    def _fetch_market_data(self, token: str) -> Dict[str, Any]:
        """获取市场数据（私有方法）。"""
        pass

# ❌ 错误的命名
class tradingagent:
    maxpositionsize = 1000
    api_timeout = 30

    def __init__(self):
        self.currentPositions = {}
        self.lastUpdate = None

    def AnalyzeMarketData(self, tokenAddress):
        pass

    def fetchMarketData(self, token):
        pass
```

### 特殊命名约定

#### 布尔变量

```python
# ✅ 使用 is/has/can 前缀
is_valid = True
has_position = False
can_trade = True
should_execute = True

# ❌ 避免否定形式
not_invalid = True  # 难以理解
is_not_valid = True  # 使用 positive naming
```

#### 集合变量

```python
# ✅ 使用复数形式
token_prices = {}
active_positions = []
api_endpoints = {'get_price', 'place_order'}

# ❌ 避免单数形式
token_price = {}  # 应该是 token_prices
active_position = []  # 应该是 active_positions
```

## 类型提示

### 基本类型提示

```python
from typing import Dict, List, Optional, Union, Callable, Any, Tuple

# ✅ 完整的类型提示
def process_trading_data(
    data: Dict[str, Union[str, float]],
    callback: Optional[Callable[[str], None]] = None,
    options: Optional[Dict[str, Any]] = None
) -> List[Tuple[str, float]]:
    """处理交易数据。"""
    pass

# 带默认值的类型提示
def calculate_position_size(
    account_balance: float,
    risk_percentage: float = 2.0,
    leverage: int = 1
) -> float:
    """计算仓位大小。"""
    pass
```

### 复杂类型提示

```python
from typing import TypedDict, Protocol, Literal, TypeVar, Generic

# TypedDict 定义结构化字典
class MarketData(TypedDict):
    symbol: str
    price: float
    volume: float
    timestamp: str

class TradingSignal(TypedDict):
    action: Literal['BUY', 'SELL', 'HOLD']
    confidence: float  # 0-1
    reasoning: str

# Protocol 定义接口
class ModelInterface(Protocol):
    def generate_response(
        self,
        system_prompt: str,
        user_content: str,
        temperature: float = 0.7,
        max_tokens: int = 1024
    ) -> str:
        ...

# Generic 类型
T = TypeVar('T')

class DataProcessor(Generic[T]):
    def __init__(self, data: List[T]) -> None:
        self.data = data

    def process(self, processor: Callable[[T], Any]) -> List[Any]:
        return [processor(item) for item in self.data]
```

### 类型别名

```python
# 创建常用类型别名
TokenAddress = str
Price = float
PositionSize = float
TradingSignal = Dict[str, Union[str, float, bool]]

# 使用类型别名
def analyze_token(token: TokenAddress, current_price: Price) -> TradingSignal:
    """分析代币生成交易信号。"""
    pass
```

## 文档字符串

### Google 风格文档字符串

```python
def calculate_risk_metrics(
    positions: List[Dict[str, float]],
    account_balance: float,
    max_risk_percentage: float = 2.0,
    time_horizon: str = '1d'
) -> Dict[str, float]:
    """计算投资组合风险指标。

    这个函数计算给定持仓组合的各种风险指标，包括 VaR、
    最大回撤、风险敞口等。

    Args:
        positions: 持仓列表，每个字典应包含：
            - 'symbol': 代币符号 (str)
            - 'amount': 持仓数量 (float)
            - 'entry_price': 入场价格 (float)
        account_balance: 账户总余额，单位 USD
        max_risk_percentage: 最大风险百分比，默认 2.0%
        time_horizon: 风险计算的时间范围，可选值：
            '1h', '4h', '1d', '1w'，默认 '1d'

    Returns:
        包含风险指标的字典：
        - 'total_exposure': 总风险敞口
        - 'var_95': 95% VaR 值
        - 'max_drawdown': 最大回撤百分比
        - 'risk_percentage': 当前风险百分比
        - 'sharpe_ratio': 夏普比率
        - 'position_count': 持仓数量

    Raises:
        ValueError: 当输入数据无效时
        InsufficientDataError: 当数据不足以计算时
        NetworkError: 当无法获取市场数据时

    Example:
        >>> positions = [
        ...     {'symbol': 'BTC', 'amount': 0.1, 'entry_price': 45000},
        ...     {'symbol': 'ETH', 'amount': 2.0, 'entry_price': 3000}
        ... ]
        >>> result = calculate_risk_metrics(positions, 10000)
        >>> print(f"风险百分比: {result['risk_percentage']:.2f}%")
        风险百分比: 1.85%

    Note:
        这个函数假设所有价格都是 USD 计价。
        计算结果基于历史数据，不代表未来表现。
    """
    pass
```

### 类文档字符串

```python
class TradingAgent:
    """AI 驱动的交易代理。

    这个类封装了交易决策逻辑，使用机器学习模型分析市场数据
    并生成交易建议。支持多种交易策略和风险管理功能。

    Attributes:
        model: AI 模型实例，用于分析市场数据
        positions: 当前持仓字典，键为代币符号，值为持仓信息
        config: 交易配置参数
        risk_manager: 风险管理器实例

    Example:
        >>> agent = TradingAgent(config={'max_risk': 2.0})
        >>> signal = agent.analyze_market('BTC')
        >>> if signal['action'] == 'BUY':
        ...     agent.execute_buy_order('BTC', signal['amount'])

    Note:
        所有交易操作都通过风险管理器验证，确保不超过预设的风险限制。
    """

    def __init__(self, config: Optional[Dict[str, Any]] = None) -> None:
        """初始化交易代理。

        Args:
            config: 可选的配置字典，包含交易参数和风险设置
        """
        pass
```

## 错误处理

### 自定义异常

```python
# src/exceptions.py

class MoonDevError(Exception):
    """基础异常类。"""
    pass

class TradingError(MoonDevError):
    """交易相关错误。"""
    pass

class InsufficientBalanceError(TradingError):
    """余额不足错误。"""
    pass

class InvalidOrderError(TradingError):
    """无效订单错误。"""
    pass

class ModelError(MoonDevError):
    """AI 模型相关错误。"""
    pass

class ConfigurationError(MoonDevError):
    """配置错误。"""
    pass
```

### 错误处理最佳实践

```python
# ✅ 良好的错误处理
def place_order(
    symbol: str,
    amount: float,
    order_type: str = 'market'
) -> Dict[str, Any]:
    """下单交易。"""
    try:
        # 验证输入参数
        if amount <= 0:
            raise InvalidOrderError(f"订单数量必须大于0，收到: {amount}")

        if order_type not in ['market', 'limit']:
            raise InvalidOrderError(f"不支持的订单类型: {order_type}")

        # 检查余额
        balance = get_account_balance()
        if balance < amount:
            raise InsufficientBalanceError(
                f"余额不足。需要: ${amount}, 可用: ${balance}"
            )

        # 执行订单
        order_id = _execute_order(symbol, amount, order_type)

        return {
            'success': True,
            'order_id': order_id,
            'symbol': symbol,
            'amount': amount,
            'type': order_type
        }

    except InvalidOrderError as e:
        logger.error(f"无效订单: {e}")
        return {'success': False, 'error': str(e)}

    except InsufficientBalanceError as e:
        logger.error(f"余额不足: {e}")
        return {'success': False, 'error': str(e)}

    except NetworkError as e:
        logger.error(f"网络错误: {e}")
        # 可以实现重试逻辑
        return {'success': False, 'error': '网络连接失败'}

    except Exception as e:
        logger.error(f"未知错误: {e}", exc_info=True)
        return {'success': False, 'error': '系统错误'}

# ❌ 不良的错误处理
def place_order_bad(symbol, amount, order_type):
    """下单交易（不良示例）。"""
    try:
        if amount <= 0:
            return False

        balance = get_account_balance()
        if balance < amount:
            return False

        order_id = _execute_order(symbol, amount, order_type)
        return order_id

    except:
        # 捕获所有异常但不处理
        return False
```

### 日志记录规范

```python
import logging
from typing import Any

# 配置日志
logger = logging.getLogger(__name__)

class TradingAgent:
    """交易代理类。"""

    def analyze_market(self, token: str) -> Dict[str, Any]:
        """分析市场数据。"""
        logger.info(f"开始分析代币: {token}")

        try:
            market_data = self._fetch_market_data(token)
            logger.debug(f"获取到市场数据: {len(market_data)} 条记录")

            analysis = self._generate_analysis(market_data)
            logger.info(f"分析完成，建议: {analysis['action']}")

            return analysis

        except ValueError as e:
            logger.error(f"输入参数错误: {e}")
            raise

        except NetworkError as e:
            logger.error(f"网络错误，重试中: {e}")
            # 实现重试逻辑
            return self._retry_analysis(token)

        except Exception as e:
            logger.error(f"分析失败: {e}", exc_info=True)
            raise TradingError(f"市场分析失败: {e}") from e
```

## 测试规范

### 测试结构

```python
# tests/unit/test_trading_agent.py

import pytest
from unittest.mock import Mock, patch, MagicMock
from src.agents.trading_agent import TradingAgent
from src.exceptions import InsufficientBalanceError, InvalidOrderError


class TestTradingAgent:
    """交易代理测试类。"""

    @pytest.fixture
    def agent(self) -> TradingAgent:
        """创建测试用的交易代理实例。"""
        config = {
            'max_risk': 2.0,
            'default_amount': 100.0
        }
        return TradingAgent(config)

    @pytest.fixture
    def mock_market_data(self) -> Dict[str, Any]:
        """提供模拟市场数据。"""
        return {
            'symbol': 'BTC-USDT',
            'price': 45000.0,
            'volume': 1000000.0,
            'change_24h': 2.5
        }

    def test_init_success(self, agent: TradingAgent) -> None:
        """测试初始化成功。"""
        assert agent.config['max_risk'] == 2.0
        assert agent.config['default_amount'] == 100.0

    def test_analyze_market_success(
        self,
        agent: TradingAgent,
        mock_market_data: Dict[str, Any]
    ) -> None:
        """测试市场分析成功场景。"""
        with patch.object(agent, '_fetch_market_data', return_value=mock_market_data):
            result = agent.analyze_market('BTC')

            assert 'action' in result
            assert result['action'] in ['BUY', 'SELL', 'HOLD']
            assert 'confidence' in result
            assert 0 <= result['confidence'] <= 1

    def test_analyze_market_invalid_token(self, agent: TradingAgent) -> None:
        """测试无效代币符号。"""
        with pytest.raises(ValueError, match="代币符号不能为空"):
            agent.analyze_market('')

    @patch('src.agents.trading_agent.get_account_balance')
    def test_place_order_insufficient_balance(
        self,
        mock_balance: Mock,
        agent: TradingAgent
    ) -> None:
        """测试余额不足场景。"""
        mock_balance.return_value = 50.0

        with pytest.raises(InsufficientBalanceError):
            agent.place_order('BTC', 100.0)

    @pytest.mark.parametrize("amount,expected", [
        (0, InvalidOrderError),
        (-10, InvalidOrderError),
        (100, None),  # 正常情况
        (1000, None)  # 大额订单
    ])
    def test_place_order_amount_validation(
        self,
        agent: TradingAgent,
        amount: float,
        expected: type
    ) -> None:
        """测试订单金额验证。"""
        with patch('src.agents.trading_agent.get_account_balance', return_value=10000):
            if expected:
                with pytest.raises(expected):
                    agent.place_order('BTC', amount)
            else:
                # 应该成功执行
                result = agent.place_order('BTC', amount)
                assert result['success']
```

### 测试数据管理

```python
# tests/fixtures/market_data.py

import pytest
from typing import Dict, Any, List

@pytest.fixture
def sample_ohlcv_data() -> List[Dict[str, Any]]:
    """提供样本 OHLCV 数据。"""
    return [
        {'timestamp': '2024-01-01T00:00:00Z', 'open': 45000, 'high': 45500, 'low': 44800, 'close': 45200, 'volume': 1000},
        {'timestamp': '2024-01-01T01:00:00Z', 'open': 45200, 'high': 45800, 'low': 45000, 'close': 45600, 'volume': 1200},
        {'timestamp': '2024-01-01T02:00:00Z', 'open': 45600, 'high': 46000, 'low': 45400, 'close': 45800, 'volume': 800},
    ]

@pytest.fixture
def mock_token_data() -> Dict[str, Any]:
    """提供模拟代币数据。"""
    return {
        'symbol': 'BTC',
        'name': 'Bitcoin',
        'price': 45800.0,
        'market_cap': 895000000000,
        'volume_24h': 25000000000,
        'change_24h': 2.5,
        'circulating_supply': 19500000
    }
```

## 性能考虑

### 内存优化

```python
# ✅ 内存优好的代码
def process_large_dataset(data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """处理大型数据集。"""
    result = []

    # 使用生成器避免一次性加载所有数据
    for chunk in _chunk_data(data, chunk_size=1000):
        processed_chunk = _process_chunk(chunk)
        result.extend(processed_chunk)

        # 及时释放内存
        del chunk
        del processed_chunk

    return result

def _chunk_data(data: List[Dict[str, Any]], chunk_size: int) -> Iterator[List[Dict[str, Any]]]:
    """将数据分块。"""
    for i in range(0, len(data), chunk_size):
        yield data[i:i + chunk_size]

# ❌ 内存使用不当
def process_large_dataset_bad(data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """处理大型数据集（不良示例）。"""
    # 一次性处理所有数据，可能导致内存不足
    all_results = []
    for item in data:
        processed = _process_item(item)
        all_results.append(processed)

    return all_results
```

### 缓存策略

```python
from functools import lru_cache
import time
from typing import Dict, Any, Optional

class CachedDataFetcher:
    """带缓存的数据获取器。"""

    def __init__(self, cache_ttl: int = 300) -> None:
        self.cache_ttl = cache_ttl
        self._cache: Dict[str, Tuple[Any, float]] = {}

    def get_token_price(self, token: str) -> float:
        """获取代币价格（带缓存）。"""
        cache_key = f"price:{token}"

        # 检查缓存
        if cache_key in self._cache:
            cached_value, timestamp = self._cache[cache_key]
            if time.time() - timestamp < self.cache_ttl:
                return cached_value

        # 获取新数据
        price = self._fetch_price_from_api(token)

        # 更新缓存
        self._cache[cache_key] = (price, time.time())

        return price

    @lru_cache(maxsize=1000)
    def get_token_info(self, token: str) -> Dict[str, Any]:
        """获取代币信息（使用 functools.lru_cache）。"""
        return self._fetch_info_from_api(token)
```

### 异步处理

```python
import asyncio
import aiohttp
from typing import List, Dict, Any

class AsyncDataProcessor:
    """异步数据处理器。"""

    async def fetch_multiple_prices(self, tokens: List[str]) -> Dict[str, float]:
        """异步获取多个代币价格。"""
        async with aiohttp.ClientSession() as session:
            tasks = [
                self._fetch_single_price(session, token)
                for token in tokens
            ]
            results = await asyncio.gather(*tasks, return_exceptions=True)

            prices = {}
            for token, result in zip(tokens, results):
                if isinstance(result, Exception):
                    logger.error(f"获取 {token} 价格失败: {result}")
                    continue
                prices[token] = result

            return prices

    async def _fetch_single_price(
        self,
        session: aiohttp.ClientSession,
        token: str
    ) -> float:
        """获取单个代币价格。"""
        url = f"https://api.example.com/price/{token}"
        async with session.get(url) as response:
            data = await response.json()
            return float(data['price'])
```

## 安全规范

### API 密钥管理

```python
import os
from typing import Optional

class SecureConfig:
    """安全的配置管理器。"""

    def __init__(self) -> None:
        self._api_keys: Dict[str, str] = {}
        self._load_keys()

    def _load_keys(self) -> None:
        """从环境变量加载 API 密钥。"""
        key_mappings = {
            'ANTHROPIC_KEY': 'anthropic',
            'OPENAI_KEY': 'openai',
            'GROQ_API_KEY': 'groq'
        }

        for env_var, key_name in key_mappings.items():
            key_value = os.getenv(env_var)
            if key_value:
                self._api_keys[key_name] = key_value
            else:
                logger.warning(f"缺少 API 密钥: {env_var}")

    def get_key(self, provider: str) -> Optional[str]:
        """获取 API 密钥。"""
        return self._api_keys.get(provider)

    def mask_key(self, key: str) -> str:
        """遮蔽 API 密钥用于日志记录。"""
        if len(key) <= 8:
            return '*' * len(key)
        return key[:4] + '*' * (len(key) - 8) + key[-4:]

# ✅ 正确使用
config = SecureConfig()
anthropic_key = config.get_key('anthropic')
logger.info(f"使用 Anthropic API 密钥: {config.mask_key(anthropic_key)}")

# ❌ 错误：直接记录 API 密钥
logger.info(f"使用 Anthropic API 密钥: {anthropic_key}")  # 安全风险！
```

### 输入验证

```python
import re
from typing import Any, Dict, List

class InputValidator:
    """输入验证器。"""

    @staticmethod
    def validate_token_address(address: str) -> bool:
        """验证代币地址格式。"""
        if not address or not isinstance(address, str):
            return False

        # Solana 地址格式验证
        solana_pattern = r'^[1-9A-HJ-NP-Za-km-z]{32,44}$'
        return bool(re.match(solana_pattern, address))

    @staticmethod
    def validate_trading_amount(amount: float) -> bool:
        """验证交易金额。"""
        return isinstance(amount, (int, float)) and 0 < amount <= 1000000

    @staticmethod
    def sanitize_user_input(input_str: str) -> str:
        """清理用户输入。"""
        if not input_str:
            return ""

        # 移除潜在危险字符
        dangerous_chars = ['<', '>', '"', "'", '&', '\x00']
        for char in dangerous_chars:
            input_str = input_str.replace(char, '')

        return input_str.strip()

# ✅ 正确的输入验证
def place_order(token_address: str, amount: float) -> Dict[str, Any]:
    """下单交易（带输入验证）。"""
    if not InputValidator.validate_token_address(token_address):
        raise ValueError(f"无效的代币地址: {token_address}")

    if not InputValidator.validate_trading_amount(amount):
        raise ValueError(f"无效的交易金额: {amount}")

    # 继续处理订单...

# ❌ 错误：缺少输入验证
def place_order_unsafe(token_address: str, amount: float) -> Dict[str, Any]:
    """下单交易（不安全）。"""
    # 直接使用用户输入，没有验证
    return _execute_order(token_address, amount)
```

### SQL 注入防护

```python
# 如果项目使用数据库，确保参数化查询

import sqlite3
from typing import List, Dict, Any

class DatabaseManager:
    """数据库管理器。"""

    def __init__(self, db_path: str) -> None:
        self.db_path = db_path

    def get_trading_history(
        self,
        user_id: int,
        start_date: str,
        end_date: str
    ) -> List[Dict[str, Any]]:
        """获取交易历史（安全版本）。"""
        query = """
        SELECT id, symbol, amount, price, timestamp
        FROM trades
        WHERE user_id = ? AND timestamp BETWEEN ? AND ?
        ORDER BY timestamp DESC
        """

        try:
            with sqlite3.connect(self.db_path) as conn:
                conn.row_factory = sqlite3.Row
                cursor = conn.execute(query, (user_id, start_date, end_date))
                return [dict(row) for row in cursor.fetchall()]

        except sqlite3.Error as e:
            logger.error(f"数据库错误: {e}")
            return []

    # ❌ 危险的 SQL 注入风险
    def get_trading_history_unsafe(self, user_id: int, start_date: str) -> List[Dict[str, Any]]:
        """获取交易历史（不安全版本 - 不要使用！）。"""
        # 危险：直接字符串拼接
        query = f"""
        SELECT * FROM trades
        WHERE user_id = {user_id} AND timestamp >= '{start_date}'
        """

        # 这种写法容易受到 SQL 注入攻击
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(query)
            return [dict(row) for row in cursor.fetchall()]
```

## Git 使用规范

### 分支命名

```bash
# 功能分支
feature/trading-agent-enhancement
feature/risk-management-system

# 修复分支
fix/order-execution-bug
fix/memory-leak-issue

# 文档分支
docs/api-reference-update
docs/installation-guide

# 热修复分支（从 main 分出）
hotfix/critical-security-fix
hotfix/production-bug-fix

# 发布分支
release/v1.2.0
```

### 提交消息格式

```bash
# 基本格式
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]

# 示例
feat(trading): 添加智能止损功能

实现了基于技术指标的动态止损系统：
- 支持 RSI 和移动平均线止损
- 可配置止损百分比
- 添加止损回测功能

Closes #123
Fixes #124

# 更多示例
fix(api): 修复订单超时处理

docs(readme): 更新安装说明
refactor(models): 重构模型工厂设计
test(trading): 添加交易代理单元测试
chore(deps): 更新依赖包版本
```

### 提交类型说明

- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档更新
- `style`: 代码格式化（不影响功能）
- `refactor`: 代码重构
- `test`: 测试相关
- `chore`: 构建过程或辅助工具的变动
- `perf`: 性能优化
- `ci`: CI/CD 相关

## 代码审查清单

### 功能性检查

- [ ] 代码实现了预期功能
- [ ] 边界条件处理正确
- [ ] 错误处理完整且合理
- [ ] 性能影响可接受
- [ ] 没有引入新的安全风险
- [ ] 向后兼容性考虑

### 代码质量检查

- [ ] 遵循项目代码规范
- [ ] 变量和函数命名清晰
- [ ] 代码结构合理，职责分离
- [ ] 注释准确且必要
- [ ] 没有重复代码
- [ ] 复杂度控制在合理范围

### 测试检查

- [ ] 添加了适当的单元测试
- [ ] 测试覆盖了主要功能路径
- [ ] 测试了边界情况和异常情况
- [ ] 集成测试通过
- [ ] 性能测试通过（如需要）

### 文档检查

- [ ] 更新了相关文档
- [ ] 添加了必要的注释
- [ ] API 文档准确
- [ ] 变更日志更新
- [ ] 使用示例清晰

### 安全检查

- [ ] 没有硬编码的密钥或密码
- [ ] 输入验证完整
- [ ] 权限检查适当
- [ ] 敏感信息处理安全
- [ ] 没有引入新的安全漏洞

---

遵循这些代码规范和质量标准，我们将能够构建高质量、可维护、安全的 AI 交易系统。所有贡献者都应该仔细阅读并严格遵守这些标准。