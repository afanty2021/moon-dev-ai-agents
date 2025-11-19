# 开发环境设置指南

## 概述

本指南将帮助您在不同操作系统上设置 Moon Dev AI Agents 的完整开发环境。支持 Windows、macOS 和 Linux 平台。

## 系统要求

### 最低硬件要求
- **CPU**: 4核心 2.0GHz
- **内存**: 8GB RAM (推荐 16GB)
- **存储**: 10GB 可用磁盘空间
- **网络**: 稳定的互联网连接

### 软件要求
- **Python**: 3.8+ (推荐 3.9+)
- **Git**: 最新版本
- **Conda**: Miniconda 或 Anaconda

## 平台特定设置

### Windows (10/11)

#### 1. 安装 Chocolatey (推荐)
```powershell
# 以管理员身份运行 PowerShell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

#### 2. 安装 Git
```powershell
choco install git
```

#### 3. 安装 Miniconda
```powershell
choco install miniconda3
# 或手动下载: https://docs.conda.io/en/latest/miniconda.html
```

#### 4. 验证安装
```powershell
git --version
conda --version
```

#### 5. 重启 PowerShell 并继续设置

### macOS

#### 1. 安装 Homebrew
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. 安装 Git (如果未预装)
```bash
brew install git
```

#### 3. 安装 Miniconda
```bash
brew install miniconda
# 或手动下载: https://docs.conda.io/en/latest/miniconda.html
```

#### 4. 配置 Shell
```bash
# 对于 zsh (默认 macOS shell)
echo 'eval "$(conda shell.zsh hook)"' >> ~/.zshrc
source ~/.zshrc

# 对于 bash
echo 'eval "$(conda shell.bash hook)"' >> ~/.bash_profile
source ~/.bash_profile
```

#### 5. 验证安装
```bash
git --version
conda --version
```

### Linux (Ubuntu/Debian)

#### 1. 更新系统包
```bash
sudo apt update && sudo apt upgrade -y
```

#### 2. 安装必需工具
```bash
sudo apt install -y curl wget git build-essential
```

#### 3. 下载并安装 Miniconda
```bash
# 64位 Linux
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

# 按照提示安装，同意许可条款，选择安装位置
```

#### 4. 配置 Shell
```bash
# 对于 bash
echo 'eval "$(conda shell.bash hook)"' >> ~/.bashrc
source ~/.bashrc

# 对于 zsh
echo 'eval "$(conda shell.zsh hook)"' >> ~/.zshrc
source ~/.zshrc
```

#### 5. 验证安装
```bash
git --version
conda --version
```

## 项目设置

### 1. 克隆仓库
```bash
git clone https://github.com/your-username/moon-dev-ai-agents.git
cd moon-dev-ai-agents
```

### 2. 创建 Conda 环境
```bash
# 使用现有的 tflow 环境 (推荐)
conda activate tflow

# 或创建新的环境
conda create -n moon-dev python=3.9 -y
conda activate moon-dev
```

### 3. 安装依赖
```bash
# 安装项目依赖
pip install -r requirements.txt

# 如果 requirements.txt 不存在，安装核心依赖
pip install pandas numpy requests termcolor anthropic openai groq
```

### 4. 配置环境变量

#### 创建 .env 文件
```bash
cp .env_example .env
```

#### 编辑 .env 文件
```bash
# 使用你喜欢的编辑器
nano .env  # Linux/macOS
notepad .env  # Windows
```

#### 必需的 API 密钥
```bash
# AI 服务
ANTHROPIC_KEY=your_claude_api_key_here
OPENAI_KEY=your_openai_api_key_here
GROQ_API_KEY=your_groq_api_key_here
GEMINI_KEY=your_gemini_api_key_here
DEEPSEEK_KEY=your_deepseek_api_key_here
GROK_API_KEY=your_xai_api_key_here

# 交易 APIs
BIRDEYE_API_KEY=your_birdeye_api_key_here
MOONDEV_API_KEY=your_moondev_api_key_here
COINGECKO_API_KEY=your_coingecko_api_key_here

# 区块链
SOLANA_PRIVATE_KEY=your_solana_private_key_here
HYPER_LIQUID_ETH_PRIVATE_KEY=your_hyperliquid_private_key_here

# 其他配置
RPC_ENDPOINT=https://api.mainnet-beta.solana.com
```

### 5. 获取 API 密钥指南

#### AI 服务密钥
- **Anthropic Claude**: https://console.anthropic.com/
- **OpenAI**: https://platform.openai.com/api-keys
- **Groq**: https://console.groq.com/keys
- **Google Gemini**: https://makersuite.google.com/app/apikey
- **DeepSeek**: https://platform.deepseek.com/
- **xAI (Grok)**: https://console.x.ai/

#### 交易数据密钥
- **BirdEye**: https://birdeye.so/developers
- **MoonDev**: 联系项目维护者获取
- **CoinGecko**: https://www.coingecko.com/en/api

### 6. 验证安装

#### 测试 Python 环境
```bash
python -c "import pandas; print('Pandas OK')"
python -c "import anthropic; print('Anthropic OK')"
```

#### 测试项目组件
```bash
# 测试基础导入
python -c "
from src.config import *
from src.models.model_factory import ModelFactory
from src.nice_funcs import token_overview
print('项目组件导入成功')
"
```

#### 运行单个代理测试
```bash
# 测试交易代理
python src/agents/trading_agent.py

