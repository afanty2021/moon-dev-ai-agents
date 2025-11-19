# Moon Dev AI Agents 文档

欢迎来到 Moon Dev AI Agents 项目的官方文档中心。这里包含了完整的项目文档、开发指南、API参考和最佳实践。

## 📚 文档导航

### 🚀 快速开始
- [项目概述](../README.md) - 项目介绍和核心特性
- [快速安装指南](quickstart.md) - 5分钟快速上手
- [环境设置指南](environment-setup.md) - 详细的开发环境配置
- [常见问题解答](troubleshooting.md) - 故障排除和问题解决

### 🏗️ 项目架构
- [系统架构](architecture.md) - 完整的系统架构和设计原理
- [API参考文档](api-reference.md) - 核心API接口详细说明
- [组件架构](components/) - 各个组件的详细文档
- [数据流程](data-flow.md) - 数据流转和处理机制

### 🤖 AI模型集成
- [LLM集成指南](llm-integration.md) - ModelFactory完整使用指南
- [AI模型配置](ai-models.md) - 支持的AI模型和配置方法
- [提示词工程](prompt-engineering.md) - 优化AI响应的最佳实践
- [多模型协作](multi-model.md) - SwarmAgent群体智能

### 📈 交易功能
- [交易代理API](api-reference.md#交易代理api) - TradingAgent详细文档
- [策略代理](strategies.md) - 策略开发和集成指南
- [风险管理](risk-management.md) - 风险控制和安全机制
- [数据提供商](data-providers.md) - 集成的数据源和API

### 🛠️ 开发指南
- [代码贡献指南](../CONTRIBUTING.md) - 如何参与项目开发
- [代码规范](coding-standards.md) - 编码标准和最佳实践
- [测试指南](testing.md) - 测试策略和质量保证
- [调试技巧](debugging.md) - 开发调试和问题诊断

### 🚀 部署运维
- [Docker部署](docker-deployment.md) - 容器化部署完整指南
- [生产环境部署](production-deployment.md) - 生产环境最佳实践
- [监控和日志](monitoring.md) - 系统监控和日志管理
- [性能优化](performance.md) - 性能调优和优化策略

### 🔒 安全合规
- [安全最佳实践](security.md) - 安全架构和威胁防护
- [合规性要求](compliance.md) - 金融监管和数据保护
- [密钥管理](key-management.md) - 安全的密钥存储和使用
- [审计日志](audit-logging.md) - 完整的审计日志系统

### 📖 参考资料
- [配置手册](configuration.md) - 完整的配置参数说明
- [术语表](glossary.md) - 项目相关术语和概念
- [更新日志](changelog.md) - 版本更新和变更记录
- [路线图](roadmap.md) - 项目发展规划和里程碑

## 🗂️ 文档分类

### 按用户类型分类

#### 👨‍💻 开发者
- 环境设置
- API参考
- 代码贡献指南
- 测试指南
- 调试技巧

#### 🔧 运维工程师
- Docker部署
- 生产环境部署
- 监控和日志
- 安全最佳实践
- 性能优化

#### 📊 业务用户
- 快速开始
- 系统架构
- 交易功能
- 风险管理
- 合规性要求

#### 🔒 安全专员
- 安全最佳实践
- 合规性要求
- 密钥管理
- 审计日志
- 威胁防护

### 按难度分类

#### 🟢 入门级
- [快速开始](../README.md)
- [环境设置](environment-setup.md)
- [常见问题](troubleshooting.md)

#### 🟡 中级
- [API参考](api-reference.md)
- [LLM集成](llm-integration.md)
- [代码规范](coding-standards.md)

#### 🔴 高级
- [生产环境部署](production-deployment.md)
- [安全最佳实践](security.md)
- [性能优化](performance.md)

## 🔍 文档搜索

### 关键主题索引
- **代理**: [TradingAgent](api-reference.md#核心代理api)、[RiskAgent](risk-management.md)、[SwarmAgent](multi-model.md)
- **AI模型**: [ModelFactory](llm-integration.md)、[Claude](ai-models.md#anthropic-claude)、[GPT](ai-models.md#openai-gpt)
- **交易**: [执行订单](api-reference.md#执行交易)、[风险控制](risk-management.md)、[策略](strategies.md)
- **部署**: [Docker](docker-deployment.md)、[Kubernetes](kubernetes.md)、[监控](monitoring.md)
- **安全**: [加密](security.md#加密和密钥管理)、[认证](security.md#身份认证和授权)、[合规](compliance.md)

### 功能快速导航
- **开始开发**: [环境设置](environment-setup.md) → [代码规范](coding-standards.md) → [API参考](api-reference.md)
- **部署生产**: [Docker](docker-deployment.md) → [生产环境](production-deployment.md) → [监控](monitoring.md)
- **安全配置**: [安全最佳实践](security.md) → [密钥管理](key-management.md) → [合规](compliance.md)
- **AI集成**: [LLM指南](llm-integration.md) → [多模型协作](multi-model.md) → [提示词工程](prompt-engineering.md)

## 📄 文档格式说明

### 代码示例
```python
# Python 代码示例
from src.models.model_factory import ModelFactory

factory = ModelFactory()
model = factory.get_model('claude')
response = model.generate_response("Hello, world!")
```

### 配置示例
```yaml
# YAML 配置示例
version: '3.8'
services:
  app:
    image: moon-dev-ai-agents:latest
    ports:
      - "8000:8000"
```

### 命令行示例
```bash
# Bash 命令示例
pip install -r requirements.txt
python src/main.py
```

### 警告和提示
> 💡 **提示**: 有用的建议和最佳实践

> ⚠️ **警告**: 需要注意的重要信息

> ❌ **错误**: 应该避免的做法

## 🔗 外部资源

### 官方链接
- [项目主页](https://github.com/afanty2021/moon-dev-ai-agents)
- [Discord社区](https://discord.gg/moondev)
- [YouTube频道](https://youtube.com/@moondev)
- [官方网站](https://moondev.ai)

### 相关文档
- [Python官方文档](https://docs.python.org/)
- [Docker文档](https://docs.docker.com/)
- [Kubernetes文档](https://kubernetes.io/docs/)
- [PostgreSQL文档](https://www.postgresql.org/docs/)

### AI模型文档
- [Anthropic Claude](https://docs.anthropic.com/)
- [OpenAI GPT](https://platform.openai.com/docs)
- [Google Gemini](https://ai.google.dev/docs)
- [Groq](https://groq.com/docs)

## 📝 文档贡献

我们欢迎社区贡献来改进文档！

### 如何贡献
1. 在GitHub上Fork项目
2. 修改文档内容
3. 提交Pull Request

### 贡献指南
- 遵循[代码贡献指南](../CONTRIBUTING.md)
- 确保文档准确性和完整性
- 使用清晰、简洁的语言
- 提供实用的示例和代码

### 文档标准
- 使用Markdown格式
- 包含代码示例
- 添加适当的图表和表格
- 提供清晰的导航链接

## 📞 获取帮助

如果在使用文档过程中遇到问题，可以通过以下方式获取帮助：

- 📋 [GitHub Issues](https://github.com/afanty2021/moon-dev-ai-agents/issues) - 报告文档问题
- 💬 [Discord社区](https://discord.gg/moondev) - 实时讨论和交流
- 📧 support@moondev.ai - 联系技术支持

## 📹 视频资源

Anthropic 团队强调了理解 AI 代理系统每一行代码的重要性，因为这是一个新的技术范式，深入理解可以发掘许多优势。以下是详细的学习资源：

- [完整文档视频播放列表](https://www.youtube.com/playlist?list=PLXrNVMjRZUJg4M4uz52iGd1LhXXGVbIFz)
- [Anthropic 构建有效代理的研究文章](https://www.anthropic.com/research/building-effective-agents)

### 各代理专门文档
每个代理都有独立的详细文档，位于 `docs/` 文件夹中：
- [TradingAgent 文档](agents/trading_agent.md)
- [RiskAgent 文档](agents/risk_agent.md)
- [StrategyAgent 文档](agents/strategy_agent.md)
- [SentimentAgent 文档](agents/sentiment_agent.md)
- 等等...

---

**最后更新**: 2024年11月19日
**文档版本**: 1.0.0
**维护者**: Moon Dev Team

感谢您使用 Moon Dev AI Agents！ 🚀