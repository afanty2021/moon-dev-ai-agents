# 故障排除指南

## 概述

本指南涵盖了 Moon Dev AI Agents 在设置、配置和运行过程中可能遇到的常见问题及其解决方案。

## 快速诊断

### 运行诊断脚本
```bash
# 下载并运行诊断脚本
curl -o diagnose.py https://raw.githubusercontent.com/your-repo/scripts/diagnose.py
python diagnose.py

# 或者使用内置诊断
python -c "
import sys
import os
import subprocess

def quick_diagnose():
    print('🔍 快速诊断...')

    # 检查 Python
    try:
        import pandas, anthropic
        print('✅ 核心依赖正常')
    except ImportError as e:
        print(f'❌ 依赖缺失: {e}')

    # 检查环境变量
    if os.path.exists('.env'):
        print('✅ .env 文件存在')
    else:
        print('❌ .env 文件缺失')

quick_diagnose()
"
```

## 安装问题

### 1. Conda 环境问题

#### 问题：conda 命令未找到
```bash
# 症状
bash: conda: command not found

# 解决方案
# 方法1: 重新安装 conda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

# 方法2: 添加到 PATH (临时)
export PATH=~/miniconda3/bin:$PATH

# 方法3: 永久添加到 shell 配置
echo 'export PATH=~/miniconda3/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

#### 问题：环境激活失败
```bash
# 症状
conda activate tflow
CommandNotFoundError: Your shell has not been properly configured

# 解决方案
# 初始化 conda for shell
conda init bash    # bash
conda init zsh     # zsh
conda init fish    # fish

# 重启终端或运行
source ~/.bashrc   # bash
source ~/.zshrc    # zsh
```

#### 问题：Python 版本不兼容
```bash
# 症状
ImportError: cannot import name 'abc' from 'collections'

# 解决方案
# 检查当前版本
python --version

# 升级到 Python 3.9+
conda install python=3.9

# 或创建新环境
conda create -n moon-dev python=3.9 -y
conda activate moon-dev
```

### 2. 依赖安装问题

#### 问题：pip 安装失败
```bash
# 症状
ERROR: Could not install packages due to an EnvironmentError

# 解决方案
# 方法1: 使用 --user 标志
pip install --user package_name

# 方法2: 升级 pip
python -m pip install --upgrade pip

# 方法3: 清理缓存
pip cache purge
pip install package_name --no-cache-dir

# 方法4: 使用 conda
conda install package_name
```

#### 问题：依赖冲突
```bash
# 症状
ERROR: pip's dependency resolver does not currently take into account

# 解决方案
# 方法1: 重新创建环境
conda create -n fresh-env python=3.9 -y
conda activate fresh-env
pip install -r requirements.txt

# 方法2: 强制重新安装
pip install -r requirements.txt --force-reinstall --no-deps

# 方法3: 逐个安装并解决冲突
pip install pandas numpy
pip install requests
pip install anthropic
# ... 依此类推
```

#### 问题：编译错误
```bash
# 症状
error: Microsoft Visual C++ 14.0 is required

# 解决方案 (Windows)
# 安装 Visual Studio Build Tools
# https://visualstudio.microsoft.com/visual-cpp-build-tools/

# 或使用预编译包
pip install package_name --only-binary=all

# 解决方案 (Linux)
sudo apt install build-essential python3-dev

# 解决方案 (macOS)
xcode-select --install
brew install gcc
```

## API 密钥问题

### 1. 密钥配置错误

#### 问题：API 密钥无效
```bash
# 症状
anthropic.AuthenticationError: Invalid API key

# 解决方案
# 1. 检查密钥格式
cat .env | grep ANTHROPIC_KEY

# 2. 确认密钥没有空格或特殊字符
# 正确: ANTHROPIC_KEY=sk-ant-api03-...
# 错误: ANTHROPIC_KEY= sk-ant-api03-...  (注意空格)

