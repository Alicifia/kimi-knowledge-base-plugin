# 本地知识库 · Local Knowledge Base Plugin for Kimi

> Copyright (c) 2026 **Alicifia** · MIT License（见 [LICENSE](LICENSE)）· 转载/二次开发请保留版权与许可声明

把任意本地文件夹变成真正可检索的知识库系统：文档导入、**向量检索**、**持久化存储**、**多文档关联**、**多知识库管理与跨库统一检索**，完全离线、零第三方依赖。

这是一个 [Kimi](https://www.kimi.com/)（Kimi Work / Kimi 桌面版）插件，安装后用自然语言即可搭建和查询知识库。

## 功能特性

- 📁 **指定任意本地目录**作为知识库，目录结构随意，递归扫描
- 🗂️ **多知识库注册表**（v0.2.0 新增）：想建几个库就建几个，`libraries` 一览、`use` 按名称切换、`forget` 注销（不删文件）
- 🌐 **跨库统一检索**（v0.2.0 新增）：`search --all` 一次搜遍全部已登记知识库，结果合并排序并标注来源库
- 🔍 **向量检索**：CJK 优化的 TF-IDF 向量化（中文单字+二元组 / 英文按词），余弦相似度，两级检索（文档级预筛 → 块级精排），30 万块的语料库单次查询约 3 秒
- 💾 **持久化存储**：每个库的索引自包含于自己目录下的 `.knowledge-base/index.sqlite`，拷贝文件夹即完成迁移
- 🔗 **多文档关联**：自动解析 `[[wikilink]]` 与 Markdown 链接（双向），并提供文档级相似度推荐
- 📚 **多格式**：`.md` / `.markdown` / `.txt` / `.epub` / `.mobi` / `.azw3` / `.azw`（PalmDOC 解压，支持双区段文件），可选 `.docx` / `.pdf`
- 🔄 **增量更新**：按内容哈希识别新增/修改/未变化文件；`--strict` 模式读取失败即退出
- 🔌 **零依赖**：仅 Python 3.8+ 标准库，无网络请求，数据不出本机

## 安装

在 Kimi 桌面版 / Kimi Work 中：

1. 克隆或下载本仓库
2. 让 agent 将本目录登记为个人插件（`kimi-plugin register-personal`），或在插件页「个人」页签安装
3. 安装后直接用自然语言提需求，例如「把 D:\笔记 建成知识库」

## 使用

一切通过自然语言驱动，agent 会调用内置 CLI `skills/knowledge-base/scripts/kb.py`：

| 你说 | 实际调用 |
|---|---|
| 把 X 目录建成知识库 | `kb.py init --root X` + `kb.py add X` |
| 在知识库里查 … | `kb.py search "…"` |
| 所有库一起搜 … | `kb.py search "…" --all` |
| 我有哪些知识库 | `kb.py libraries` |
| 切换到某个库 | `kb.py use --root <名称>` |
| 这篇和哪些文档相关 | `kb.py related <标题>` |
| 更新知识库 | `kb.py add X`（增量） |

CLI 也完全可以脱离 Kimi 单独使用：

```bash
python kb.py init --root /path/to/notes --name 我的笔记
python kb.py add /path/to/notes
python kb.py search "检索增强生成" --top 5
python kb.py search "检索增强生成" --all   # 跨全部已登记知识库
python kb.py libraries                   # 列出全部知识库
python kb.py related "我的笔记.md"
python kb.py status
```

## 工作原理

```
文档 → 文本抽取（mobi/azw3/azw 走 PalmDOC LZ77，epub 走 zip+HTML 剥离）
     → Markdown 标题层级切块（目标 ~500 字符，重叠 80）
     → CJK 单字+二元组 TF-IDF 向量化（L2 归一化，稀疏 dict 存 SQLite）
     → 检索：查询向量 × 文档级向量预筛 Top-N → 候选文档内块级精排
```

已知限制：HuffCDIC 压缩的老版 Kindle 文件暂不支持（会明确报错而非静默失败）。

## 更新日志

- **v0.2.0** — 多知识库注册表：不限数量的知识库统一登记（`init` 自动登记、`libraries` 一览、`use` 按名称切换、`forget` 注销不删文件）；跨库统一检索 `search --all`，结果合并排序并标注来源库。旧版配置自动迁移，已有知识库无需重建。
- **v0.1.0** — 首个公开发布：文档导入、向量检索、SQLite 持久化、多文档关联。

## 目录结构

```
├── kimi.plugin.json                    # 插件清单
├── LICENSE                             # MIT，Copyright (c) 2026 Alicifia
└── skills/knowledge-base/
    ├── SKILL.md                        # agent 使用说明（技能定义）
    └── scripts/kb.py                   # 知识库引擎（单文件，零依赖）
```

## License

MIT License — Copyright (c) 2026 **Alicifia**（[https://github.com/Alicifia](https://github.com/Alicifia)）。

允许自由使用、修改、分发，但必须在副本或主要部分中保留上述版权与许可声明。
