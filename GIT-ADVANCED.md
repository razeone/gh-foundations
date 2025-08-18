# Git Advanced - Complex Workflows and Expert Techniques

This guide covers advanced Git concepts, complex workflows, and expert-level techniques for managing large codebases and complex development scenarios.

## Table of Contents

1. [Advanced Merge Strategies](#advanced-merge-strategies)
2. [Submodules and Subtrees](#submodules-and-subtrees)
3. [Git Bisect for Debugging](#git-bisect-for-debugging)
4. [Reflog and Data Recovery](#reflog-and-data-recovery)
5. [Performance Optimization](#performance-optimization)
6. [Advanced Branching Patterns](#advanced-branching-patterns)
7. [Git Internals and Troubleshooting](#git-internals-and-troubleshooting)
8. [Complex Rebase Scenarios](#complex-rebase-scenarios)

---

## Advanced Merge Strategies

### Merge Strategy Options

Git offers several merge strategies for different scenarios:

```bash
# Recursive strategy (default for two branches)
git merge -s recursive feature-branch

# Ours strategy - always choose our version
git merge -s ours feature-branch

# Theirs strategy - always choose their version  
git merge -s recursive -X theirs feature-branch

# Octopus merge (multiple branches)
git merge branch1 branch2 branch3

# Subtree merge
git merge -s subtree external-library
```

### Custom Merge Strategies

```bash
# Merge with custom message
git merge --no-ff -m "Merge feature: user authentication system" feature/auth

# Ignore whitespace differences
git merge -X ignore-space-change feature-branch

# Merge with patience algorithm (better conflict resolution)
git merge -X patience feature-branch

# Merge favoring our side for conflicts
git merge -X ours feature-branch
```

### Advanced Conflict Resolution

```bash
# Rerere (reuse recorded resolution)
git config rerere.enabled true

# When you encounter the same conflict again, Git will auto-resolve
git rerere diff
git rerere status

# Manual three-way merge
git merge-file current.txt base.txt other.txt
```

### Merge vs Rebase Decision Matrix

| Scenario | Recommendation | Command |
|----------|---------------|---------|
| Feature branch to main | Squash merge | `git merge --squash feature` |
| Keeping linear history | Rebase | `git rebase main` |
| Preserving context | Merge commit | `git merge --no-ff feature` |
| Multiple contributors | Merge | `git merge feature` |
| Private feature branch | Rebase | `git rebase -i main` |

---

## Submodules and Subtrees

### Working with Submodules

**Scenario**: Including a library as a submodule in your project.

```bash
# Add a submodule
git submodule add https://github.com/library/awesome-lib.git lib/awesome-lib

# Clone repo with submodules
git clone --recursive https://github.com/yourrepo/project.git

# Update submodules in existing repo
git submodule init
git submodule update

# Update all submodules to latest
git submodule update --remote

# Update specific submodule
git submodule update --remote lib/awesome-lib
```

### Submodule Workflow

```bash
# Working inside a submodule
cd lib/awesome-lib
git checkout main
git pull origin main

# Commit submodule change in parent
cd ../..
git add lib/awesome-lib
git commit -m "Update awesome-lib to latest version"

# Push changes
git push origin main

# Remove a submodule
git submodule deinit lib/awesome-lib
git rm lib/awesome-lib
git commit -m "Remove awesome-lib submodule"
```

### Subtrees as Alternative

```bash
# Add subtree (copies the code into your repo)
git subtree add --prefix=lib/awesome-lib https://github.com/library/awesome-lib.git main --squash

# Update subtree
git subtree pull --prefix=lib/awesome-lib https://github.com/library/awesome-lib.git main --squash

# Push changes back to subtree
git subtree push --prefix=lib/awesome-lib https://github.com/library/awesome-lib.git main
```

### Submodules vs Subtrees Comparison

| Feature | Submodules | Subtrees |
|---------|------------|----------|
| Code inclusion | Reference only | Full copy |
| Repository size | Smaller | Larger |
| Complexity | Higher | Lower |
| Offline work | Requires init | Always available |
| Contribution | Easy | More complex |

---

## Git Bisect for Debugging

### Finding Bugs with Binary Search

**Scenario**: A bug was introduced somewhere in the last 50 commits.

```bash
# Start bisect session
git bisect start

# Mark current commit as bad
git bisect bad

# Mark a known good commit
git bisect good v1.2.0

# Git will checkout a commit in the middle
# Test your application...

# If bug exists in current commit
git bisect bad

# If bug doesn't exist
git bisect good

# Continue until Git finds the problematic commit
# Git will show: "commit abc123 is the first bad commit"

# End bisect session
git bisect reset
```

### Automated Bisect

```bash
# Automate bisect with a test script
git bisect start HEAD v1.2.0
git bisect run ./test-script.sh

# test-script.sh example:
#!/bin/bash
make test
if [ $? -eq 0 ]; then
    exit 0  # good commit
else
    exit 1  # bad commit
fi
```

### Bisect with Skip

```bash
# Skip commits that can't be tested (e.g., won't compile)
git bisect skip

# Skip multiple commits
git bisect skip commit1 commit2 commit3

# Skip a range
git bisect skip commit1..commit2
```

---

## Reflog and Data Recovery

### Understanding Reflog

```bash
# View reflog (local history of HEAD)
git reflog

# View reflog for specific branch
git reflog show feature-branch

# View reflog with dates
git reflog --date=relative

# Example output:
# a1b2c3d HEAD@{0}: commit: Add user service
# e4f5g6h HEAD@{1}: rebase finished: returning to refs/heads/main
# i7j8k9l HEAD@{2}: rebase: Add authentication
```

### Recovery Scenarios

#### Recover Deleted Branch

```bash
# Find the branch in reflog
git reflog
# e4f5g6h HEAD@{3}: checkout: moving from feature-branch to main

# Recreate the branch
git branch feature-branch e4f5g6h

# Or checkout directly
git checkout -b feature-branch e4f5g6h
```

#### Recover from Hard Reset

```bash
# Before reset, you were at commit a1b2c3d
# After hard reset, commits seem lost

# Find lost commits in reflog
git reflog
# a1b2c3d HEAD@{1}: commit: Important feature

# Restore to that commit
git reset --hard a1b2c3d

# Or create a new branch from that commit
git branch recovery-branch a1b2c3d
```

#### Recover Staged Files

```bash
# Find lost staged content
git fsck --lost-found

# Look for dangling blobs
git show <blob-hash>

# Extract content
git show <blob-hash> > recovered-file.txt
```

### Reflog Expiry and Cleanup

```bash
# Configure reflog expiry
git config gc.reflogExpire "90 days"
git config gc.reflogExpireUnreachable "30 days"

# Manually expire reflog
git reflog expire --expire=30.days refs/heads/main

# Clean up unreachable objects
git gc --prune=now
```

---

## Performance Optimization

### Repository Maintenance

```bash
# Garbage collection
git gc --aggressive

# Pack loose objects
git repack -ad

# Verify repository integrity
git fsck --full

# Clean up unnecessary files
git clean -fdx

# Prune remote tracking branches
git remote prune origin
```

### Large Repository Optimization

```bash
# Shallow clone for CI/CD
git clone --depth 1 https://github.com/repo/large-project.git

# Partial clone (Git 2.19+)
git clone --filter=blob:none https://github.com/repo/large-project.git

# Sparse checkout for monorepos
git config core.sparseCheckout true
echo "frontend/*" > .git/info/sparse-checkout
git read-tree -m -u HEAD
```

### Git LFS (Large File Storage)

```bash
# Install Git LFS
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "assets/*"

# Add .gitattributes
git add .gitattributes

# Normal Git workflow
git add large-file.psd
git commit -m "Add design file"
git push origin main

# View LFS files
git lfs ls-files

# Pull LFS files
git lfs pull
```

---

## Advanced Branching Patterns

### Feature Flag Branches

```bash
# Create feature flag branch
git checkout -b feature-flag/new-ui

# Merge with feature flag off by default
git checkout main
git merge --no-ff feature-flag/new-ui

# Enable feature gradually
# Update feature flag configuration
git commit -m "Enable new UI for 10% of users"
```

### Release Train Model

```bash
# Weekly release branches
git checkout develop
git checkout -b release/2024-01-15

# Cherry-pick features for this release
git cherry-pick feature1-commit
git cherry-pick feature2-commit

# Deploy release branch
git tag v2024.01.15
git checkout main
git merge release/2024-01-15
```

### Parallel Development Branches

```bash
# Maintain multiple active versions
git checkout -b support/v1.x main~50
git checkout -b support/v2.x main~20

# Apply fixes to multiple versions
git checkout support/v1.x
git cherry-pick security-fix-commit

git checkout support/v2.x
git cherry-pick security-fix-commit
```

---

## Git Internals and Troubleshooting

### Understanding Git Objects

```bash
# Show object type
git cat-file -t a1b2c3d

# Show object content
git cat-file -p a1b2c3d

# List all objects
git rev-list --objects --all

# Show tree structure
git ls-tree HEAD
git ls-tree -r HEAD  # recursive
```

### Debugging Git Operations

```bash
# Debug git commands
GIT_TRACE=1 git merge feature-branch
GIT_TRACE_PACKET=1 git push origin main
GIT_TRACE_SETUP=1 git status

# Debug pack files
git verify-pack -v .git/objects/pack/pack-*.idx

# Analyze repository size
git count-objects -vH
```

### Fixing Corrupted Repository

```bash
# Check repository integrity
git fsck --full --strict

# Fix loose object corruption
git hash-object -w file.txt

# Rebuild index
rm .git/index
git reset

# Clone from backup if severely corrupted
git clone --mirror backup-repo.git
```

---

## Complex Rebase Scenarios

### Interactive Rebase Mastery

```bash
# Reorder commits
git rebase -i HEAD~5
# Change order of 'pick' lines

# Edit a commit in the middle
git rebase -i HEAD~3
# Change 'pick' to 'edit' for target commit
# Make changes, then:
git add .
git commit --amend
git rebase --continue
```

### Rebase onto Different Base

```bash
# Move feature branch to different base
git rebase --onto main feature-old-base feature-branch

# Example: Move commits from feature-branch that come after feature-old-base
# onto main branch
```

### Resolving Complex Rebase Conflicts

```bash
# Start rebase
git rebase main

# For each conflict:
# 1. Resolve conflicts in files
# 2. Stage resolved files
git add resolved-file.js

# Continue rebase
git rebase --continue

# Skip problematic commit
git rebase --skip

# Abort and start over
git rebase --abort
```

### Rebase vs Merge in Team Workflows

| Situation | Use Rebase | Use Merge |
|-----------|------------|-----------|
| Feature branch updates | ✓ | ✗ |
| Shared branch updates | ✗ | ✓ |
| Public commits | ✗ | ✓ |
| Clean linear history | ✓ | ✗ |
| Preserve merge context | ✗ | ✓ |

---

## Advanced Git Workflows

### Continuous Integration Patterns

```bash
# Pre-receive hook for CI
#!/bin/sh
# .git/hooks/pre-receive

while read oldrev newrev refname; do
    if [ "$refname" = "refs/heads/main" ]; then
        # Trigger CI pipeline
        curl -X POST https://ci.example.com/trigger \
             -d "commit=$newrev"
    fi
done
```

### Multi-Repository Coordination

```bash
# Git meta repository pattern
# .gitmodules
[submodule "service-a"]
    path = services/service-a
    url = https://github.com/company/service-a.git
[submodule "service-b"]
    path = services/service-b
    url = https://github.com/company/service-b.git

# Update all services
git submodule foreach 'git pull origin main'
```

### Advanced Tagging Strategies

```bash
# Semantic versioning tags
git tag -a v1.2.3 -m "Release version 1.2.3"

# Pre-release tags
git tag -a v1.3.0-rc.1 -m "Release candidate 1"

# Build metadata tags
git tag -a v1.2.3+build.123 -m "Build 123"

# Signed tags for security
git tag -s v1.2.3 -m "Signed release"

# Verify signed tags
git tag -v v1.2.3
```

---

## Tips for Advanced Git Usage

### Configuration for Power Users

```bash
# Enhanced diff display
git config --global diff.algorithm patience
git config --global diff.compactionHeuristic true

# Better merge conflict resolution
git config --global merge.conflictStyle diff3

# Automatic rebase for pulls
git config --global pull.rebase true

# Push current branch by default
git config --global push.default current

# Enhanced logging
git config --global alias.tree "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all"
```

### Scripting and Automation

```bash
# Get current branch name
current_branch=$(git symbolic-ref --short HEAD)

# Check if working directory is clean
if [ -z "$(git status --porcelain)" ]; then
    echo "Working directory clean"
else
    echo "Uncommitted changes"
fi

# Loop through all branches
for branch in $(git branch -r | grep -v HEAD); do
    echo "Processing $branch"
    git checkout $branch
    # Perform operations...
done
```

---

This guide covers advanced Git concepts for expert-level usage. For practical real-world scenarios and troubleshooting, see [GIT-SCENARIOS.md](./GIT-SCENARIOS.md).