# A 股股票预测 - TimesFM 使用指南

## 📊 项目简介

使用 Google 的 TimesFM（Time Series Foundation Model）进行 A 股股票价格预测。TimesFM 是一个零样本时间序列预测模型，无需训练即可使用。

### ✨ 特点

- **零样本预测**: 无需训练，直接使用
- **概率预测**: 提供 80% 置信区间
- **支持协变量**: 可结合成交量、技术指标等外部变量
- **高效推理**: GPU 加速，快速预测

## 🚀 快速开始

### 1. 安装依赖

```bash
# 基础依赖
pip install akshare timesfm[torch] matplotlib pandas numpy

# 如果需要协变量预测（XReg）
pip install timesfm[xreg]
```

### 2. 基础使用

```bash
# 预测平安银行 (000001)
python a_stock_forecast.py --stock 000001

# 预测贵州茅台 (600519)，预测 60 天
python a_stock_forecast.py --stock 600519 --horizon 60

# 预测宁德时代 (300750)，使用 5 年历史数据
python a_stock_forecast.py --stock 300750 --period 5y
```

### 3. 参数说明

| 参数 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| `--stock` | 股票代码 | 必需 | `000001`, `600519`, `300750` |
| `--period` | 历史数据周期 | `3y` | `1y`, `2y`, `3y`, `5y` |
| `--horizon` | 预测天数 | `30` | `10`, `30`, `60`, `90` |
| `--context` | 上下文长度 | `512` | `256`, `512`, `1024` |
| `--price-type` | 价格类型 | `close` | `open`, `close`, `high`, `low` |
| `--output` | 图表保存路径 | 自动生成 | `forecast.png` |

## 📈 使用示例

### 示例 1: 基础预测

```python
import akshare as ak
import timesfm
import numpy as np

# 获取数据
df = ak.stock_zh_a_hist(symbol="000001", period="daily", adjust="qfq")
prices = df['收盘'].values

# 预测
model = timesfm.TimesFM_2p5_200M_torch.from_pretrained("google/timesfm-2.5-200m-pytorch")
model.compile(timesfm.ForecastConfig(max_context=512, max_horizon=30))

forecast, quantiles = model.forecast(horizon=30, inputs=[prices[-512:]])
```

### 示例 2: 批量预测多只股票

```python
stocks = ["000001", "600519", "300750", "000858"]

for code in stocks:
    df = ak.stock_zh_a_hist(symbol=code, period="daily", adjust="qfq")
    prices = df['收盘'].values

    forecast, _ = model.forecast(horizon=30, inputs=[prices[-512:]])
    print(f"{code}: 预测价格 {forecast[0][-1]:.2f}")
```

## 🔧 高级功能

### 协变量预测 (XReg)

结合成交量、技术指标等外部变量提高预测准确性：

```bash
# 需要先安装 xreg 依赖
pip install timesfm[xreg]

# 使用协变量预测
python a_stock_forecast_with_covariates.py --stock 000001
```

支持的协变量：
- 成交量 (volume)
- 成交额 (amount)
- 换手率 (turnover)
- 技术指标 (MA, MACD, RSI 等)

### 自定义技术指标

```python
import pandas as pd
import talib

# 计算移动平均线
df['MA5'] = talib.MA(df['收盘'], timeperiod=5)
df['MA20'] = talib.MA(df['收盘'], timeperiod=20)

# 计算 MACD
df['MACD'], df['SIGNAL'], df['HIST'] = talib.MACD(df['收盘'])

# 计算 RSI
df['RSI'] = talib.RSI(df['收盘'], timeperiod=14)
```

## 📊 输出说明

### 预测结果

- **点预测**: 单一预测值（期望值）
- **分位数预测**: 10%, 20%, ..., 90% 分位数
- **置信区间**: 80% 置信区间（10%-90% 分位数）

### 图表说明

生成的图表包含：
1. **历史价格 + 预测**: 显示历史数据和预测趋势
2. **置信区间**: 阴影区域表示 80% 置信区间
3. **预测统计**: 当前价格、预测均价、涨跌幅等

## ⚠️ 注意事项

### 模型限制

1. **单变量预测**: TimesFM 主要针对单变量时间序列设计
2. **上下文限制**: 最大上下文长度为 16,384 个点
3. **预测范围**: 推荐预测 10-60 天，过长期准确性下降

### 风险提示

⚠️ **重要声明**: 本工具仅供学习和研究使用，不构成任何投资建议！

- 股票市场具有高度不确定性
- 历史表现不代表未来收益
- 模型预测仅供参考，实际投资需谨慎
- 请结合基本面分析、市场环境等多方面因素

### 数据来源

- **数据提供**: AkShare (开源财经数据接口)
- **数据延迟**: 免费数据可能有 1-3 个交易日延迟
- **数据准确性**: 请以官方数据为准

## 🛠️ 故障排除

### 常见问题

**Q: akshare 安装失败？**
```bash
pip install akshare -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**Q: 模型下载慢？**
```bash
# 设置 HF 镜像（国内用户）
export HF_ENDPOINT=https://hf-mirror.com
```

**Q: GPU 内存不足？**
```python
# 减小上下文长度
model.compile(timesfm.ForecastConfig(
    max_context=256,  # 从 512 减到 256
    max_horizon=30
))
```

**Q: 获取不到数据？**
```python
# 检查股票代码是否正确
# 000001-000999 为深圳股票
# 600000-600999 为上海股票
# 300000-300999 为创业板股票
```

## 📚 参考资料

- [TimesFM 论文](https://arxiv.org/abs/2310.10688)
- [TimesFM GitHub](https://github.com/google-research/timesfm)
- [AkShare 文档](https://akshare.akfamily.xyz/)
- [Hugging Face TimesFM](https://huggingface.co/google/timesfm-2.5-200m-pytorch)

## 📝 License

Apache-2.0

---

**免责声明**: 本项目仅供学习研究使用，不构成投资建议。投资有风险，入市需谨慎。
