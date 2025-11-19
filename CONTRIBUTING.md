# 贡献指南

感谢您对 Moon Dev AI Agents 项目的关注！我们欢迎所有形式的贡献，包括代码、文档、测试、反馈和建议。

## 目录

1. [贡献方式](#贡献方式)
2. [开发环境设置](#开发环境设置)
3. [代码规范](#代码规范)
4. [提交流程](#提交流程)
5. [代码审查](#代码审查)
6. [测试指南](#测试指南)
7. [文档贡献](#文档贡献)
8. [社区准则](#社区准则)
9. [获得帮助](#获得帮助)

## 贡献方式

### 🐛 报告 Bug

发现问题时，请通过以下步骤报告：

1. **检查现有 Issues**：确保问题未被报告
2. **创建新 Issue**：使用 Bug Report 模板
3. **提供详细信息**：
   - 重现步骤
   - 期望行为 vs 实际行为
   - 环境信息（操作系统、Python版本等）
   - 错误日志和截图

### 💡 功能建议

提出新功能建议时：

1. **描述用例**：说明为什么需要这个功能
2. **设计方案**：提出可能的实现方案
3. **考虑影响**：分析对现有功能的影响
4. **标记标签**：使用 `enhancement` 标签

### 🔧 代码贡献

#### 小修改（ typo 修复、简单 Bug 修复）

1. Fork 项目
2. 创建特性分支
3. 提交修改
4. 创建 Pull Request

#### 大修改（新功能、重构）

1. **创建 Issue 讨论**：在开始编码前讨论设计
2. **创建计划**：分解任务，制定实施计划
3. **分步实施**：分阶段提交，保持每个 PR 小而专注
4. **完整测试**：确保所有测试通过

### 📚 文档贡献

文档改进同样重要！您可以：

- 修正错误和不清楚的地方
- 添加使用示例
- 翻译文档
- 改进代码注释

## 开发环境设置

### 1. Fork 和克隆

```bash
# Fork 项目，然后克隆你的 Fork
git clone https://github.com/YOUR_USERNAME/moon-dev-ai-agents.git
cd moon-dev-ai-agents

# 添加上游仓库
git remote add upstream https://github.com/afanty2021/moon-dev-ai-agents.git
```

### 2. 设置开发环境

```bash
# 激活 conda 环境
conda activate tflow

# 安装开发依赖
pip install -r requirements.txt
pip install -r requirements-dev.txt  # 开发专用依赖

# 安装 pre-commit 钩子
pre-commit install
```

### 3. 验证环境

```bash
# 运行健康检查
python scripts/health_check.py

# 运行测试套件
python -m pytest tests/

# 检查代码格式
black --check .
isort --check-only .
```

## 代码规范

### Python 代码风格

我们使用以下工具确保代码质量：

- **Black**：代码格式化
- **isort**：导入排序
- **flake8**：代码检查
- **mypy**：类型检查
- **pre-commit**：Git 提交钩子

#### 格式化规则

```bash
# 自动格式化代码
black src/ tests/

# 排序导入
isort src/ tests/

# 检查代码质量
flake8 src/ tests/
```

#### 代码格式示例

```python
# ✅ 好的代码风格
from typing import Dict, List, Optional
import pandas as pd
from termcolor import cprint

from src.models.model_factory import ModelFactory
from src.config import AI_TEMPERATURE, MAX_TOKENS


class TradingAgent:
    """交易代理类，负责执行交易决策。

    Attributes:
        model: AI 模型实例
        config: 交易配置字典
    """

    def __init__(self, config: Optional[Dict] = None) -> None:
        """初始化交易代理。

        Args:
            config: 可选的配置字典
        """
        self.model = ModelFactory().get_model('claude')
        self.config = config or {}

    def analyze_market(
        self,
        token: str,
        timeframe: str = '1H'
    ) -> Dict[str, any]:
        """分析市场数据。

        Args:
            token: 代币符号
            timeframe: 时间框架

        Returns:
            包含分析结果的字典

        Raises:
            ValueError: 当代币符号无效时
        """
        if not token:
            raise ValueError("代币符号不能为空")

        # 实现逻辑...
        return {'action': 'HOLD', 'confidence': 0.5}

# ❌ 避免的代码风格
import sys, os, json
def analyze(token,timeframe):
    if token==None:
        return None
    # 缺少类型提示和文档字符串
    pass
```

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名 | PascalCase | `TradingAgent`, `ModelFactory` |
| 函数名 | snake_case | `analyze_market`, `get_token_price` |
| 变量名 | snake_case | `token_price`, `market_data` |
| 常量 | UPPER_SNAKE_CASE | `MAX_POSITION_SIZE`, `API_TIMEOUT` |
| 私有方法 | 下划线前缀 | `_validate_config`, `_parse_response` |
| 模块名 | snake_case | `trading_agent.py`, `model_factory.py` |

### 类型提示

所有公共接口都应该有类型提示：

```python
from typing import Dict, List, Optional, Union, Callable, Any

def process_data(
    data: pd.DataFrame,
    callback: Optional[Callable[[str], None]] = None,
    options: Optional[Dict[str, Any]] = None
) -> List[Dict[str, Union[str, float]]]:
    """处理数据并返回结果列表。"""
    pass
```

### 文档字符串

使用 Google 风格的文档字符串：

```python
def calculate_risk_metrics(
    positions: List[Dict[str, float]],
    account_balance: float,
    max_risk_percentage: float = 2.0
) -> Dict[str, float]:
    """计算投资组合风险指标。

    Args:
        positions: 持仓列表，每个字典包含 'amount' 和 'entry_price'
        account_balance: 账户总余额
        max_risk_percentage: 最大风险百分比

    Returns:
        包含风险指标的字典：
        - 'total_exposure': 总风险敞口
        - 'risk_percentage': 当前风险百分比
        - 'max_position_size': 建议最大仓位大小

    Raises:
        ValueError: 当输入数据无效时
        InsufficientDataError: 当数据不足时

    Example:
        >>> positions = [{'amount': 100, 'entry_price': 50}]
        >>> result = calculate_risk_metrics(positions, 1000)
        >>> print(result['risk_percentage'])
    """
    pass
```

## 提交流程

### 1. 创建分支

```bash
# 同步最新代码
git checkout main
git pull upstream main

# 创建功能分支
git checkout -b feature/your-feature-name
# 或
git checkout -b fix/bug-description
```

### 2. 进行开发

```bash
# 添加修改
git add .

# 提交（使用有意义的消息）
git commit -m "feat: 添加新的风险分析功能

- 实现 VaR 计算算法
- 添加风险指标可视化
- 更新相关测试"

# 推送到你的 Fork
git push origin feature/your-feature-name
```

### 3. 提交消息格式

使用 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**类型说明**：
- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档更新
- `style`: 代码格式（不影响功能）
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具相关

**示例**：
```
feat(trading): 添加智能止损功能

- 实现动态止损价格计算
- 支持多种止损策略
- 添加回测验证

Closes #123
```

### 4. 创建 Pull Request

1. 在 GitHub 上创建 Pull Request
2. 使用 PR 模板填写必要信息
3. 等待代码审查
4. 根据反馈修改代码

### 5. PR 模板

```markdown
## 描述
简要描述这个 PR 的目的和实现的功能。

## 变更类型
- [ ] Bug 修复
- [ ] 新功能
- [ ] 重构
- [ ] 文档更新
- [ ] 测试改进

## 测试
- [ ] 添加了新的测试
- [ ] 所有测试通过
- [ ] 手动测试完成

## 检查清单
- [ ] 代码符合项目规范
- [ ] 自我审查了代码
- [ ] 添加了必要的注释
- [ ] 更新了相关文档
- [ ] 没有引入新的警告

## 相关 Issue
Closes #(issue number)
```

## 代码审查

### 审查者指南

#### 审查重点

1. **功能正确性**：代码是否实现了预期功能
2. **代码质量**：是否遵循项目规范
3. **性能影响**：是否会影响系统性能
4. **安全性**：是否存在安全隐患
5. **测试覆盖**：是否有足够的测试

#### 审查流程

1. **自动检查**：CI/CD 自动运行测试
2. **人工审查**：至少需要一人审查
3. **反馈处理**：作者根据反馈修改
4. **最终批准**：审查者批准合并

#### 审查评论规范

```markdown
# 建议
建议使用 `with` 语句来管理资源，确保正确清理。

# 问题
这里可能存在内存泄漏，需要在 finally 块中释放资源。

# 赞扬
很好的错误处理！这样的实现很健壮。

# 问题（必须修复）
必须添加类型提示，这是项目的规范要求。
```

### 被审查者指南

#### 响应审查

1. **及时响应**：在 24 小时内回复
2. **解释设计**：解释设计决策
3. **接受建议**：开放接受改进建议
4. **更新代码**：及时修改问题

#### 处理不同意见

```markdown
# 理性讨论
感谢建议！我考虑过这个方案，但选择当前实现是因为...
不过我愿意听听更多想法。

# 询问澄清
能详细说明为什么建议这样做吗？我可能忽略了某些方面。

# 提供替代方案
我理解你的担忧。如果采用这种折中方案怎么样？
```

## 测试指南

### 测试结构

```
tests/
├── unit/           # 单元测试
├── integration/    # 集成测试
├── e2e/           # 端到端测试
├── fixtures/      # 测试数据
└── conftest.py    # pytest 配置
```

### 编写测试

#### 单元测试

```python
# tests/unit/test_trading_agent.py
import pytest
from unittest.mock import Mock, patch
from src.agents.trading_agent import TradingAgent

class TestTradingAgent:
    """交易代理单元测试"""

    def setup_method(self):
        """每个测试方法前运行"""
        self.agent = TradingAgent()

    @patch('src.agents.trading_agent.ModelFactory')
    def test_analyze_market_success(self, mock_factory):
        """测试市场分析成功场景"""
        # 安排
        mock_model = Mock()
        mock_model.generate_response.return_value = "BUY"
        mock_factory.return_value.get_model.return_value = mock_model

        # 执行
        result = self.agent.analyze_market("BTC")

        # 断言
        assert result['action'] == 'BUY'
        assert isinstance(result['confidence'], (int, float))

    def test_analyze_market_invalid_token(self):
        """测试无效代币符号"""
        with pytest.raises(ValueError, match="代币符号不能为空"):
            self.agent.analyze_market("")
```

#### 集成测试

```python
# tests/integration/test_api_integration.py
import pytest
from src.agents.api import MoonDevAPI

class TestAPIIntegration:
    """API 集成测试"""

    @pytest.mark.slow
    def test_get_liquidation_data(self):
        """测试获取清算数据"""
        api = MoonDevAPI()
        data = api.get_liquidation_data()

        assert 'total_liquidated' in data
        assert isinstance(data['total_liquidated'], (int, float))

    @pytest.mark.network
    def test_api_timeout_handling(self):
        """测试 API 超时处理"""
        api = MoonDevAPI(timeout=1)  # 极短超时
        # 测试超时处理逻辑
```

### 测试命令

```bash
# 运行所有测试
pytest

# 运行特定测试
pytest tests/unit/test_trading_agent.py

# 运行带覆盖率的测试
pytest --cov=src --cov-report=html

# 运行特定标记的测试
pytest -m "not slow"  # 跳过慢测试
pytest -m "integration"  # 只运行集成测试

# 生成覆盖率报告
coverage run -m pytest
coverage html
```

### 测试最佳实践

1. **测试命名**：使用描述性的测试名称
2. **AAA 模式**：Arrange（安排）、Act（执行）、Assert（断言）
3. **独立性**：测试之间不应相互依赖
4. **模拟外部依赖**：使用 mock 隔离外部服务
5. **边界条件**：测试边界和异常情况

## 文档贡献

### 代码文档

- **所有公共函数**：必须有文档字符串
- **复杂逻辑**：添加内联注释
- **类型提示**：为所有接口提供类型信息
- **示例代码**：在文档中提供使用示例

### README 更新

- **新功能**：更新 README 中的功能列表
- **API 变更**：更新 API 文档
- **安装步骤**：保持安装说明最新
- **配置选项**：文档化新的配置参数

### 文档写作规范

```markdown
# 使用清晰的标题结构
## 主要概念
### 子概念

# 代码块使用语法高亮
```python
def example_function():
    pass
```

# 使用表格整理信息
| 参数 | 类型 | 描述 |
|------|------|------|
| timeout | int | 请求超时时间（秒） |

# 使用提示和警告
> 💡 **提示**: 这是一个有用的提示

> ⚠️ **警告**: 注意这个重要信息
```

## 社区准则

### 行为准则

我们致力于为每个人提供友好、安全和欢迎的环境：

1. **尊重他人**：礼貌、包容、尊重不同观点
2. **建设性反馈**：提供建设性的、具体的反馈
3. **专业沟通**：保持专业和友好的沟通方式
4. **帮助新手**：耐心帮助新贡献者
5. **遵守规则**：遵循项目规范和流程

### 沟通渠道

- **GitHub Issues**: 报告 Bug 和功能建议
- **GitHub Discussions**: 一般讨论和问答
- **Discord**: 实时聊天和社区交流
- **Pull Request**: 代码审查和技术讨论

### 冲突解决

如果遇到冲突或问题：

1. **直接沟通**：首先尝试直接沟通解决
2. **寻求调解**：请求项目维护者帮助调解
3. **官方渠道**：使用官方行为准则报告流程

## 获得帮助

### 开发问题

- **查看文档**：首先查看现有文档
- **搜索 Issues**：查看是否有类似问题
- **创建 Issue**：如果问题未解决，创建新 Issue
- **加入 Discord**：在社区中寻求帮助

### 技术支持

```bash
# 获取项目状态
python scripts/status_check.py

# 检查环境配置
python scripts/env_check.py

# 运行诊断工具
python scripts/diagnose.py
```

### 资源链接

- [项目文档](https://docs.moondev.ai)
- [API 参考](https://api.moondev.ai)
- [视频教程](https://youtube.com/@moondev)
- [社区 Discord](https://discord.gg/moondev)

## 致谢

感谢所有为项目做出贡献的开发者！您的贡献让这个项目变得更好。

### 贡献者名单

- [主要贡献者列表]
- [代码贡献者]
- [文档贡献者]
- [社区支持者]

---

**再次感谢您的贡献！让我们一起构建更好的 AI 交易系统！** 🚀