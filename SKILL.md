---
name: github-https-remote-push
description: Use when a local project needs to commit and push changes to a GitHub HTTPS remote, including routine commits, first-time repository setup, upstream tracking, Windows Git Credential Manager authentication, 首次提交, 日常提交, 推送到 GitHub, 远程仓库, or 凭据管理器.
---

# GitHub HTTPS Remote Push

## Overview

Use this workflow to commit local project changes and push them to a GitHub repository over HTTPS. It covers both existing Git repositories and first-time repository setup.

## 中文说明

当用户要求把本地项目或当前改动提交并推送到 GitHub 远程仓库时，使用这个 skill；它不只用于首次提交，也用于后续每一次常规提交和推送。核心思路是：先判断当前目录是不是 Git 仓库、是否已有远端、当前分支是否跟踪远端；再检查工作区改动和 `.gitignore`；运行项目验证；提交改动；最后通过 GitHub HTTPS 远端推送，并在需要时使用 Git Credential Manager 处理认证。

如果目录还不是 Git 仓库，才执行初始化流程；如果已经是 Git 仓库，就沿用现有分支和远端，不要重新初始化或覆盖历史。如果远端已经有提交，先同步或集成远端历史，避免强推覆盖。完成后必须验证本地分支、远端分支和最新提交是否一致。

在 Windows PowerShell 中，如果 `npm test` 或 `npm run build` 因为 `npm.ps1 cannot be loaded` 被执行策略拦截，不要把它判断为项目失败，改用 `npm.cmd test` 或 `npm.cmd run build` 重新验证。推送时如果出现无法读取 GitHub 用户名或无法交互输入凭据，优先检查 `git credential-manager --version`，然后对当前仓库设置 `git config credential.helper manager`，再重新执行推送。

## Workflow

1. Enter the project directory and inspect repository state:

```powershell
git rev-parse --is-inside-work-tree
git status --short --branch
git remote -v
git branch --show-current
```

If the directory is already a Git repository, continue from its current branch and remotes. If it is not a Git repository, use the first-time setup step below.

2. For an existing repository, inspect tracking and remote state:

```powershell
git status --short --branch
git branch -vv
git fetch --prune
git status --short --branch
```

If the local branch is behind, diverged, or has no upstream, resolve that deliberately before pushing. Do not force-push unless the user explicitly asks and the risk is understood.

3. For first-time setup only, check whether the GitHub remote already has history, then initialize:

```powershell
git ls-remote --heads https://github.com/<owner>/<repo>.git
git init
git branch -M main
git remote add origin https://github.com/<owner>/<repo>.git
```

If `git ls-remote` shows existing remote branches, do not push a new unrelated root commit over them. Fetch, clone, or integrate the remote history first.

4. Review ignored files before staging. For Vite/Node projects, make sure `.gitignore` covers generated dependencies, builds, logs, local env files, and TypeScript build info:

```gitignore
node_modules
dist
.vite
coverage
*.local
*.tsbuildinfo
vite-dev*.log
```

Adapt this list to the project stack. Never commit credentials, local env secrets, dependency folders, build output, logs, or caches.

5. Run project verification before committing. On Windows PowerShell, if `npm` is blocked by the script execution policy, use `npm.cmd`:

```powershell
npm.cmd test
npm.cmd run build
```

Use the repository's actual verification commands if it is not a Node project.

6. Stage, review, and commit:

```powershell
git add .
git status --short --branch
git commit -m "<clear commit message>"
```

If there are no staged changes, do not create an empty commit unless the user explicitly asked for one.

7. Push to GitHub:

```powershell
git push
```

If the branch has no upstream yet, use:

```powershell
git push -u origin <branch-name>
```

For first-time `main` pushes, this is usually:

```powershell
git push -u origin main
```

If pushing over HTTPS fails with a missing username or non-interactive prompt, check and enable Git Credential Manager:

```powershell
git credential-manager --version
git config credential.helper manager
git push
```

Complete any GitHub login or authorization window that Git Credential Manager opens. If the branch still has no upstream, rerun the upstream push form.

8. Verify the result:

```powershell
git status --short --branch
git branch -vv
git log --oneline --decorate -1
git remote -v
git ls-remote --heads origin <branch-name>
```

Only report success when the working tree is clean or expected, the local branch tracks the intended remote branch, the remote branch exists, and the latest local commit is present on the remote.

## Common Pitfalls

- Do not reinitialize an existing Git repository.
- Do not commit `node_modules`, `dist`, logs, local env files, secrets, or build caches.
- Do not force-push into a remote that already has unrelated commits.
- Do not treat PowerShell `npm.ps1 cannot be loaded` as a project test failure; rerun with `npm.cmd`.
- Do not paste GitHub tokens into chat or commit them. Use Git Credential Manager, GitHub CLI, or another approved credential flow.
