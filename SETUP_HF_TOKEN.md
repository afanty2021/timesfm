# TimesFM - HF_TOKEN 配置指南

## 📝 配置步骤

### 1. 获取 Hugging Face Token

访问 [Hugging Face Tokens](https://huggingface.co/settings/tokens) 创建一个新 token。

**推荐权限**：
- ✅ Read access to contents of all public models
- ✅ Read access to contents of all public datasets

### 2. 配置到 .zshrc

```bash
# 编辑 .zshrc
nano ~/.zshrc

# 添加以下行
export HF_TOKEN="hf_your_token_here"
# 或
export HUGGING_FACE_HUB_TOKEN="hf_your_token_here"
```

### 3. 重新加载配置

```bash
source ~/.zshrc
```

### 4. 验证配置

```bash
# 方法1: 直接检查
echo $HF_TOKEN

# 方法2: 运行测试脚本
python test_hf_token.py
```

## 🚀 快速测试

```bash
# 1. 设置临时 token（用于测试）
export HF_TOKEN="hf_your_token_here"

# 2. 运行测试
python test_hf_token.py
```

## 🔧 永久配置

将以下内容添加到 `~/.zshrc`：

```bash
# Hugging Face Token
export HF_TOKEN="hf_your_actual_token"
```

然后运行：
```bash
source ~/.zshrc
```

## 📌 注意事项

1. **Token 安全**: 不要将 token 提交到 Git 仓库
2. **权限选择**: 仅授予必要的权限
3. **Token 更新**: 定期更新 token 以提高安全性

## ❓ 常见问题

### Q: 我需要 token 吗？
A: 对于公开模型（如 `google/timesfm-2.5-200m-pytorch`），通常不需要。但配置后可以避免速率限制。

### Q: 如何检查 token 是否有效？
A: 运行 `python test_hf_token.py` 或使用:
```python
from huggingface_hub import whoami
print(whoami())
```

### Q: TimesFM 可以离线使用吗？
A: 首次需要下载模型，之后可以离线使用。
