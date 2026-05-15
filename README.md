# GitHub HTTPS Remote Push Skill

This repository contains a Codex skill for committing local project changes and pushing them to a GitHub HTTPS remote.

## Purpose

Use this skill when a local project needs to be committed and pushed to GitHub, including:

- routine commits and pushes
- first-time Git repository setup
- upstream tracking setup
- Windows Git Credential Manager authentication
- Chinese requests such as "提交到 GitHub", "推送远程仓库", "首次提交", or "日常提交"

## Files

- `SKILL.md`: the main skill instructions
- `agents/openai.yaml`: Codex UI metadata for the skill

## 中文说明

这个仓库保存的是一个 Codex skill，用于把本地项目改动提交并推送到 GitHub HTTPS 远程仓库。

它不只适用于首次提交，也适用于后续每一次常规提交和推送。流程会先检查本地 Git 状态、远端地址、分支跟踪关系和工作区改动，再运行项目验证、提交改动并推送到 GitHub。如果 Windows PowerShell 拦截 `npm.ps1`，会改用 `npm.cmd`；如果 HTTPS 推送需要认证，会使用 Git Credential Manager。

## Usage

Place this folder in a Codex skills directory, then ask Codex to commit and push a local project to a GitHub HTTPS repository. Codex should load `SKILL.md` when the request involves GitHub remote pushes.
