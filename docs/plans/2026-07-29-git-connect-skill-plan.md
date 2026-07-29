# git-connect Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a cross-agent compatible Skill that connects a local project to an existing remote Git repository with one command, auto-detecting and fixing missing init/commit/remote states.

**Architecture:** A single Agent Skills standard `SKILL.md` file under `skills/git-connect/` containing frontmatter, state-detection workflow, edge-case handling, and example prompts. The Skill is packaged in a standalone git repository that can be cloned or installed as a pi package.

**Tech Stack:** Markdown, Git CLI, Agent Skills standard.

---

## File Structure

```
~/ai-tasks/git-connect-skill/
├── README.md                          # Project overview and install instructions
├── LICENSE                            # MIT license
├── docs/
│   ├── 2026-07-29-git-connect-skill-design.md
│   └── plans/
│       └── 2026-07-29-git-connect-skill-plan.md
└── skills/
    └── git-connect/
        └── SKILL.md                   # The actual skill consumed by agents
```

---

### Task 1: Create Project Directory Structure

**Files:**
- Create: `~/ai-tasks/git-connect-skill/README.md`
- Create: `~/ai-tasks/git-connect-skill/LICENSE`
- Create: `~/ai-tasks/git-connect-skill/skills/git-connect/SKILL.md`

- [ ] **Step 1: Create directories**

Run:

```bash
mkdir -p ~/ai-tasks/git-connect-skill/skills/git-connect
mkdir -p ~/ai-tasks/git-connect-skill/docs/plans
```

Expected: Directories exist with no errors.

- [ ] **Step 2: Verify structure**

Run:

```bash
find ~/ai-tasks/git-connect-skill -type d | sort
```

Expected output:

```
/Users/sevan/ai-tasks/git-connect-skill
/Users/sevan/ai-tasks/git-connect-skill/docs
/Users/sevan/ai-tasks/git-connect-skill/docs/plans
/Users/sevan/ai-tasks/git-connect-skill/skills
/Users/sevan/ai-tasks/git-connect-skill/skills/git-connect
```

---

### Task 2: Write the Skill File

**Files:**
- Create: `~/ai-tasks/git-connect-skill/skills/git-connect/SKILL.md`

- [ ] **Step 1: Write SKILL.md content**

Create `~/ai-tasks/git-connect-skill/skills/git-connect/SKILL.md` with:

```markdown
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
```

- [ ] **Step 2: Verify SKILL.md exists and has valid frontmatter**

Run:

```bash
head -n 5 ~/ai-tasks/git-connect-skill/skills/git-connect/SKILL.md
```

Expected output starts with:

```yaml
---
name: git-connect
```

---

### Task 3: Write README.md

**Files:**
- Create: `~/ai-tasks/git-connect-skill/README.md`

- [ ] **Step 1: Write README.md content**

Create `~/ai-tasks/git-connect-skill/README.md` with:

```markdown
# git-connect

A cross-agent Skill that connects a local project to an existing remote Git repository in one step.

## What it does

Given a remote repository URL, the Skill automatically:

1. Initializes a git repo if needed (`git init`)
2. Creates an initial commit if there are none
3. Adds or updates the `origin` remote
4. Pushes the current branch to the remote with tracking (`git push -u origin <branch>`)

## Installation

### Manual install

```bash
cp -r skills/git-connect ~/.agents/skills/
```

### Install as a pi package

```bash
pi install git:github.com/HuskkyQ/git-connect
```

## Usage

In any agent that supports the [Agent Skills standard](https://agentskills.io), say:

> "Connect this project to git@github.com:yourname/yourrepo.git"

The agent will detect the current project state and execute the necessary git commands.

## Requirements

- Git installed on the system
- SSH or HTTPS access to the remote repository
- At least one file in the project directory if an initial commit is needed

## License

MIT
```

- [ ] **Step 2: Verify README.md renders correctly**

Run:

```bash
wc -l ~/ai-tasks/git-connect-skill/README.md
```

Expected: file has more than 20 lines.

---

### Task 4: Write LICENSE

**Files:**
- Create: `~/ai-tasks/git-connect-skill/LICENSE`

- [ ] **Step 1: Write MIT LICENSE**

Create `~/ai-tasks/git-connect-skill/LICENSE` with:

