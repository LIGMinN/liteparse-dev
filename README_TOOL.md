# liteparse — VA-shared 轻量 PDF 解析

快速本地 PDF 解析，返回带 bounding box 的结构化文本，接入 hermes-agent VA 工具体系（2 个 tool_id）。

## 捆绑模式

**P** (Project-Local venv) — 详见 `.toolconfig.yml`

## 目录结构

```
liteparse-dev-shared/
├── .toolconfig.yml         # 捆绑声明
├── README_TOOL.md          # 本文档
├── bin/
│   ├── liteparse-cli       # 统一入口（bash wrapper）
│   └── liteparse-cli.py    # 内部 Python 实现
├── .data/                  # 工具数据目录
├── .venv/                  # 独立 Python venv（liteparse 2.0.4 + .so）
├── packages/python/        # liteparse Python 包源码（maturin 构建）
├── crates/                 # liteparse Rust 源码（PyO3 绑定）
└── ...
```

## 快速开始

```bash
cd support/va-shared-support/liteparse-dev-shared

# 健康检查
bin/liteparse-cli health

# 解析 PDF（输出文本）
bin/liteparse-cli parse document.pdf

# 解析 PDF（输出 JSON 含 bbox）
bin/liteparse-cli parse document.pdf --format json

# 截图 PDF 第 1 页
bin/liteparse-cli screenshot document.pdf --page 1 --output page1.png
```

## HA 接入

注册 2 个 tool_id：

- `va_lp_parse` — 解析 PDF 文件
- `va_lp_screenshot` — 截取 PDF 页面

## 故障排查

**`liteparse not found`**：
- venv 未建立或 liteparse 包丢失。建议从 HA venv 复制：
  ```bash
  HA_VENV=../../platform/hermes-agent-custom-dev/.venv
  cp -r $HA_VENV/lib/python3.12/site-packages/liteparse .venv/lib/python3.12/site-packages/
  cp -r $HA_VENV/lib/python3.12/site-packages/liteparse-2.0.4.dist-info .venv/lib/python3.12/site-packages/
  ```
- 注：liteparse 是 maturin 构建的 PyO3 包（Rust + Python），PyPI 暂无 macOS x86_64 wheel，从源码构建需 Rust 工具链 + 系统库（libllhttp 等）。

**从源码构建（Apple Silicon 或 Linux）**：
```bash
cd packages/python
VIRTUAL_ENV=../../.venv uv pip install maturin
VIRTUAL_ENV=../../.venv maturin develop --release
```

## 升级与维护

- 当前 liteparse 版本：2.0.4
- API 版本：1.0
- HA 兼容范围：>=2.5,<3.0

## 历史

- 2026-06-14：迁移到 va-shared-support/，建独立 venv，加 bin/liteparse-cli bash wrapper（VA 捆绑规范 v2 阶段 2.3）
- 此前：在 agents-support/liteparse-dev-va/，依赖 HA venv 中的 liteparse