# 测试风险代理
python src/agents/risk_agent.py
```

## IDE 配置

### VS Code 配置

#### 1. 安装扩展
```bash
# 命令行安装
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-python.black-formatter
code --install-extension ms-python.isort
```

#### 2. 创建 .vscode/settings.json
```json
{
    "python.defaultInterpreterPath": "~/miniconda3/envs/tflow/bin/python",
    "python.formatting.provider": "black",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "python.linting.pycodestyleEnabled": true,
    "python.sortImports.args": ["--profile", "black"],
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    }
}
```

### PyCharm 配置

#### 1. 设置 Python 解释器
- 打开 File → Settings → Project → Python Interpreter
- 点击齿轮图标 → Add → Conda Environment
- 选择现有的 `tflow` 环境或创建新的

#### 2. 配置代码风格
- 设置 Code Style → Python 为 Black
- 启用 Import organizer 为 isort

## 数据库设置 (可选)

### Redis 缓存 (推荐用于生产环境)
```bash
# Ubuntu/Debian
sudo apt install redis-server

# macOS
brew install redis

# Windows (WSL)
sudo apt install redis-server

# 启动 Redis
redis-server

# 测试连接
redis-cli ping
```

### PostgreSQL (用于高级分析)
```bash
# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib

# macOS
brew install postgresql
brew services start postgresql

# 创建数据库
sudo -u postgres createdb moon_dev_agents
```

## Docker 设置 (可选)

### 1. 安装 Docker
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# macOS
brew install docker docker-compose

# Windows
# 下载 Docker Desktop: https://www.docker.com/products/docker-desktop
```

### 2. 构建项目镜像
```bash
# 创建 Dockerfile (项目根目录)
docker build -t moon-dev-agents .

# 运行容器
docker run -it --env-file .env moon-dev-agents
```

## 验证完整设置

### 1. 运行健康检查脚本
```bash
# 创建 health_check.py
cat > health_check.py << 'EOF'
#!/usr/bin/env python3
import sys
import os
from pathlib import Path

def check_environment():
    """检查环境设置"""
    print("🔍 检查开发环境...")

    # 检查 Python 版本
    if sys.version_info < (3, 8):
        print("❌ Python 版本过低，需要 3.8+")
        return False
    print(f"✅ Python 版本: {sys.version}")

    # 检查必需的包
    required_packages = [
        'pandas', 'numpy', 'requests', 'anthropic',
        'openai', 'termcolor'
    ]

    for package in required_packages:
        try:
            __import__(package)
            print(f"✅ {package} 已安装")
        except ImportError:
            print(f"❌ {package} 未安装")
            return False

    # 检查项目文件
    project_files = [
        'src/config.py',
        'src/main.py',
        'src/nice_funcs.py',
        'src/models/model_factory.py'
    ]

    for file_path in project_files:
        if Path(file_path).exists():
            print(f"✅ {file_path} 存在")
        else:
            print(f"❌ {file_path} 不存在")
            return False

    # 检查环境变量
    env_file = Path('.env')
    if env_file.exists():
        print("✅ .env 文件存在")

        # 检查关键环境变量
        with open(env_file) as f:
            content = f.read()

        required_vars = [
            'ANTHROPIC_KEY', 'OPENAI_KEY', 'BIRDEYE_API_KEY'
        ]

        for var in required_vars:
            if f"{var}=" in content and f"{var}=your_" not in content:
                print(f"✅ {var} 已配置")
            else:
                print(f"⚠️  {var} 需要配置")
    else:
        print("❌ .env 文件不存在")
        return False

    print("\n🎉 环境检查完成！")
    return True

if __name__ == "__main__":
    if check_environment():
        print("环境设置正确，可以开始开发！")
        sys.exit(0)
    else:
        print("请修复上述问题后重试。")
        sys.exit(1)
EOF

# 运行健康检查
python health_check.py
```

### 2. 运行系统集成测试
```bash
# 测试模型工厂
python -c "
from src.models.model_factory import ModelFactory
factory = ModelFactory()
print('✅ ModelFactory 初始化成功')
"

# 测试配置加载
python -c "
from src.config import *
print(f'✅ 配置加载成功，监控代币数量: {len(MONITORED_TOKENS)}')
"
```

## 常见问题解决

### Python 版本问题
```bash
# 检查当前 Python 版本
python --version

# 如果版本过低，使用 conda 升级
conda install python=3.9
```

### 依赖冲突
```bash
# 清理并重新安装
pip uninstall -y -r requirements.txt
pip install -r requirements.txt --no-cache-dir
```

### 权限问题 (Linux/macOS)
```bash
# 修复 conda 权限
chmod +x ~/miniconda3/bin/*
sudo chown -R $USER ~/miniconda3
```

### 网络连接问题
```bash
# 配置代理 (如果需要)
pip config set global.proxy http://proxy.example.com:8080

# 或使用国内镜像
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple/
```

## 性能优化建议

### 1. 使用 SSD 存储
- 将项目放在 SSD 上提高 I/O 性能
- 配置适当的内存和交换空间

### 2. 内存优化
```bash
# 增加 conda 环境内存限制
conda config --set env_prompt '({name}) '
```

### 3. 并行处理
```bash
# 安装并行处理库
pip install joblib multiprocessing
```

## 下一步

完成环境设置后，你可以：

1. 阅读 [CLAUDE.md](../CLAUDE.md) 了解项目架构
2. 查看 [快速开始指南](quickstart.md) 运行第一个代理
3. 浏览 [API 文档](api-reference.md) 了解可用功能
4. 加入 [Discord 社区](https://discord.gg/) 获取支持

## 获取帮助

如果遇到问题，请：

1. 查看 [故障排除指南](troubleshooting.md)
2. 搜索 [GitHub Issues](https://github.com/your-repo/issues)
3. 加入 Discord 社区求助
4. 提交新的 Issue 描述问题

恭喜！你已经成功设置了 Moon Dev AI Agents 的开发环境。现在可以开始探索 AI 交易的世界了！ 🚀