# 3. 重新生成密钥
# 访问: https://console.anthropic.com/
```

#### 问题：环境变量未加载
```bash
# 症状
KeyError: 'ANTHROPIC_KEY'

# 解决方案
# 方法1: 手动加载环境变量
export $(cat .env | xargs)

# 方法2: 使用 python-dotenv
pip install python-dotenv

# 在代码中添加:
from dotenv import load_dotenv
load_dotenv()
```

#### 问题：密钥权限不足
```bash
# 症状
anthropic.PermissionError: Insufficient permissions

# 解决方案
# 检查 API 计划和权限
# 1. 登录 AI 服务提供商控制台
# 2. 确认账户状态正常
# 3. 检查 API 使用配额
# 4. 确认所需的权限已启用
```

### 2. 网络连接问题

#### 问题：API 请求超时
```bash
# 症状
requests.exceptions.Timeout: Request timed out

# 解决方案
# 方法1: 增加超时时间
# 在代码中修改:
response = requests.get(url, timeout=30)

# 方法2: 配置代理
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=https://proxy.example.com:8080

# 方法3: 使用不同的网络
# 尝试切换网络或使用 VPN
```

#### 问题：DNS 解析失败
```bash
# 症状
requests.exceptions.ConnectionError: Failed to resolve

# 解决方案
# 方法1: 更换 DNS
sudo systemctl stop systemd-resolved
sudo rm /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# 方法2: 刷新 DNS 缓存
sudo systemctl restart systemd-resolved  # Ubuntu
sudo dscacheutil -flushcache             # macOS
ipconfig /flushdns                      # Windows
```

## 运行时问题

### 1. 模块导入错误

#### 问题：找不到项目模块
```bash
# 症状
ModuleNotFoundError: No module named 'src'

# 解决方案
# 方法1: 添加到 PYTHONPATH
export PYTHONPATH="${PYTHONPATH}:$(pwd)"

# 方法2: 使用 -m 运行
python -m src.main

# 方法3: 创建 setup.py (项目根目录)
cat > setup.py << 'EOF'
from setuptools import setup, find_packages

setup(
    name='moon-dev-ai-agents',
    packages=find_packages(),
    install_requires=[
        'pandas', 'numpy', 'requests', 'anthropic', 'openai'
    ]
)
EOF

pip install -e .
```

#### 问题：相对导入错误
```bash
# 症状
ImportError: attempted relative import with no known parent package

# 解决方案
# 确保从项目根目录运行
cd /path/to/moon-dev-ai-agents
python src/agents/trading_agent.py

# 不要这样:
cd src/agents
python trading_agent.py
```

### 2. 配置文件问题

#### 问题：配置文件缺失
```bash
# 症状
FileNotFoundError: [Errno 2] No such file or directory: 'config.py'

# 解决方案
# 确保在正确的目录
ls -la src/config.py

# 如果缺失，从模板创建
cp src/config_example.py src/config.py
```

#### 问题：配置值类型错误
```bash
# 症状
TypeError: 'str' object cannot be interpreted as an integer

# 解决方案
# 检查 config.py 中的类型
# 错误: usd_size = "25"
# 正确: usd_size = 25

# 验证配置
python -c "
from src.config import *
print(f'usd_size: {usd_size} (type: {type(usd_size)})')
"
```

### 3. 数据获取问题

#### 问题：API 限制
```bash
# 症状
429 Too Many Requests

# 解决方案
# 方法1: 实现重试机制
import time
import random

def api_call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt + random.random()
            time.sleep(wait_time)

# 方法2: 降低请求频率
# 在 config.py 中调整:
API_RATE_LIMIT = 1  # 每秒请求数
```

#### 问题：数据格式错误
```bash
# 症状
json.JSONDecodeError: Expecting value

# 解决方案
# 添加错误处理
try:
    data = response.json()