```text
MIT License

Copyright (c) 2026 HuskkyQ

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Verify LICENSE file exists**

Run:

```bash
ls -l ~/ai-tasks/git-connect-skill/LICENSE
```

Expected: file exists with non-zero size.

---

### Task 5: Install Skill Locally for Testing

**Files:**
- Create: `~/.agents/skills/git-connect/` (copy)

- [ ] **Step 1: Copy skill to global skills directory**

Run:

```bash
mkdir -p ~/.agents/skills
cp -r ~/ai-tasks/git-connect-skill/skills/git-connect ~/.agents/skills/
```

Expected: No errors.

- [ ] **Step 2: Verify installation path**

Run:

```bash
ls -l ~/.agents/skills/git-connect/SKILL.md
```

Expected: `SKILL.md` exists.

---

### Task 6: Test the Skill Workflow Manually

**Files:**
- Create: temporary test directory

- [ ] **Step 1: Create a test project directory**

Run:

```bash
mkdir -p /tmp/git-connect-test
cd /tmp/git-connect-test
rm -rf .git
```

Expected: directory exists and is not a git repo.

- [ ] **Step 2: Create a file and run the git-connect workflow manually**

Run:

```bash
cd /tmp/git-connect-test
echo "hello world" > README.md
git init
git add .
git commit -m "initial commit"
git remote add origin git@github.com:HuskkyQ/git-connect.git
git push -u origin main
```

Expected: `git init`, `git commit`, and `git remote add` succeed. `git push` may fail if SSH auth is not configured for GitHub — that is acceptable for this manual test; the goal is to verify the command sequence is correct.

- [ ] **Step 3: Verify final git state**

Run:

```bash
cd /tmp/git-connect-test
git remote -v
git branch -vv
```

Expected output contains:

```
origin  git@github.com:HuskkyQ/git-connect.git (fetch)
origin  git@github.com:HuskkyQ/git-connect.git (push)
* main                [origin/main]
```

The exact push result depends on SSH authentication.

- [ ] **Step 4: Clean up test directory**

Run:

```bash
rm -rf /tmp/git-connect-test
```

Expected: directory removed.

---

### Task 7: Initialize Git Repository and Push to Remote

**Files:**
- Modify: `~/ai-tasks/git-connect-skill/.git/` (init)
- Modify: `~/ai-tasks/git-connect-skill/` (commit and push)

- [ ] **Step 1: Initialize git repo in the skill project**

Run:

```bash
cd ~/ai-tasks/git-connect-skill
git init
```

Expected output:

```
Initialized empty Git repository in /Users/sevan/ai-tasks/git-connect-skill/.git/
```

- [ ] **Step 2: Add remote origin**

Run:

```bash
cd ~/ai-tasks/git-connect-skill
git remote add origin git@github.com:HuskkyQ/git-connect.git
```

Expected: No output, exit code 0.

- [ ] **Step 3: Stage and commit all files**

Run:

```bash
cd ~/ai-tasks/git-connect-skill
git add .
git commit -m "feat: add git-connect skill"
```

Expected output shows one commit created with the skill files.

- [ ] **Step 4: Push to remote**

Run:

```bash
cd ~/ai-tasks/git-connect-skill
git push -u origin main
```

Expected: Push succeeds and remote `main` branch is created.

- [ ] **Step 5: Verify remote state**

Run:

```bash
cd ~/ai-tasks/git-connect-skill
git remote -v
git status
```

Expected:

```
origin  git@github.com:HuskkyQ/git-connect.git (fetch)
origin  git@github.com:HuskkyQ/git-connect.git (push)
On branch main
Your branch is up to date with 'origin/main'.
```

---

## Self-Review

**Spec coverage:**
- Auto-detect git repo state: covered in Task 2 SKILL.md workflow steps 1-3.
- Add/update remote origin: covered in Task 2 SKILL.md workflow step 5.
- Push current branch: covered in Task 2 SKILL.md workflow step 6.
- Cross-agent compatibility: covered via Agent Skills standard frontmatter and directory layout.
- Independent project with remote repo: covered in Tasks 3, 4, and 7.

**Placeholder scan:**
- No "TBD", "TODO", or "implement later".
- No vague "add error handling" steps; each command is explicit.
- Remote URL `git@github.com:HuskkyQ/git-connect.git` is provided by the user and used directly.

**Type consistency:**
- Directory names and Skill name match (`git-connect`).
- Branch detection relies on `git branch --show-current` output, not hardcoded values.
