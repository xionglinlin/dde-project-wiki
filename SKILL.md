---
name: dde-project-wiki
description: |
  为 DDE 项目构建和维护持久化知识库。基于 Karpathy LLM Wiki 模式适配 deepin 技术栈。

  TRIGGER when:
  - 用户修改项目文件 (.cpp, .h, .hpp, .c, .cc, .cxx, .qml, .js, .go, .proto, .sh, .py, CMakeLists.txt, *.cmake, *.pro)
  - 用户询问项目架构 ("这个项目怎么工作的", "分析一下框架", "解释这个模块", "这个模块是干什么的")
  - 用户分析 bug 或功能 ("这个 bug 可能的原因", "分析一下这个问题", "帮我看看这个错误")
  - git 显示 wiki 上次更新后有新变更
  - 用户说 "更新 wiki", "同步文档", "lint wiki", "检查文档"

  DO NOT TRIGGER when:
  - 用户只想读取单个文件
  - 用户询问与项目无关的通用编程问题
  - 用户只是格式化代码或添加注释
  - 用户询问外部库或第三方工具的使用
---

# DDE 项目 Wiki

为 DDE 项目自动构建和维护持久化知识库。基于 Karpathy LLM Wiki 模式适配 deepin 技术栈：LLM 读取项目源码、文档和 git 历史，构建结构化的 wiki 文档，并在代码演进时保持同步。你写代码，LLM 写文档。

## 决策树

```
项目中是否存在 wiki/ 目录？
├─ 否 → 用户说 "初始化 wiki"、"建立知识库"、"创建项目文档" → 执行 INIT
├─ 是 →
│   ├─ 有代码变更（上次 log.md 记录后）？ → 执行 INGEST
│   ├─ 用户询问架构、功能、bug？ → 执行 QUERY
│   ├─ 用户说 "lint wiki"、"检查文档"、"文档健康检查" ？ → 执行 LINT
│   └─ 用户完成 bug 分析或功能分析？ → 执行 FILE_ANALYSIS
```

## Wiki 目录结构

```
wiki/
├── index.md            # 内容目录 — 所有页面链接 + 摘要
├── log.md              # 操作日志 — 按时间追加的操作记录
├── schema.md           # Wiki 规范 — 与用户共同演进的约定
├── overview.md         # 项目总览 — 架构、技术栈、关键决策
├── concepts/           # 架构概念页面
│   ├── signal-slot-patterns.md
│   ├── qml-architecture.md
│   ├── build-system.md
│   └── ...
├── entities/           # 组件实体页面
│   ├── cpp/            # C++ 类/组件
│   ├── qml/            # QML 组件
│   ├── go/             # Go 服务/包
│   └── proto/          # 协议定义
├── bug-analysis/       # Bug 分析记录
└── queries/            # 有价值的查询归档
```

## 操作

### INIT（初始化）

1. 扫描代码库结构 — 识别语言（C++、QML、Go、Shell、Python）、框架（Qt）、构建工具（CMake、go mod）
2. 读取现有文档 — README.md、README.zh_CN.md、CLAUDE.md、Qt 文档注释、配置文件
3. 读取 git 历史 — `git log --oneline -30` 获取近期上下文
4. 编写 `overview.md` — 项目用途、技术栈、架构概览、各子项目关系、构建部署信息
5. 为主要组件创建实体页面 — C++ 类放 `entities/cpp/`，QML 组件放 `entities/qml/`，Go 服务放 `entities/go/`
6. 为架构模式创建概念页面 — Qt Signal/Slot 机制、QML 与 C++ 交互、Go 服务架构、构建系统
7. 生成 mermaid 架构图
8. 创建 `index.md`、`log.md`、`schema.md`
9. 在项目 CLAUDE.md 中添加：`本项目在 wiki/ 目录维护知识库，架构问题请查阅 wiki/index.md。代码变更后保持 wiki 更新。`

### INGEST（代码变更摄入）

1. 检测变更 — 当前会话中修改的文件，或会话开始时 `git diff --name-only` 自上次 log.md 日期
2. 读取变更文件，理解变更性质
3. 分类：结构性变更（新模块、重命名、API 变化、架构调整）vs 琐碎变更（格式化、注释、小修复）
4. 结构性变更：更新相关 entity/concept 页面
5. 架构变动：更新 `overview.md` 和 mermaid 图
6. 更新交叉引用和 `index.md`
7. 追加到 `log.md`
8. 跳过琐碎变更 — 不污染 wiki

### QUERY（查询）

1. 读取 `index.md` 找到相关页面
2. 读取 wiki 页面 + 必要时读取源码补充细节
3. 综合回答，引用 wiki 页面和源码位置 `src/path/file.cpp:42`
4. 如果回答产生有价值的综合分析，归档到 `queries/`
5. 更新 `index.md` 和 `log.md`

### LINT（健康检查）

1. 对比 wiki 与实际代码库状态
2. 检查：所有主要模块是否都有文档？实体页面引用的文件是否还存在？架构描述是否当前？是否有新模块缺少 wiki 页面？
3. 修复过时内容，标记需要人工判断的问题
4. 建议文档缺口和待探索的问题
5. 追加到 `log.md`

### FILE_ANALYSIS（分析归档）

当用户完成 bug 分析或功能分析后执行：

1. 判断分析是否有持续价值：
   - 问题根因分析 → 值得保存
   - 修复方案 → 值得保存
   - 代码流程分析 → 值得保存
   - 简单的 "这个文件在哪" → 不保存

2. 检查是否提供了 bug 关联信息：
   - 用户提供了 bug 编号（如 PMS-12345）→ 记录到 `pms` 字段
   - 用户提供了链接 → 记录到 `pms_url` 字段
   - 用户未提供 → 不记录关联信息

3. 创建 `bug-analysis/` 页面

4. 更新相关 entity/concept 页面的交叉引用
5. 更新 `index.md`
6. 追加到 `log.md`

## 页面规范

每个 wiki 页面都有 YAML frontmatter：

```yaml
---
title: 页面标题
type: concept | entity | bug-analysis | query | overview
tags: [标签1, 标签2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
related_files: [src/path/file.cpp, ...]
---
```

实体页面额外字段：
```yaml
# C++ 类
signals: [signal1(args), ...]
slots: [slot1(args), ...]

# QML 组件
cpp_backend: [src/controller.cpp, ...]

# Go 服务
depends_on: [DBus, systemd, ...]

# Bug 分析
pms: PMS-12345              # 可选
pms_url: https://...        # 可选
status: open | fixed | wontfix
```

- 使用 `[[wikilinks]]` 进行页面交叉引用
- 引用源码使用 `src/path/file.cpp:42` 格式
- 使用 mermaid 代码块绘制架构和流程图
- 每个页面以 1-2 句摘要开头

## index.md 格式

按类别组织。每条目：`- [页面标题](path.md) — 一行摘要 (N 个源文件, YYYY-MM-DD)`

## log.md 格式

追加式。每条目：`## [YYYY-MM-DD] 操作类型 | 标题`。可用 `grep "^## \[" wiki/log.md` 解析。

## 关键规则

- LLM 编写和维护 wiki。用户负责写代码和提问。
- 源码是真相来源 — wiki 文档它，永不与之矛盾。
- 关注结构性/架构性变更。跳过琐碎编辑。
- 每次操作都要更新 `index.md` 并追加到 `log.md`。
- 当代码和 wiki 不一致时，以代码为准 — 更新 wiki。
- 有价值的分析结果归档到 `bug-analysis/` — 知识持续累积。

详细工作流参见 [references/operations.md](references/operations.md)。
