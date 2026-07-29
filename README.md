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
