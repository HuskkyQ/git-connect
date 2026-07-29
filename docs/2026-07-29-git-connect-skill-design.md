# git-connect Skill 设计文档

## 目标

创建一个跨 agent 通用的 Skill，让用户只需提供远程仓库地址，就能一键将本地项目连接到该远程仓库。

Skill 要能自动判断本地项目的 git 状态，并补齐缺失步骤（init、commit、remote、push）。

## 适用范围

- **Agent**：pi、Claude Code、Codex 等支持 [Agent Skills 标准](https://agentskills.io) 的 agent
- **运行时安装位置**：`~/.agents/skills/git-connect/`（全局共享，跨 agent 可见）
- **本 Skill 源码仓库**：`~/ai-tasks/git-connect-skill/`，可推送到 GitHub/GitLab 等远程平台供分享

## 设计原则

1. **纯 Skill 实现**：不依赖特定 agent 的 Extension API，只使用标准 Markdown + bash 工具
2. **自动状态检测**：不假设项目当前状态，由 Skill 指导 agent 逐步检查并执行
3. **安全优先**：覆盖已有 remote 前先询问用户；不自动强制 push
4. **跨 agent 兼容**：严格遵循 Agent Skills 标准，目录名与 Skill 名一致

## Skill 结构

```
git-connect/
└── SKILL.md          # 必需：包含 frontmatter + 执行流程
```

### Frontmatter

```yaml
---
name: git-connect
description: Connect a local project to an existing remote Git repository. Use when the user provides a repository URL and wants to link the current project to it, handling git init, initial commit, remote setup, and first push automatically.
license: MIT
---
```

## 执行流程

```text
用户给出远程仓库地址
    │
    ▼
检查 git 是否安装
    └─ 否 → 报错，提示安装 git
    │
    ▼
检查当前目录是否在 git 仓库内（git rev-parse --git-dir）
    ├─ 否 → git init
    │
    ▼
检查是否有 commit（git rev-parse HEAD）
    ├─ 否 → git add . → git commit -m "initial commit"
    │       └─ 无文件可提交 → 提示用户先创建文件
    │
    ▼
检查当前分支名（git branch --show-current）
    └─ 为空 → 由 git 使用默认分支名（通常是 main），无需强制创建
    │
    ▼
检查 remote "origin" 是否存在（git remote get-url origin）
    ├─ 不存在 → git remote add origin <URL>
    ├─ 存在且 URL 相同 → 跳过添加，直接 push
    └─ 存在但 URL 不同 → 询问用户是否覆盖
        ├─ 是 → git remote set-url origin <URL>
        └─ 否 → 终止
    │
    ▼
执行 git push -u origin <当前分支>
    └─ 失败 → 返回错误信息，不继续
```

## 边界情况处理

| 场景 | 处理方式 |
|------|----------|
| 未安装 git | 立即报错，提示用户安装 |
| 项目目录为空，无可提交文件 | 提示用户先创建至少一个文件 |
| `origin` 已存在且 URL 相同 | 复用，不重复添加 |
| `origin` 已存在但 URL 不同 | 询问用户是否覆盖 |
| 分支名不是 `main` | 自动检测当前分支名并使用；未创建分支时依赖 git 默认配置 |
| push 因认证/权限失败 | 返回原始错误，由用户处理 |
| 远程仓库不是空仓库 | push 失败时提示用户可能需要先 pull/fetch |

## 输出规范

Skill 执行完成后，agent 应向用户汇报：

1. 本地项目是否已初始化为 git 仓库
2. 是否创建了初始 commit
3. `origin` 的 URL
4. push 是否成功
5. 若失败，给出错误信息和下一步建议

## 安装方式

### 方式一：手动复制到全局 Skill 目录

```bash
cp -r skills/git-connect ~/.agents/skills/
```

### 方式二：作为 pi package 安装

本 Skill 可打包为 pi package：

```json
{
  "name": "git-connect-skill",
  "keywords": ["pi-package"],
  "pi": {
    "skills": ["./skills"]
  }
}
```

```bash
pi install git:github.com/<user>/git-connect-skill
```

## 源码仓库管理

Skill 本身维护在 `~/ai-tasks/git-connect-skill/` 这个独立 git 仓库中：

```
git-connect-skill/
├── README.md
├── LICENSE
├── docs/
│   └── 2026-07-29-git-connect-skill-design.md
└── skills/
    └── git-connect/
        └── SKILL.md
```

推送流程：

```bash
cd ~/ai-tasks/git-connect-skill
git init
git remote add origin <远程仓库地址>
git add .
git commit -m "initial commit"
git push -u origin main
```

## 成功标准

1. 在任意支持 Agent Skills 的 agent 中，说出"把当前项目连接到 git@github.com:user/repo.git"后，agent 能自动完成整套流程
2. 对三种初始状态（未 init、已 init 无 commit、已 init 有 commit 无 remote）都能正确处理
3. 不覆盖用户已配置的 remote，除非用户明确同意
4. Skill 自身源码可独立推送到远程仓库

## 后续可扩展

- 支持自动检测 GitHub/GitLab/Gitee 等平台并给出对应创建仓库指引
- 支持 `--dry-run` 模式，只输出将要执行的命令
- 支持自定义初始 commit 信息
