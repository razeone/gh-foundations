# Git Medium Level - Concepts and Practical Scenarios

This guide covers intermediate Git concepts with practical examples and real-world scenarios. Perfect for developers who have mastered the basics and want to level up their Git skills.

## Table of Contents

1. [Interactive Rebase and Commit Management](#interactive-rebase-and-commit-management)
2. [Advanced Stash Operations](#advanced-stash-operations)
3. [Branch Management Strategies](#branch-management-strategies)
4. [Conflict Resolution Techniques](#conflict-resolution-techniques)
5. [Git Hooks and Automation](#git-hooks-and-automation)
6. [Working with Multiple Remotes](#working-with-multiple-remotes)
7. [Practical Scenarios](#practical-scenarios)

---

## Interactive Rebase and Commit Management

### Squashing Commits

**Scenario**: You have multiple commits for a single feature and want to clean up the history before merging.

```bash
# View recent commits
git log --oneline -5

# Example output:
# a1b2c3d Add user authentication tests
# e4f5g6h Fix typo in authentication logic
# i7j8k9l Add user authentication feature
# m0n1o2p Update documentation
# q3r4s5t Initial commit

# Squash the last 3 commits into one
git rebase -i HEAD~3

# This opens an editor with:
# pick i7j8k9l Add user authentication feature
# pick e4f5g6h Fix typo in authentication logic
# pick a1b2c3d Add user authentication tests

# Change to:
# pick i7j8k9l Add user authentication feature
# squash e4f5g6h Fix typo in authentication logic
# squash a1b2c3d Add user authentication tests
```

### Editing Commit Messages

```bash
# Edit the last commit message
git commit --amend -m "New commit message"

# Edit multiple commit messages
git rebase -i HEAD~3
# Change 'pick' to 'reword' for commits you want to edit
```

### Splitting a Commit

```bash
# Split the last commit
git reset HEAD~1
git add file1.js
git commit -m "Add file1 functionality"
git add file2.js
git commit -m "Add file2 functionality"
```

---

## Advanced Stash Operations

### Partial Stashing

**Scenario**: You have changes to multiple files but only want to stash some of them.

```bash
# Stash only specific files
git stash push -m "WIP: user service changes" src/userService.js src/userModel.js

# Stash everything except specific files
git stash push -m "All except config" -- . ':!config.json'

# Interactive stashing (choose hunks)
git stash -p
```

### Stash with Untracked Files

```bash
# Stash including untracked files
git stash -u

# Stash including ignored files
git stash -a

# Apply stash and keep it in stash list
git stash apply stash@{1}

# Create a branch from a stash
git stash branch feature-from-stash stash@{0}
```

### Advanced Stash Management

```bash
# Show what's in a specific stash
git stash show -p stash@{1}

# Apply only specific files from a stash
git checkout stash@{0} -- src/utils.js

# Clear all stashes
git stash clear

# Drop a specific stash
git stash drop stash@{2}
```

---

## Branch Management Strategies

### Feature Branch Workflow

**Scenario**: Working on a feature while keeping main branch clean.

```bash
# Start feature development
git checkout main
git pull origin main
git checkout -b feature/user-dashboard

# Work on feature...
git add .
git commit -m "Add dashboard layout"

# Keep feature branch updated with main
git checkout main
git pull origin main
git checkout feature/user-dashboard
git rebase main

# Or merge main into feature
git merge main
```

### Release Branch Strategy

```bash
# Create release branch
git checkout -b release/v1.2.0 develop

# Finalize release
git commit -m "Bump version to 1.2.0"

# Merge to main and tag
git checkout main
git merge release/v1.2.0
git tag v1.2.0

# Merge back to develop
git checkout develop
git merge release/v1.2.0

# Clean up
git branch -d release/v1.2.0
```

### Hotfix Workflow

```bash
# Emergency fix on production
git checkout main
git checkout -b hotfix/critical-security-fix

# Make the fix
git commit -m "Fix security vulnerability in auth"

# Deploy to main
git checkout main
git merge hotfix/critical-security-fix
git tag v1.2.1

# Merge to develop
git checkout develop
git merge hotfix/critical-security-fix

# Clean up
git branch -d hotfix/critical-security-fix
```

---

## Conflict Resolution Techniques

### Understanding Conflict Markers

```bash
# When you encounter a merge conflict:
git merge feature-branch

# Conflict markers in file:
<<<<<<< HEAD
const apiUrl = 'https://api.production.com';
=======
const apiUrl = 'https://api.staging.com';
>>>>>>> feature-branch
```

### Conflict Resolution Strategies

```bash
# Use merge tool
git mergetool

# Keep your version
git checkout --ours conflicted-file.js

# Keep their version
git checkout --theirs conflicted-file.js

# Abort merge
git merge --abort

# Continue after resolving
git add conflicted-file.js
git commit
```

### Three-Way Merge Resolution

```bash
# Show three-way diff
git show :1:file.js  # common ancestor
git show :2:file.js  # our version
git show :3:file.js  # their version

# Resolve and stage
git add file.js
```

---

## Git Hooks and Automation

### Pre-commit Hook Example

```bash
# .git/hooks/pre-commit
#!/bin/sh

# Run linting
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Commit aborted."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi
```

### Pre-push Hook

```bash
# .git/hooks/pre-push
#!/bin/sh

protected_branch='main'
current_branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')

if [ $protected_branch = $current_branch ]; then
    echo "Direct push to main branch is not allowed"
    exit 1
fi
```

### Commit Message Hook

```bash
# .git/hooks/commit-msg
#!/bin/sh

commit_regex='^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,50}'

if ! grep -qE "$commit_regex" "$1"; then
    echo "Invalid commit message format!"
    echo "Format: type(scope): description"
    echo "Example: feat(auth): add user login validation"
    exit 1
fi
```

---

## Working with Multiple Remotes

### Fork Workflow

```bash
# Add upstream remote
git remote add upstream https://github.com/original/repo.git

# Fetch from upstream
git fetch upstream

# Update your main from upstream
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

### Multiple Remote Workflow

```bash
# Add multiple remotes
git remote add production https://github.com/company/prod-repo.git
git remote add staging https://github.com/company/staging-repo.git

# Push to different remotes
git push origin feature-branch
git push staging feature-branch

# Pull from specific remote
git pull upstream main
```

---

## Practical Scenarios

### Scenario 1: Accidentally Committed to Wrong Branch

```bash
# You committed to main instead of feature branch
git log --oneline -3
# a1b2c3d (HEAD -> main) Add new feature
# e4f5g6h Previous commit
# i7j8k9l Even older commit

# Create feature branch at current HEAD
git branch feature/new-feature

# Reset main to previous commit
git reset --hard HEAD~1

# Switch to feature branch
git checkout feature/new-feature
# Your commit is now on the correct branch!
```

### Scenario 2: Recover Lost Commits

```bash
# You accidentally hard reset and lost commits
git reflog
# e4f5g6h HEAD@{0}: reset: moving to HEAD~2
# a1b2c3d HEAD@{1}: commit: Add important feature
# m0n1o2p HEAD@{2}: commit: Fix bug

# Recover the lost commit
git cherry-pick a1b2c3d
```

### Scenario 3: Split Repository History

```bash
# Extract subdirectory into new repo with history
git filter-branch --prune-empty --subdirectory-filter src/ -- --all

# Or using newer git-filter-repo
git filter-repo --path src/
```

### Scenario 4: Synchronize Diverged Branches

```bash
# Branches have diverged
git status
# Your branch and 'origin/main' have diverged

# Option 1: Rebase to linear history
git pull --rebase origin main

# Option 2: Merge commit
git pull origin main

# Option 3: Reset to remote (lose local commits)
git reset --hard origin/main
```

### Scenario 5: Large File Cleanup

```bash
# Remove large files from history
git filter-branch --force --index-filter \
'git rm --cached --ignore-unmatch path/to/large-file.zip' \
--prune-empty --tag-name-filter cat -- --all

# Or using git-filter-repo
git filter-repo --path-glob '*.zip' --invert-paths
```

---

## Tips and Best Practices

### Commit Message Conventions

```bash
# Good commit messages:
git commit -m "feat(auth): add OAuth2 integration"
git commit -m "fix(api): handle null response in user service"
git commit -m "docs: update installation instructions"
git commit -m "refactor(utils): extract common validation logic"

# Conventional Commits format:
# type(scope): description
# 
# body (optional)
# 
# footer (optional)
```

### Branch Naming Conventions

```bash
# Feature branches
feature/user-authentication
feature/payment-integration
feat/shopping-cart

# Bug fixes
bugfix/login-error
fix/memory-leak-in-parser
hotfix/security-vulnerability

# Maintenance
chore/update-dependencies
docs/api-documentation
refactor/user-service
```

### Workflow Optimization

```bash
# Create aliases for common operations
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'

# Useful aliases
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

---

This covers the essential medium-level Git concepts. For advanced topics like submodules, subtrees, and complex merge strategies, see [GIT-ADVANCED.md](./GIT-ADVANCED.md).