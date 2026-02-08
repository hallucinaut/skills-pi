---
name: git-workflow
description: "Manage git operations, branching strategies, and version control workflows. Use when creating branches, merging changes, resolving conflicts, or implementing git best practices."
---

# Git Workflow Skill

Manage git operations, branching strategies, and version control workflows.

## When to Use

Use this skill when the user wants to:
- Create and manage git branches
- Implement branching strategies (Git Flow, trunk-based)
- Merge and resolve conflicts
- Create pull requests
- Rebase and squash
- Manage git history
- Configure git settings

## Git Workflow Patterns

### Branching Strategies

#### Git Flow
```
main → develop → feature/* → release/* → hotfix/*
```

#### Trunk-Based Development
```
main → PR → merge → deploy
Feature branches are short-lived (1-3 days)
```

#### GitHub Flow
```
main → PR → merge → deploy
Continuous deployment
```

### Common Operations

#### Branch Creation
```bash
# Create feature branch
git checkout -b feature/my-feature

# Create issue branch
git checkout -b issue/123-fix-bug

# Stash changes before branching
git stash push -m "WIP: working on feature"
git checkout -b feature/my-feature
git stash pop
```

#### Merging
```bash
# Merge into branch
git merge --no-ff feature/my-feature

# Squash merge for clean history
git merge --squash -m "Implement feature" feature/my-feature

# Rebase to keep history linear
git rebase main
```

#### Conflict Resolution
```bash
# Resolve merge conflicts
git status  # Identify conflicted files
git add <resolved-files>
git commit -m "Resolve merge conflicts"

# Accept incoming changes
git merge --theirs

# Accept current changes
git merge --ours
```

### Git Best Practices

#### Commits
- **Atomic**: Small, focused commits
- **Descriptive**: Clear commit messages
- **Conventional**: Use Conventional Commits format
- **Squash**: Group related changes

#### Commit Message Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**: feat, fix, docs, style, refactor, test, chore, perf, build, ci, revert

#### Repository Cleanup
```bash
# Clean up branches merged to main
git branch --merged main | xargs -r git branch -d

# Delete remote branches
git push origin --delete <branch-name>

# Remove stale remote tracking branches
git remote prune origin
```

## Configuration

### Git Configuration
```bash
# Set user
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Configure signing
git config --global commit.gpgsign true

# Enable autocorrect
git config --global help.autocorrect 1
```

### Alias Setup
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
```

## Deliverables

- Branch strategy implementation
- Git configuration files
- Commit message conventions
- Workflow documentation
- Git cleanup scripts

## Quality Checklist

- Branches are named consistently
- Commit messages are descriptive
- History is clean and linear
- No merge conflicts in main
- Commit convention is followed
- Branches are deleted after merge
- Documentation is updated
