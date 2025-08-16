# Git Merge Strategies - Complete Guide

This comprehensive guide covers all Git merge strategies, conflict resolution techniques, and best practices for different scenarios.

## Table of Contents

1. [Understanding Merge Strategies](#understanding-merge-strategies)
2. [Fast-Forward vs Non-Fast-Forward Merges](#fast-forward-vs-non-fast-forward-merges)
3. [Recursive Merge Strategy](#recursive-merge-strategy)
4. [Octopus Merge Strategy](#octopus-merge-strategy)
5. [Ours and Theirs Strategies](#ours-and-theirs-strategies)
6. [Subtree Merge Strategy](#subtree-merge-strategy)
7. [Conflict Resolution Patterns](#conflict-resolution-patterns)
8. [Merge vs Rebase Decision Guide](#merge-vs-rebase-decision-guide)
9. [Practical Examples](#practical-examples)

---

## Understanding Merge Strategies

Git merge strategies determine how Git combines changes from different branches. Each strategy is designed for specific scenarios and has different trade-offs.

### Available Merge Strategies

```bash
# List all available merge strategies
git merge --help | grep -A 20 "MERGE STRATEGIES"

# Common strategies:
# - recursive (default for two branches)
# - octopus (default for multiple branches)
# - ours
# - subtree
```

### Strategy Selection

```bash
# Explicitly specify a strategy
git merge -s <strategy> <branch>

# With strategy options
git merge -s recursive -X <option> <branch>
```

---

## Fast-Forward vs Non-Fast-Forward Merges

### Fast-Forward Merge

**When**: Target branch hasn't diverged from source branch.

```bash
# Initial state:
# main:     A---B---C
# feature:            D---E

# Fast-forward merge
git checkout main
git merge feature

# Result:
# main:     A---B---C---D---E
# feature:                  ^
```

```bash
# Force fast-forward (fail if not possible)
git merge --ff-only feature

# Check if fast-forward is possible
git merge-base --is-ancestor main feature
echo $?  # 0 if fast-forward possible
```

### Non-Fast-Forward Merge

**When**: Branches have diverged.

```bash
# Initial state:
# main:     A---B---C---F
# feature:      \---D---E

# Non-fast-forward merge
git checkout main
git merge feature

# Result:
# main:     A---B---C---F---M
#               \       /
# feature:       \---D---E
```

```bash
# Force merge commit (even if fast-forward possible)
git merge --no-ff feature

# This is useful for preserving branch context
```

### Choosing Fast-Forward Behavior

```bash
# Configure default behavior
git config --global merge.ff false    # Always create merge commit
git config --global merge.ff only     # Only allow fast-forward
git config --global merge.ff true     # Default behavior
```

---

## Recursive Merge Strategy

The recursive strategy is Git's default for two-branch merges. It's sophisticated and handles most scenarios well.

### Basic Recursive Merge

```bash
# Default merge (uses recursive strategy)
git merge feature-branch

# Explicitly specify recursive
git merge -s recursive feature-branch
```

### Recursive Strategy Options

```bash
# Favor our side during conflicts
git merge -s recursive -X ours feature-branch

# Favor their side during conflicts
git merge -s recursive -X theirs feature-branch

# Ignore whitespace changes
git merge -s recursive -X ignore-space-change feature-branch

# Ignore all whitespace
git merge -s recursive -X ignore-all-space feature-branch

# Use patience diff algorithm
git merge -s recursive -X patience feature-branch
```

### Practical Example: Handling Conflicting Refactoring

**Scenario**: Both branches refactored the same function differently.

```javascript
// Original (common ancestor):
function calculatePrice(item) {
    return item.price * item.quantity;
}

// Main branch:
function calculatePrice(item) {
    const price = item.price || 0;
    const quantity = item.quantity || 1;
    return price * quantity;
}

// Feature branch:
function calculateTotalPrice(item) {
    return item.price * item.quantity * (1 + item.tax);
}
```

```bash
# Merge with conflict resolution
git merge feature-branch

# Manual resolution required:
# <<<<<<< HEAD
# function calculatePrice(item) {
#     const price = item.price || 0;
#     const quantity = item.quantity || 1;
#     return price * quantity;
# }
# =======
# function calculateTotalPrice(item) {
#     return item.price * item.quantity * (1 + item.tax);
# }
# >>>>>>> feature-branch

# Resolved version:
function calculateTotalPrice(item) {
    const price = item.price || 0;
    const quantity = item.quantity || 1;
    const tax = item.tax || 0;
    return price * quantity * (1 + tax);
}

git add src/pricing.js
git commit -m "Merge: combine price validation and tax calculation"
```

---

## Octopus Merge Strategy

Used for merging multiple branches simultaneously. Default when merging more than two branches.

### Basic Octopus Merge

```bash
# Merge multiple feature branches
git checkout main
git merge feature-1 feature-2 feature-3

# Explicitly use octopus strategy
git merge -s octopus feature-1 feature-2 feature-3
```

### Octopus Limitations

- **No conflict resolution**: Octopus aborts if any conflicts occur
- **Clean merges only**: All changes must be in different files or non-overlapping areas

### Practical Example: Release Branch

**Scenario**: Merging multiple completed features for a release.

```bash
# Features developed in parallel:
# feature-1: Authentication system
# feature-2: User dashboard  
# feature-3: Email notifications

# Verify no conflicts between features
git merge-tree $(git merge-base main feature-1) feature-1 feature-2

# Perform octopus merge
git checkout main
git merge feature-1 feature-2 feature-3

# Result: Single merge commit with multiple parents
git log --graph --oneline -5
# *   a1b2c3d Merge branches 'feature-1', 'feature-2' and 'feature-3'
# |\|\|
# | | * e4f5g6h Add email notifications
# | * | i7j8k9l Add user dashboard
# * | | m0n1o2p Add authentication system
# |/ /
# * p3q4r5s Latest main commit
```

---

## Ours and Theirs Strategies

### Ours Strategy

Always keeps the current branch's version, ignoring the other branch entirely.

```bash
# Use 'ours' strategy - keeps only our changes
git merge -s ours feature-branch
```

**Use cases**:
- Abandoning a feature branch but keeping merge history
- Marking branches as merged without taking their changes

```bash
# Example: Mark experimental branch as merged
git checkout main
git merge -s ours experimental-feature
# Creates merge commit but ignores all changes from experimental-feature
```

### Theirs Option (with Recursive)

Note: There's no "theirs" strategy, but you can use the theirs option with recursive:

```bash
# Favor their changes in conflicts
git merge -s recursive -X theirs feature-branch

# This still does a real merge but prefers their side for conflicts
```

### Practical Example: Config File Conflicts

**Scenario**: Merging a feature that changed configuration files.

```bash
# Configuration conflict in database.yml
# <<<<<<< HEAD
# database:
#   host: localhost
#   port: 5432
# =======
# database:
#   host: prod-server
#   port: 3306
# >>>>>>> feature-branch

# Keep production config (theirs)
git merge -s recursive -X theirs feature-branch

# Or resolve manually if you need both environments
```

---

## Subtree Merge Strategy

Used for merging a project as a subdirectory of another project.

### Basic Subtree Merge

```bash
# Add external project as remote
git remote add library-project https://github.com/example/library.git
git fetch library-project

# Merge as subtree
git merge -s subtree --allow-unrelated-histories library-project/main
```

### Subtree with Prefix

```bash
# Merge into specific subdirectory
git read-tree --prefix=vendor/library/ -u library-project/main
git commit -m "Add library as subtree"

# Update subtree
git pull -s subtree library-project main
```

### Practical Example: Vendor Libraries

**Scenario**: Including a third-party library in your project.

```bash
# Project structure before:
# myproject/
# ├── src/
# ├── tests/
# └── README.md

# Add library as subtree
git remote add utils-lib https://github.com/company/utils.git
git subtree add --prefix=lib/utils utils-lib main --squash

# Project structure after:
# myproject/
# ├── src/
# ├── tests/
# ├── lib/
# │   └── utils/          # <- Subtree content
# │       ├── index.js
# │       └── package.json
# └── README.md

# Update subtree later
git subtree pull --prefix=lib/utils utils-lib main --squash
```

---

## Conflict Resolution Patterns

### Understanding Conflict Markers

```bash
# Three-way conflict markers
<<<<<<< HEAD (current change)
const apiUrl = 'https://api.production.com';
||||||| merged common ancestors
const apiUrl = 'https://api.development.com';
=======
const apiUrl = 'https://api.staging.com';
>>>>>>> feature-branch (incoming change)
```

### Resolution Strategies by File Type

#### Code Files

```bash
# View three-way diff
git show :1:file.js  # Common ancestor
git show :2:file.js  # Our version (HEAD)
git show :3:file.js  # Their version (feature-branch)

# Use merge tool
git mergetool file.js

# Manual resolution workflow:
# 1. Edit file to resolve conflicts
# 2. Remove conflict markers
# 3. Test the resolution
# 4. Stage and commit
git add file.js
git commit
```

#### Configuration Files

```bash
# For config files, often want to keep both settings
# Original conflict:
# <<<<<<< HEAD
# timeout: 30
# =======
# retries: 3
# >>>>>>> feature-branch

# Resolution - keep both:
timeout: 30
retries: 3
```

#### Documentation Files

```bash
# For README.md conflicts, merge both sets of documentation
# Use recursive merge with patience for better results
git merge -s recursive -X patience feature-branch
```

### Advanced Conflict Resolution

#### Custom Merge Drivers

```bash
# .gitattributes
*.json merge=json-merge

# Configure custom merge driver
git config merge.json-merge.driver './merge-json.sh %O %A %B %L'
```

#### Rerere (Reuse Recorded Resolution)

```bash
# Enable rerere
git config rerere.enabled true

# When same conflict occurs again, Git auto-resolves
git config rerere.autoupdate true

# View recorded resolutions
git rerere diff
git rerere status
```

---

## Merge vs Rebase Decision Guide

### Decision Matrix

| Scenario | Recommendation | Reasoning |
|----------|---------------|-----------|
| Feature branch to main | **Squash merge** | Clean history, atomic feature |
| Hotfix to main | **Fast-forward merge** | Preserve urgency context |
| Release branch to main | **Merge commit** | Preserve release context |
| Personal feature work | **Rebase** | Clean up before sharing |
| Shared branch updates | **Merge** | Don't rewrite public history |
| Code review process | **Squash merge** | One commit per feature |

### Workflow-Specific Strategies

#### GitHub Flow

```bash
# Feature development
git checkout -b feature/new-login
# ... work and commit ...

# Before creating PR, clean up history
git rebase -i main

# After PR approval, use squash merge in GitHub UI
# Or locally:
git checkout main
git merge --squash feature/new-login
git commit -m "feat: implement new login system"
```

#### Git Flow

```bash
# Feature to develop
git checkout develop
git merge --no-ff feature/new-feature

# Release to main
git checkout main
git merge --no-ff release/1.2.0
git tag v1.2.0

# Hotfix to main and develop
git checkout main
git merge --no-ff hotfix/security-fix
git checkout develop
git merge --no-ff hotfix/security-fix
```

---

## Practical Examples

### Example 1: Large Team Merge Conflicts

**Scenario**: Multiple developers working on the same large file.

```bash
# Use recursive merge with rename detection
git merge -s recursive -X find-renames=40 feature-branch

# If conflicts are too complex, consider breaking them down:
# 1. Create intermediate branch
git checkout -b merge-prep feature-branch
git rebase main  # Resolve conflicts incrementally

# 2. Then merge the cleaned branch
git checkout main
git merge merge-prep
```

### Example 2: Vendor Library Integration

**Scenario**: Integrating an external library with local modifications.

```bash
# Option 1: Subtree merge
git subtree add --prefix=vendor/library external-lib main

# Option 2: Modified ours strategy
git merge -s ours external-lib  # Initial merge
# Then manually apply needed changes from external-lib

# Option 3: Cherry-pick specific commits
git cherry-pick external-lib/commit1
git cherry-pick external-lib/commit2
```

### Example 3: Database Migration Conflicts

**Scenario**: Two branches create conflicting database migrations.

```bash
# Migration files conflict:
# 001_add_users_table.sql (main)
# 001_add_products_table.sql (feature)

# Resolution strategy:
# 1. Rename one migration
mv 001_add_products_table.sql 002_add_products_table.sql

# 2. Update migration dependencies
# 3. Merge without conflicts
git add .
git commit -m "Resolve migration numbering conflict"
```

### Example 4: Configuration Environment Conflicts

**Scenario**: Different environments need different configurations.

```bash
# Use conditional merge for environment configs
if [ "$ENVIRONMENT" = "production" ]; then
    git merge -s recursive -X ours feature-branch
else
    git merge -s recursive -X theirs feature-branch
fi

# Or use merge drivers for automatic resolution
# .gitattributes:
# config/*.yml merge=env-config

# Custom merge driver script
git config merge.env-config.driver './resolve-env-config.sh %A %B'
```

---

## Best Practices Summary

### When to Use Each Strategy

1. **Fast-forward merge**: Simple feature branches, hotfixes
2. **Recursive merge**: Most two-branch merges, handles renames well
3. **Octopus merge**: Multiple simple features, release preparation
4. **Ours strategy**: Abandon branches, ignore external changes
5. **Subtree merge**: Vendor libraries, external project integration

### Merge Commit Messages

```bash
# Good merge commit messages:
git merge --no-ff -m "feat: merge user authentication system

- Add login/logout functionality
- Implement JWT token validation  
- Add password reset flow
- Include comprehensive tests

Closes #123" feature/auth

# Conventional commits for merges:
git merge --no-ff -m "feat(auth): implement OAuth2 integration" feature/oauth
```

### Pre-merge Checklist

1. **Verify branch is up to date**: `git pull origin main`
2. **Run tests on feature branch**: `npm test` or equivalent
3. **Check for conflicts**: `git merge-tree $(git merge-base main feature) main feature`
4. **Review changes**: `git diff main..feature`
5. **Choose appropriate merge strategy**
6. **Write descriptive merge commit message**

---

This comprehensive guide should help you choose the right merge strategy for any situation. For more complex scenarios involving rebasing and history rewriting, see [GIT-ADVANCED.md](./GIT-ADVANCED.md).