# Programming

记录不同编程语言的特点、差异和设计哲学。

## 本地启动预览

```bash
uvx zensical serve
```

## 开发环境设置

```bash
# 安装 WebP 转换工具
brew install webp

# 安装 pre-commit
brew install pre-commit

# 启用 Git hooks（仅需执行一次）
pre-commit install
```

> 配置完成后，commit 时会自动将 PNG/JPG/GIF 图片转换为 WebP 格式并更新 Markdown 引用。
