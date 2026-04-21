# DDE Project Wiki

为 DDE（Deepin Desktop Environment）项目定制的 Claude Code Skill，基于 [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 模式实现持久化知识库。

## 为什么创建这个 Skill

在日常开发中，我们经常遇到这样的场景：

> "帮我分析一下 dde-shell 的架构" → AI 给了一堆分析
> "这个 bug 可能是什么原因" → AI 又给了一堆分析
> "上次那个登录问题是怎么修的来着？" → AI 重新分析一遍...

**问题**：每次 AI 分析的结果都没有持久化，下次遇到类似问题又要重新分析一遍，知识无法累积。

**解决**：让 AI 将分析结果自动写入结构化的 wiki，形成可累积的知识库。每次分析都在为未来的查询积累知识。

## 功能特性

### 核心功能

| 功能 | 说明 |
|------|------|
| **自动初始化** | 扫描代码库，自动生成架构文档、组件说明 |
| **增量更新** | 代码变更后自动更新相关 wiki 页面 |
| **Bug 分析归档** | 分析结果自动归档，支持关联 PMS 系统 |
| **智能查询** | 基于已有知识回答问题，引用源码位置 |
| **健康检查** | 检测 wiki 与代码的同步状态 |

### 技术栈适配

专为 DDE 项目定制，支持：

- **C++ / Qt / QML** — 前端界面开发
- **Go** — 后端服务
- **Shell / Python** — 脚本工具
- **CMake / qmake** — 构建系统

### Wiki 结构

```
wiki/
├── index.md            # 内容目录
├── log.md              # 操作日志
├── overview.md         # 项目总览
├── concepts/           # 架构概念
│   ├── signal-slot-patterns.md
│   ├── qml-architecture.md
│   └── ...
├── entities/           # 组件实体
│   ├── cpp/            # C++ 类
│   ├── qml/            # QML 组件
│   └── go/             # Go 服务
├── bug-analysis/       # Bug 分析记录
└── queries/            # 查询归档
```

## 安装

```bash
# 复制到 Claude Code 全局 skills 目录
cp -r dde-project-wiki ~/.claude/skills/
```

## 使用方法

### 初始化

在项目目录中启动 Claude Code，然后说：

```
初始化项目 wiki
```

或

```
建立知识库
```

Claude 会自动：
1. 扫描代码库结构
2. 读取现有文档和 git 历史
3. 创建 wiki 目录和初始页面
4. 生成架构图和组件说明

### 日常使用

初始化后，wiki 会自动维护：

**询问架构**
```
分析一下 dde-shell 的启动流程
```
→ Claude 会查询 wiki，综合已有知识回答，如有新的有价值分析会归档

**Bug 分析**
```
帮我分析一下这个 bug：登录界面点击后无响应
PMS 编号：PMS-12345
```
→ 分析完成后自动归档到 `bug-analysis/`，并关联 PMS

**代码变更**
```
帮我重构一下登录模块
```
→ 变更完成后自动更新相关 wiki 页面

**健康检查**
```
lint wiki
```
→ 检查 wiki 与代码的同步状态，发现过时或缺失的文档

**导入历史文档**
```
把 ~/notes/ 下的分析文档导入到 wiki
```
→ 扫描目录，智能分类，转换为 wiki 格式

### PMS 关联

Bug 分析支持可选的 PMS 关联：

```
# 提供编号
帮我分析这个 bug，PMS-12345

# 提供链接
帮我分析这个 bug，https://pms.deepin.com/bug/PMS-12345

# 不提供（仅记录分析内容）
帮我分析这个 bug
```

### 导入历史文档

如果你之前已经有手动保存的分析文档，可以导入到 wiki：

```
# 导入整个目录
把 ~/notes/ 下的分析文档导入到 wiki

# 导入单个文件
把这个文件导入 wiki：~/bug-report.md
```

导入时 Claude 会：
1. 读取文档内容
2. 智能判断文档类型（bug 分析/架构概念/组件实体）
3. 转换为 wiki 格式，添加 frontmatter
4. 提取相关源文件和 PMS 编号（如有）
5. 更新交叉引用

**注意**：原文档不会被修改或删除。

## 自动触发

安装后，以下场景会自动触发 skill：

| 触发场景 | 示例 |
|---------|------|
| 修改项目文件 | 修改 .cpp/.h/.qml/.go 等文件后 |
| 询问项目架构 | "这个模块怎么工作的"、"分析框架" |
| 分析 bug | "这个 bug 可能的原因"、"帮我看看这个问题" |
| 代码变更检测 | 新会话开始时检测到代码漂移 |
| 导入历史文档 | "导入这些分析"、"把之前的文档加入 wiki" |
| 手动触发 | "更新 wiki"、"lint wiki" |


## 工作原理

```
┌─────────────────────────────────────────────────────────┐
│                      用户操作                            │
│  (修改代码 / 询问架构 / 分析 bug / 导入文档)            │
└─────────────────────┬───────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────┐
│                   Claude Code Skill                      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────────────┐ │
│  │  INIT   │ │ INGEST  │ │  QUERY  │ │ FILE_ANALYSIS │ │
│  │ 初始化  │ │ 摄入变更 │ │  查询   │ │   分析归档    │ │
│  └─────────┘ └─────────┘ └─────────┘ └───────────────┘ │
│  ┌─────────┐ ┌─────────┐                               │
│  │  LINT   │ │ MIGRATE │                               │
│  │健康检查 │ │导入文档 │                               │
│  └─────────┘ └─────────┘                               │
└─────────────────────┬───────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────┐
│                      Wiki 目录                           │
│  (结构化文档，持续累积，交叉引用)                         │
└─────────────────────────────────────────────────────────┘
```

### 操作说明

| 操作 | 触发条件 | 作用 |
|------|---------|------|
| INIT | 首次初始化 | 扫描代码库，创建初始 wiki |
| INGEST | 检测到代码变更 | 更新相关 wiki 页面 |
| QUERY | 用户询问问题 | 查询 wiki 综合回答 |
| LINT | 用户请求检查 | 健康检查，发现不一致 |
| FILE_ANALYSIS | 完成 bug/功能分析 | 归档分析结果 |
| MIGRATE | 用户导入历史文档 | 智能分类转换并导入 |

## 文件说明

```
dde-project-wiki/
├── SKILL.md                    # Skill 定义（触发条件、操作流程）
├── hooks.json                  # 自动触发钩子配置
├── scripts/
│   └── check-project-drift.sh  # 会话启动时的漂移检测脚本
└── references/
    └── operations.md           # 详细操作手册和模板
```

## 示例

### 生成的实体页面

```yaml
---
title: LoginDialog
type: entity
tags: [cpp, qt, dialog]
created: 2026-04-21
updated: 2026-04-21
related_files:
  - src/widgets/login-dialog.h
  - src/widgets/login-dialog.cpp
inherits: QDialog
signals:
  - loginRequested(QString username)
slots:
  - onLoginButtonClicked()
---

登录对话框组件，负责收集用户凭证并触发认证流程。

## 主要功能

- 用户名/密码输入
- 记住密码选项
- 登录按钮交互

## 信号

### loginRequested(QString username)
用户点击登录按钮时触发，携带用户名。

## 依赖

- [[SessionService]] — 认证服务
- [[AuthController]] — 认证控制器
```

### 生成的 Bug 分析页面

```yaml
---
title: 登录界面点击无响应
type: bug-analysis
tags: [bug, login, freeze]
created: 2026-04-21
status: fixed
related_files:
  - src/widgets/login-dialog.cpp
  - src/core/event-handler.cpp
pms: PMS-12345
pms_url: https://pms.deepin.com/bug/PMS-12345
---

## 问题描述

用户点击登录按钮后界面无响应，持续约 3-5 秒。

## 根因分析

登录按钮的槽函数中执行了同步 DBus 调用，阻塞了 UI 线程。

## 修复方案

将 DBus 调用改为异步模式，使用 QDBusPendingCallWatcher 处理回调。

## 验证方法

1. 启动应用
2. 输入凭证并点击登录
3. 确认界面保持响应
```

## 致谢

- 基于 [Andrej Karpathy 的 LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 原始实现来自 [karpathy-wiki](https://github.com/toolboxdotmd/karpathy-wiki)

## 许可证

MIT License
