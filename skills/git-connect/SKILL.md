---
name: git-connect
description: Connect a local project to an existing remote Git repository. Use when the user provides a repository URL and wants to link the current project to it, handling git init, initial commit, remote setup, and first push automatically.
license: MIT
---

# git-connect

Connect the current local project to an existing remote Git repository with one command.

## When to use

Use this skill when the user says something like:
- "Connect this project to git@github.com:user/repo.git"
- "Push this to the remote at https://github.com/user/repo.git"
- "Set up remote origin for this repo"
- "把当前项目连接到 git@github.com:user/repo.git"

## Workflow

1. **Check git availability**
   ```bash
   git --version
   ```
   If git is not installed, stop and tell the user to install git.

2. **Check if current directory is a git repository**
   ```bash
   git rev-parse --git-dir
   ```
   - If it fails, run `git init`.

3. **Check if there is at least one commit**
   ```bash
   git rev-parse HEAD
   ```
   - If it fails, stage files and create an initial commit:
     ```bash
     git add .
     git commit -m "initial commit"
     ```
   - If `git add .` results in "nothing to commit", stop and tell the user to create at least one file first.

4. **Get current branch name**
   ```bash
   git branch --show-current
   ```
   - Save the output as `<branch>`.
   - If empty (should not happen after a commit), rely on git's default branch name.

5. **Check existing remote "origin"**
   ```bash
   git remote get-url origin
   ```
   - If it fails (no origin), add the remote:
     ```bash
     git remote add origin <repository-url>
     ```
   - If it succeeds and the URL equals `<repository-url>`, skip to step 6.
   - If it succeeds but the URL differs, ask the user:
     > Remote "origin" already points to `<existing-url>`. Do you want to overwrite it with `<repository-url>`?
     - If yes: `git remote set-url origin <repository-url>`
     - If no: stop and do not proceed.

6. **Push to remote**
   ```bash
   git push -u origin <branch>
   ```
   - If it fails, report the exact error and suggest next steps (e.g., check SSH key, check permissions, or pull first if the remote is not empty).

## Output summary

After finishing, report to the user:
- Whether `git init` was run
- Whether an initial commit was created
- The final URL of remote `origin`
- Whether `git push` succeeded
- Any error message and suggested next step if something failed

## Example

User: "Connect this project to git@github.com:HuskkyQ/git-connect.git"

Agent execution:
1. `git rev-parse --git-dir` → fails → `git init`
2. `git rev-parse HEAD` → fails → `git add .` → `git commit -m "initial commit"`
3. `git branch --show-current` → `main`
4. `git remote get-url origin` → fails → `git remote add origin git@github.com:HuskkyQ/git-connect.git`
5. `git push -u origin main`
