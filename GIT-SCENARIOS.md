# Git Practical Scenarios - Real-World Problem Solving

This guide provides practical, real-world scenarios with step-by-step solutions. Each scenario includes the problem context, multiple solution approaches, and complete code examples.

## Table of Contents

1. [Emergency Hotfixes](#emergency-hotfixes)
2. [Team Collaboration Issues](#team-collaboration-issues)
3. [Release Management](#release-management)
4. [Code Review Workflows](#code-review-workflows)
5. [Data Recovery Scenarios](#data-recovery-scenarios)
6. [Large Codebase Management](#large-codebase-management)
7. [Integration Challenges](#integration-challenges)
8. [Performance and Optimization](#performance-and-optimization)

---

## Emergency Hotfixes

### Scenario 1: Critical Production Bug During Release

**Problem**: Production is down due to a bug. A new release is in progress, but you need to fix the issue without deploying unfinished features.

**Context**:
- Production: `v1.2.3` (stable)
- Main branch: Contains v1.3.0 features (unreleased)
- Bug: Security vulnerability in authentication

**Solution**:

```bash
# 1. Create hotfix branch from production tag
git checkout v1.2.3
git checkout -b hotfix/security-auth-fix

# 2. Implement the fix
echo "// Security fix for authentication vulnerability
function validateToken(token) {
    if (!token || typeof token !== 'string') {
        throw new Error('Invalid token format');
    }
    
    // Additional validation
    if (token.length < 10) {
        throw new Error('Token too short');
    }
    
    return jwt.verify(token, process.env.JWT_SECRET);
}" > src/auth/tokenValidator.js

git add src/auth/tokenValidator.js
git commit -m "fix(security): add token validation to prevent injection

- Validate token format and length
- Prevent null/undefined token processing
- Add proper error handling

Fixes CVE-2024-001234"

# 3. Test the fix
npm test -- --grep "token validation"

# 4. Create new patch version
git tag v1.2.4
git push origin hotfix/security-auth-fix
git push origin v1.2.4

# 5. Deploy to production
# (Your deployment process here)

# 6. Merge back to main
git checkout main
git merge hotfix/security-auth-fix

# 7. Clean up
git branch -d hotfix/security-auth-fix
git push origin --delete hotfix/security-auth-fix
```

### Scenario 2: Rollback Strategy

**Problem**: New deployment broke production, need to rollback while preserving recent data changes.

```bash
# Option A: Revert the problematic commit
git log --oneline -5
# a1b2c3d (HEAD -> main) Deploy v1.3.0
# e4f5g6h Add user dashboard
# i7j8k9l Fix authentication bug
# m0n1o2p Update dependencies
# q3r4s5t Previous stable version

# Create revert commit
git revert a1b2c3d -m "Revert v1.3.0 deployment

Production issues identified:
- Database connection errors
- Memory leaks in user service
- Performance degradation

Rolling back to investigate offline."

git push origin main

# Option B: Reset to previous version (more drastic)
git reset --hard q3r4s5t
git push --force-with-lease origin main

# Option C: Create fix-forward branch
git checkout -b fix/production-issues
# Fix issues here...
git checkout main
git merge fix/production-issues
```

---

## Team Collaboration Issues

### Scenario 3: Merge Conflict in Team Environment

**Problem**: Multiple developers modified the same file, creating complex conflicts during merge.

**Context**:
- Developer A: Added new API endpoints
- Developer B: Refactored existing endpoints
- File: `src/api/routes.js`

**Original file**:
```javascript
// src/api/routes.js
const express = require('express');
const router = express.Router();

router.get('/users', getUsers);
router.post('/users', createUser);

module.exports = router;
```

**Developer A's changes** (feature/api-expansion):
```javascript
const express = require('express');
const router = express.Router();

router.get('/users', getUsers);
router.post('/users', createUser);
router.get('/users/:id', getUserById);
router.put('/users/:id', updateUser);
router.delete('/users/:id', deleteUser);

module.exports = router;
```

**Developer B's changes** (feature/api-refactor):
```javascript
const express = require('express');
const { userController } = require('../controllers');
const router = express.Router();

router.get('/users', userController.getAll);
router.post('/users', userController.create);

module.exports = router;
```

**Resolution Steps**:

```bash
# 1. Attempt merge
git checkout main
git merge feature/api-refactor  # Merges cleanly
git merge feature/api-expansion  # Conflicts!

# 2. Analyze the conflict
git status
# Unmerged paths:
#   both modified:   src/api/routes.js

# 3. View the conflict
cat src/api/routes.js
# <<<<<<< HEAD
# const express = require('express');
# const { userController } = require('../controllers');
# const router = express.Router();
# 
# router.get('/users', userController.getAll);
# router.post('/users', userController.create);
# =======
# const express = require('express');
# const router = express.Router();
# 
# router.get('/users', getUsers);
# router.post('/users', createUser);
# router.get('/users/:id', getUserById);
# router.put('/users/:id', updateUser);
# router.delete('/users/:id', deleteUser);
# >>>>>>> feature/api-expansion

# 4. Resolve by combining both changes
cat > src/api/routes.js << 'EOF'
const express = require('express');
const { userController } = require('../controllers');
const router = express.Router();

// User CRUD operations
router.get('/users', userController.getAll);
router.post('/users', userController.create);
router.get('/users/:id', userController.getById);
router.put('/users/:id', userController.update);
router.delete('/users/:id', userController.delete);

module.exports = router;
EOF

# 5. Update controller to match new methods
cat > src/controllers/userController.js << 'EOF'
const userService = require('../services/userService');

module.exports = {
    getAll: async (req, res) => {
        const users = await userService.getAllUsers();
        res.json(users);
    },
    
    create: async (req, res) => {
        const user = await userService.createUser(req.body);
        res.status(201).json(user);
    },
    
    getById: async (req, res) => {
        const user = await userService.getUserById(req.params.id);
        res.json(user);
    },
    
    update: async (req, res) => {
        const user = await userService.updateUser(req.params.id, req.body);
        res.json(user);
    },
    
    delete: async (req, res) => {
        await userService.deleteUser(req.params.id);
        res.status(204).send();
    }
};
EOF

# 6. Test the resolution
npm test

# 7. Complete the merge
git add src/api/routes.js src/controllers/userController.js
git commit -m "merge: combine API expansion with refactoring

- Integrate new CRUD endpoints from feature/api-expansion
- Apply controller pattern from feature/api-refactor
- Update method names to match controller conventions

Resolves conflicts between API expansion and refactoring work"
```

### Scenario 4: Sync Fork with Upstream

**Problem**: Your fork is behind the upstream repository, and you need to sync while preserving your work.

```bash
# 1. Add upstream remote (if not already added)
git remote add upstream https://github.com/original/repo.git
git remote -v
# origin    https://github.com/yourusername/repo.git (fetch)
# origin    https://github.com/yourusername/repo.git (push)
# upstream  https://github.com/original/repo.git (fetch)
# upstream  https://github.com/original/repo.git (push)

# 2. Fetch upstream changes
git fetch upstream

# 3. Check current status
git status
git log --oneline --graph origin/main upstream/main

# 4. Sync your main branch
git checkout main
git merge upstream/main

# If there are conflicts, resolve them

# 5. Update your feature branches
git checkout feature/my-awesome-feature
git rebase main  # or git merge main

# 6. Push updates to your fork
git push origin main
git push --force-with-lease origin feature/my-awesome-feature
```

---

## Release Management

### Scenario 5: Preparing a Release with Multiple Features

**Problem**: Multiple teams have completed features. You need to create a stable release while some features might not be ready.

**Context**:
- Features completed: A, B, C
- Features in progress: D, E
- Target: v2.0.0 release

```bash
# 1. Create release branch from main
git checkout main
git pull origin main
git checkout -b release/v2.0.0

# 2. Cherry-pick only completed features
# Identify feature commits
git log --oneline --grep="feat" --since="2024-01-01"

# Cherry-pick stable features
git cherry-pick a1b2c3d  # Feature A
git cherry-pick e4f5g6h  # Feature B  
git cherry-pick i7j8k9l  # Feature C

# 3. Update version and changelog
echo "2.0.0" > VERSION

cat > CHANGELOG.md << 'EOF'
# Changelog

## [2.0.0] - 2024-01-15

### Added
- Feature A: User authentication system
- Feature B: Advanced search functionality  
- Feature C: Real-time notifications

### Fixed
- Memory leak in background processing
- Performance issues with large datasets

### Security
- Updated dependencies to patch vulnerabilities
- Enhanced input validation
EOF

git add VERSION CHANGELOG.md
git commit -m "chore: prepare release v2.0.0"

# 4. Run comprehensive tests
npm run test:integration
npm run test:e2e
npm run security:audit

# 5. Create release candidate
git tag v2.0.0-rc.1
git push origin release/v2.0.0
git push origin v2.0.0-rc.1

# 6. After testing, finalize release
git tag v2.0.0
git push origin v2.0.0

# 7. Merge to main and back to develop
git checkout main
git merge release/v2.0.0
git checkout develop
git merge release/v2.0.0

# 8. Clean up
git branch -d release/v2.0.0
```

### Scenario 6: Semantic Versioning Automation

**Problem**: Automatically determine version numbers based on conventional commits.

```bash
#!/bin/bash
# scripts/bump-version.sh

# Get current version
CURRENT_VERSION=$(git describe --tags --abbrev=0)
echo "Current version: $CURRENT_VERSION"

# Analyze commits since last tag
COMMITS=$(git log ${CURRENT_VERSION}..HEAD --oneline)

# Determine version bump type
MAJOR_BUMP=false
MINOR_BUMP=false
PATCH_BUMP=false

while IFS= read -r commit; do
    if [[ $commit =~ feat(\(.+\))?!: || $commit =~ fix(\(.+\))?!: ]]; then
        MAJOR_BUMP=true
    elif [[ $commit =~ feat(\(.+\))?: ]]; then
        MINOR_BUMP=true
    elif [[ $commit =~ fix(\(.+\))?: ]]; then
        PATCH_BUMP=true
    fi
done <<< "$COMMITS"

# Calculate new version
IFS='.' read -ra VERSION_PARTS <<< "${CURRENT_VERSION#v}"
MAJOR=${VERSION_PARTS[0]}
MINOR=${VERSION_PARTS[1]}
PATCH=${VERSION_PARTS[2]}

if [ "$MAJOR_BUMP" = true ]; then
    MAJOR=$((MAJOR + 1))
    MINOR=0
    PATCH=0
elif [ "$MINOR_BUMP" = true ]; then
    MINOR=$((MINOR + 1))
    PATCH=0
elif [ "$PATCH_BUMP" = true ]; then
    PATCH=$((PATCH + 1))
else
    echo "No version bump needed"
    exit 0
fi

NEW_VERSION="v${MAJOR}.${MINOR}.${PATCH}"
echo "New version: $NEW_VERSION"

# Create tag
git tag -a $NEW_VERSION -m "Release $NEW_VERSION"
git push origin $NEW_VERSION
```

---

## Code Review Workflows

### Scenario 7: Large Pull Request Management

**Problem**: A pull request has become too large and needs to be split for better review.

```bash
# Original large feature branch
git log --oneline feature/large-feature
# a1b2c3d Add user dashboard UI
# e4f5g6h Add dashboard API endpoints
# i7j8k9l Add user authentication
# m0n1o2p Add database migrations
# q3r4s5t Add user models

# Strategy: Split into logical chunks

# 1. Create base branch for the feature
git checkout main
git checkout -b feature/user-system-base

# 2. Cherry-pick foundational commits
git cherry-pick q3r4s5t  # User models
git cherry-pick m0n1o2p  # Database migrations
git push origin feature/user-system-base

# 3. Create PR for base changes
# Open PR: feature/user-system-base -> main

# 4. Create authentication branch
git checkout feature/user-system-base
git checkout -b feature/user-authentication
git cherry-pick i7j8k9l  # Authentication logic
git push origin feature/user-authentication

# 5. Create API branch  
git checkout feature/user-system-base
git checkout -b feature/user-api
git cherry-pick e4f5g6h  # API endpoints
git push origin feature/user-api

# 6. Create UI branch
git checkout feature/user-api  # Depends on API
git checkout -b feature/user-dashboard
git cherry-pick a1b2c3d  # Dashboard UI
git push origin feature/user-dashboard

# PR dependency chain:
# main <- feature/user-system-base
# feature/user-system-base <- feature/user-authentication
# feature/user-system-base <- feature/user-api  
# feature/user-api <- feature/user-dashboard
```

### Scenario 8: Addressing Review Comments

**Problem**: Reviewer requested changes that affect multiple commits in your PR.

```bash
# Current PR commits:
git log --oneline origin/main..HEAD
# a1b2c3d Add user service tests
# e4f5g6h Implement user service  
# i7j8k9l Add user model

# Review feedback:
# 1. Change username field to email
# 2. Add validation to user service
# 3. Update tests accordingly

# Solution: Interactive rebase to modify multiple commits
git rebase -i origin/main

# In the editor, mark commits to edit:
# edit i7j8k9l Add user model
# edit e4f5g6h Implement user service
# edit a1b2c3d Add user service tests

# For each commit, make the required changes:

# First stop: Update user model
cat > src/models/User.js << 'EOF'
class User {
    constructor(email, name, password) {
        this.email = email;  // Changed from username
        this.name = name;
        this.password = password;
        this.createdAt = new Date();
        
        this.validate();
    }
    
    validate() {
        if (!this.email || !this.email.includes('@')) {
            throw new Error('Valid email is required');
        }
        if (!this.name || this.name.length < 2) {
            throw new Error('Name must be at least 2 characters');
        }
    }
}

module.exports = User;
EOF

git add src/models/User.js
git commit --amend --no-edit
git rebase --continue

# Second stop: Update user service
cat > src/services/UserService.js << 'EOF'
const User = require('../models/User');

class UserService {
    async createUser(userData) {
        // Validation
        if (!userData.email) {
            throw new Error('Email is required');
        }
        
        // Check if user exists
        const existing = await this.findByEmail(userData.email);
        if (existing) {
            throw new Error('User with this email already exists');
        }
        
        const user = new User(userData.email, userData.name, userData.password);
        return await this.save(user);
    }
    
    async findByEmail(email) {
        // Database lookup logic
        return await db.users.findOne({ email });
    }
    
    async save(user) {
        return await db.users.create(user);
    }
}

module.exports = UserService;
EOF

git add src/services/UserService.js
git commit --amend --no-edit
git rebase --continue

# Third stop: Update tests
cat > tests/UserService.test.js << 'EOF'
const UserService = require('../src/services/UserService');

describe('UserService', () => {
    let userService;
    
    beforeEach(() => {
        userService = new UserService();
    });
    
    describe('createUser', () => {
        it('should create user with valid email', async () => {
            const userData = {
                email: 'test@example.com',
                name: 'Test User',
                password: 'password123'
            };
            
            const user = await userService.createUser(userData);
            expect(user.email).toBe('test@example.com');
        });
        
        it('should reject invalid email', async () => {
            const userData = {
                email: 'invalid-email',
                name: 'Test User',
                password: 'password123'
            };
            
            await expect(userService.createUser(userData))
                .rejects.toThrow('Valid email is required');
        });
        
        it('should reject duplicate email', async () => {
            const userData = {
                email: 'existing@example.com',
                name: 'Test User',
                password: 'password123'
            };
            
            // Mock existing user
            jest.spyOn(userService, 'findByEmail')
                .mockResolvedValue({ email: 'existing@example.com' });
            
            await expect(userService.createUser(userData))
                .rejects.toThrow('User with this email already exists');
        });
    });
});
EOF

git add tests/UserService.test.js
git commit --amend --no-edit
git rebase --continue

# Push updated commits
git push --force-with-lease origin feature/user-system
```

---

## Data Recovery Scenarios

### Scenario 9: Recover Accidentally Deleted Work

**Problem**: Accidentally deleted a branch with important work before merging.

```bash
# Situation: Someone ran `git branch -D feature/important-work`

# 1. Find the lost branch in reflog
git reflog show --all | grep "important-work"
# a1b2c3d refs/heads/feature/important-work@{0}: commit: Complete important feature
# e4f5g6h refs/heads/feature/important-work@{1}: commit: Add tests

# 2. Recreate the branch
git branch feature/important-work-recovered a1b2c3d

# 3. Verify the recovery
git checkout feature/important-work-recovered
git log --oneline -5

# 4. If reflog doesn't help, try fsck
git fsck --lost-found
git show <dangling-commit-hash>
```

### Scenario 10: Recover from Corrupted Repository

**Problem**: Repository corruption after power failure during Git operation.

```bash
# 1. Check repository integrity
git fsck --full
# error: object file .git/objects/ab/cd1234... is empty
# error: HEAD: invalid sha1 pointer 1234567890123456789012345678901234567890

# 2. Try to recover from backup
cp -r .git .git.backup

# 3. Reset HEAD to last known good commit
# Find last good commit in reflog
git reflog | head -20

# If reflog is corrupted, try packed refs
cat .git/packed-refs
git update-ref HEAD <last-good-commit-hash>

# 4. Rebuild index if corrupted
rm .git/index
git reset

# 5. If severe corruption, clone from remote
cd ..
git clone https://github.com/yourrepo/project.git project-recovered
cd project-recovered

# Copy any uncommitted work from corrupted repo
cp ../project-corrupted/uncommitted-file.js .
git add uncommitted-file.js
git commit -m "Recover uncommitted work from corrupted repository"
```

---

## Large Codebase Management

### Scenario 11: Monorepo with Sparse Checkout

**Problem**: Working with a large monorepo but only need specific directories for your work.

```bash
# 1. Clone with filter (Git 2.19+)
git clone --filter=blob:none https://github.com/company/monorepo.git
cd monorepo

# 2. Enable sparse checkout
git config core.sparseCheckout true

# 3. Define what you want to checkout
cat > .git/info/sparse-checkout << 'EOF'
# Only checkout specific services
services/user-service/
services/notification-service/
shared/common/
docs/
*.md
package.json
EOF

# 4. Apply sparse checkout
git read-tree -m -u HEAD

# 5. Verify what's checked out
find . -type f | head -20

# 6. Working with sparse checkout
# Add more paths as needed
echo "services/payment-service/" >> .git/info/sparse-checkout
git read-tree -m -u HEAD

# 7. Temporarily disable sparse checkout
git config core.sparseCheckout false
git read-tree -m -u HEAD
```

### Scenario 12: Large File Management with Git LFS

**Problem**: Repository contains large assets that slow down clone and fetch operations.

```bash
# 1. Install and initialize Git LFS
git lfs install

# 2. Track large files
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.mp4"
git lfs track "assets/images/*.png"

# 3. Add .gitattributes
git add .gitattributes
git commit -m "Add LFS tracking for large files"

# 4. Move existing large files to LFS
# For existing files in history, use migration
git lfs migrate import --include="*.psd,*.zip" --everything

# 5. Verify LFS files
git lfs ls-files
# 12345678 * assets/design.psd
# 87654321 * assets/video.mp4

# 6. Clone with LFS
git lfs clone https://github.com/yourrepo/project.git

# 7. Work with LFS files normally
cp new-design.psd assets/
git add assets/new-design.psd
git commit -m "Add new design mockup"
git push origin main

# 8. Download only pointers (for CI/CD)
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/yourrepo/project.git
```

---

## Integration Challenges

### Scenario 13: Merging Repositories

**Problem**: Need to combine two separate repositories while preserving history.

```bash
# Repositories to merge:
# - main-project (keep as primary)
# - utility-library (merge as subdirectory)

# 1. In main project, add remote for utility library
cd main-project
git remote add utility-lib ../utility-library
git fetch utility-lib

# 2. Create merge branch
git checkout -b merge-utility-lib

# 3. Create subdirectory for merged content
mkdir lib/utility
git add lib/utility
git commit -m "Prepare directory for utility library merge"

# 4. Merge utility library with subtree strategy
git merge -s ours utility-lib/main --allow-unrelated-histories
git read-tree --prefix=lib/utility/ -u utility-lib/main
git commit -m "Merge utility-library as subdirectory

- Preserve full history of utility-library
- Place files under lib/utility/
- Maintain both project lineages"

# 5. Alternative: Use subtree merge properly
git subtree add --prefix=lib/utility utility-lib main

# 6. Update imports in main project
# Update file paths from:
# const utils = require('utility-library');
# To:
# const utils = require('./lib/utility');

# 7. Clean up
git remote remove utility-lib
git branch -d merge-utility-lib
```

### Scenario 14: Cross-Repository Dependencies

**Problem**: Managing dependencies between multiple related repositories.

```bash
# Project structure:
# - shared-components (library)
# - web-app (depends on shared-components)
# - mobile-app (depends on shared-components)

# Solution 1: Git submodules
cd web-app
git submodule add https://github.com/company/shared-components.git lib/shared

# Update shared components
cd lib/shared
git pull origin main
cd ../..
git add lib/shared
git commit -m "Update shared components to latest version"

# Solution 2: Package.json with Git URLs
# In web-app/package.json:
{
    "dependencies": {
        "shared-components": "git+https://github.com/company/shared-components.git#v1.2.0"
    }
}

# Solution 3: Automated dependency updates
# .github/workflows/update-dependencies.yml
name: Update Dependencies
on:
  repository_dispatch:
    types: [shared-components-updated]

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Update submodule
        run: |
          git submodule update --remote lib/shared
          git add lib/shared
          git commit -m "Auto-update shared components"
          git push
```

---

## Performance and Optimization

### Scenario 15: Optimizing Large Repository Performance

**Problem**: Git operations are slow on a large repository with extensive history.

```bash
# 1. Analyze repository size
git count-objects -vH
# count 150000
# size 2.5 GiB
# in-pack 148000
# packs 1
# size-pack 1.8 GiB

# 2. Identify large objects
git rev-list --objects --all | 
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' |
  sed -n 's/^blob //p' |
  sort --numeric-sort --key=2 |
  tail -20

# 3. Clean up repository
git gc --aggressive --prune=now

# 4. Repack repository
git repack -ad

# 5. For very large repos, consider partial clone
git clone --filter=blob:limit=1m https://github.com/large/repo.git

# 6. Use shallow clone for CI
git clone --depth 1 https://github.com/large/repo.git

# 7. Enable file system monitor (if available)
git config core.fsmonitor true
git config core.untrackedcache true

# 8. Optimize for specific workflows
# For frequent status checks:
git config feature.manyFiles true

# For repositories with many refs:
git config core.preloadindex true
```

### Scenario 16: Repository Split

**Problem**: Repository has grown too large and needs to be split into multiple repositories.

```bash
# Original structure:
# monorepo/
# ├── frontend/
# ├── backend/
# ├── mobile/
# └── shared/

# Goal: Split into separate repositories while preserving history

# 1. Extract frontend
git clone --no-hardlinks monorepo frontend-repo
cd frontend-repo

# 2. Use filter-repo to extract only frontend directory
git filter-repo --path frontend/ --path-rename frontend/:

# 3. Clean up references
git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d

# 4. Repeat for other components
cd ../
git clone --no-hardlinks monorepo backend-repo
cd backend-repo
git filter-repo --path backend/ --path-rename backend/:

# 5. Create shared library repository
cd ../
git clone --no-hardlinks monorepo shared-lib-repo
cd shared-lib-repo
git filter-repo --path shared/ --path-rename shared/:

# 6. Update main repository to use new structure
cd ../monorepo
git submodule add https://github.com/company/frontend-repo.git frontend
git submodule add https://github.com/company/backend-repo.git backend
git submodule add https://github.com/company/shared-lib-repo.git shared

# Remove old directories
git rm -r frontend/ backend/ mobile/ shared/
git commit -m "Convert to multi-repository structure with submodules"

# 7. Update CI/CD to work with new structure
# .github/workflows/build.yml
jobs:
  build:
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Build all components
        run: |
          cd frontend && npm run build
          cd ../backend && npm run build
```

---

This comprehensive guide provides practical solutions for real-world Git scenarios. Each example includes context, multiple solution approaches, and complete implementation details to help you handle complex situations in professional development environments.