except json.JSONDecodeError:
    print(f"Invalid JSON response: {response.text}")
    # 使用空数据或默认值
    data = {}
```

## 性能问题

### 1. 内存不足

#### 问题：内存使用过高
```bash
# 症状
MemoryError: Unable to allocate array
Killed (程序被终止)

# 解决方案
# 方法1: 监控内存使用
pip install memory-profiler

# 在代码中添加:
@profile
def memory_intensive_function():
    # 你的代码
    pass

# 运行分析
python -m memory_profiler your_script.py

# 方法2: 优化数据结构
# 使用更高效的数据类型
df = pd.DataFrame({'column': column_data}, dtype='float32')

# 方法3: 分批处理
def process_in_chunks(data, chunk_size=1000):
    for i in range(0, len(data), chunk_size):
        yield data[i:i + chunk_size]

# 方法4: 及时释放内存
import gc
del large_variable
gc.collect()
```

### 2. 处理速度慢

#### 问题：响应时间过长
```bash
# 症状
程序运行缓慢，响应时间 > 10秒

# 解决方案
# 方法1: 使用并发处理
import concurrent.futures

def parallel_process(items, func, max_workers=4):
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(executor.map(func, items))
    return results

# 方法2: 缓存结果
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_computation(param1, param2):
    # 昂贵的计算
    return result

# 方法3: 向量化操作
# 避免循环，使用 pandas/numpy 向量化
df['new_column'] = df['column1'] * df['column2']
```

## 代理特定问题

### 1. TradingAgent 问题

#### 问题：交易执行失败
```bash
# 症状
Trade execution failed: Insufficient balance

# 解决方案
# 检查账户余额
python -c "
from src.nice_funcs import get_account_balance
balance = get_account_balance()
print(f'账户余额: ${balance}')
"

# 检查交易参数
python -c "
from src.config import usd_size, MAX_POSITION_PERCENTAGE
print(f'交易金额: ${usd_size}')
print(f'最大仓位百分比: {MAX_POSITION_PERCENTAGE}%')
"
```

#### 问题：滑点过高
```bash
# 症状
实际成交价格与预期相差过大

# 解决方案
# 调整滑点设置
# 在 config.py 中:
SLIPPAGE = 299  # 2.99% 滑点容忍度

# 或使用限价单
def limit_order_buy(symbol, amount, max_price):
    # 限价买入逻辑
    pass
```

### 2. ModelFactory 问题

#### 问题：模型初始化失败
```bash
# 症状
ModelFactoryError: Failed to initialize model

# 解决方案
# 检查模型可用性
python -c "
from src.models.model_factory import ModelFactory
factory = ModelFactory()
print('可用模型:', factory.available_models())
"

# 测试单个模型
python -c "
import anthropic
client = anthropic.Anthropic()
response = client.messages.create(
    model='claude-3-haiku-20240307',
    max_tokens=10,
    messages=[{'role': 'user', 'content': 'Hi'}]
)
print('Claude 连接正常')
"
```

### 3. 数据收集问题

#### 问题：OHLCV 数据缺失
```bash
# 症状
No OHLCV data available for token

# 解决方案
# 检查代币地址
python -c "
from src.nice_funcs import token_overview
try:
    overview = token_overview('YOUR_TOKEN_ADDRESS')
    print('代币数据正常')
except Exception as e:
    print(f'代币数据错误: {e}')
"

# 手动获取数据
python -c "
from src.ohlcv_collector import OHLCVCollector
collector = OHLCVCollector()
data = collector.get_data('TOKEN_ADDRESS', '1H', 24)
print(f'获取到 {len(data)} 条数据')
"
```

## 调试技巧

### 1. 日志调试
```bash
# 启用详细日志
export LOG_LEVEL=DEBUG

# 或在代码中设置
import logging
logging.basicConfig(level=logging.DEBUG)

# 使用结构化日志
logger = logging.getLogger(__name__)
logger.info("开始处理代币: %s", token_address)
logger.error("API 调用失败: %s", str(error))
```

### 2. 断点调试
```python
# 使用 pdb 调试器
import pdb

def your_function():
    # 在这里设置断点
    pdb.set_trace()

    # 你的代码
    result = some_computation()
    return result

# 运行
python your_script.py
```

### 3. 性能分析
```python
# 使用 cProfile 分析性能
python -m cProfile -o profile_output your_script.py

# 分析结果
python -c "
import pstats
p = pstats.Stats('profile_output')
p.sort_stats('cumulative').print_stats(10)
"
```

## 环境特定问题

### 1. Windows 问题

#### 问题：路径分隔符
```python
# 症状
FileNotFoundError: [Errno 2] No such file or directory: 'src\\agents\\trading_agent.py'

# 解决方案
# 使用 pathlib
from pathlib import Path
file_path = Path('src') / 'agents' / 'trading_agent.py'

# 或使用 os.path
import os
file_path = os.path.join('src', 'agents', 'trading_agent.py')
```

#### 问题：编码问题
```python
# 症状
UnicodeDecodeError: 'utf-8' codec can't decode

# 解决方案
# 指定编码
with open('file.txt', 'r', encoding='utf-8') as f:
    content = f.read()
```

### 2. macOS 问题

#### 问题：Python 版本冲突
```bash
# 症状
多个 Python 版本混乱

# 解决方案
# 使用 brew 管理的 Python
brew install python@3.9

# 或使用 pyenv
brew install pyenv
pyenv install 3.9.13
pyenv global 3.9.13
```

### 3. Linux 问题

#### 问题：权限问题
```bash
# 症状
PermissionError: [Errno 13] Permission denied

# 解决方案
# 检查文件权限
ls -la your_file.py

# 修复权限
chmod 644 your_file.py  # 文件
chmod 755 your_script.py # 可执行脚本

# 或更改所有者
sudo chown $USER:$USER your_file.py
```

## 获取帮助

### 1. 收集诊断信息
```bash
# 创建诊断报告
python -c "
import sys
import os
import platform
import pandas
import anthropic

print('=== 诊断报告 ===')
print(f'Python: {sys.version}')
print(f'系统: {platform.system()} {platform.release()}')
print(f'Pandas: {pandas.__version__}')
print(f'Anthropic: {anthropic.__version__}')
print(f'工作目录: {os.getcwd()}')
print(f'Python 路径: {sys.path[:3]}')
print('================')
" > diagnostic_report.txt
```

### 2. 社区支持
- **GitHub Issues**: 提交新 issue 并附上诊断报告
- **Discord 社区**: 加入 #support 频道
- **文档**: 查看最新文档和 FAQ

### 3. 问题报告模板
```markdown
## 问题描述
[详细描述遇到的问题]

## 重现步骤
1. 执行命令: [运行的命令]
2. 期望结果: [期望发生什么]
3. 实际结果: [实际发生了什么]

## 环境信息
- 操作系统: [Windows/macOS/Linux 版本]
- Python 版本: [Python 版本]
- 项目版本: [Git commit hash 或版本号]

## 错误信息
```
[粘贴完整的错误信息]
```

## 尝试的解决方案
[已经尝试过的解决方法]
```

## 预防措施

### 1. 定期更新
```bash
# 定期更新依赖
pip install --upgrade -r requirements.txt

# 更新项目
git pull origin main
```

### 2. 备份配置
```bash
# 备份重要文件
cp .env .env.backup
cp config.py config.py.backup
```

### 3. 监控系统
```bash
# 设置监控脚本
#!/bin/bash
# monitor.sh
python health_check.py
if [ $? -ne 0 ]; then
    echo "系统异常，发送通知"
    # 发送邮件或通知
fi
```

通过遵循本指南，你应该能够解决大部分常见问题。如果问题仍然存在，请不要犹豫，寻求社区支持！