# GitHub Foundations - Hoja de Referencia
# GH Foundations

Welcome to **GH Foundations**! This repository is designed to help modern developers master essential tools and concepts, with a special focus on certification preparation.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Git Explanations](#git-explanations)
3. [Advanced Git Guides](#advanced-git-guides)
4. [Modern Developer Tools](#modern-developer-tools)
5. [Certification Preparation Guide](#certification-preparation-guide)

---

## Project Overview

This project provides resources, guides, and practical examples for developers aiming to strengthen their foundational skills. Whether you're preparing for a certification or looking to improve your workflow, you'll find valuable content here.

---

## Git Explanations

Git is a distributed version control system that enables teams to collaborate efficiently. Below are key concepts and commands every developer should know:

### Key Concepts

- **Repository**: A directory tracked by Git.
- **Commit**: A snapshot of changes in the repository.
- **Branch**: An independent line of development.
- **Merge**: Combining changes from different branches.
- **Clone**: Copying a remote repository locally.
- **Remote**: A version of your project hosted on the internet or network.

### Essential Commands

```bash
# Initialize a new repository
git init

# Clone an existing repository
git clone <repo_url>

# Check status
git status

# Stage changes
git add <file>

# Commit changes
git commit -m "Commit message"

# Push to remote
git push origin <branch>

# Pull from remote
git pull origin <branch>

# Create a new branch
git checkout -b <branch_name>

# Merge a branch
git merge <branch_name>
```

### Best Practices

- Commit often with clear messages.
- Use branches for features and fixes.
- Regularly pull changes to stay up-to-date.
- Resolve conflicts promptly.

---

## Advanced Git Guides

For developers ready to master advanced Git concepts and real-world scenarios:

### 📚 Comprehensive Git Documentation

- **[GIT-MEDIUM.md](./GIT-MEDIUM.md)** - Intermediate Git concepts with practical scenarios
  - Interactive rebase and commit management
  - Advanced stash operations
  - Conflict resolution techniques
  - Git hooks and automation
  - Working with multiple remotes

- **[GIT-ADVANCED.md](./GIT-ADVANCED.md)** - Expert-level Git techniques and complex workflows
  - Advanced merge strategies
  - Submodules and subtrees
  - Git bisect for debugging
  - Performance optimization
  - Git internals and troubleshooting

- **[GIT-MERGE-STRATEGIES.md](./GIT-MERGE-STRATEGIES.md)** - Complete guide to Git merge strategies
  - Fast-forward vs non-fast-forward merges
  - Recursive, octopus, ours, and subtree strategies
  - Conflict resolution patterns
  - Merge vs rebase decision guide

- **[GIT-SCENARIOS.md](./GIT-SCENARIOS.md)** - Real-world problem solving with practical examples
  - Emergency hotfixes and rollback strategies
  - Team collaboration workflows
  - Release management scenarios
  - Data recovery techniques
  - Large codebase management

---

## Modern Developer Tools

Below is a curated set of tools every developer should be familiar with, especially for certification preparation:

### Version Control
- **Git**: Distributed version control system.
- **GitHub/GitLab/Bitbucket**: Remote hosting and collaboration platforms.

### Editors & IDEs
- **Visual Studio Code**: Lightweight, extensible code editor.
- **JetBrains IDEs**: Professional development environments.

### Package Managers
- **npm/yarn/pnpm**: JavaScript package managers.
- **pip/conda**: Python package managers.
- **Homebrew**: macOS/Linux package manager.

### Containers & Virtualization
- **Docker**: Containerization platform.
- **Kubernetes**: Container orchestration.

### CI/CD & Automation
- **GitHub Actions**: Workflow automation.
- **Jenkins**: Automation server.
- **CircleCI/Travis CI**: Continuous integration platforms.

### Cloud Platforms
- **AWS/Azure/GCP**: Major cloud providers.
- **Terraform**: Infrastructure as code.

### Testing & Quality
- **Jest/Mocha**: JavaScript testing frameworks.
- **PyTest**: Python testing framework.
- **ESLint/Prettier**: Code linting and formatting.

### Security
- **Dependabot**: Automated dependency updates.
- **Snyk**: Vulnerability scanning.

### Collaboration
- **Slack/Discord**: Team communication.
- **Trello/Jira**: Project management.

---

## Certification Preparation Guide

To prepare for developer certifications, focus on:

1. **Understanding Core Concepts**: Master Git, containers, CI/CD, and cloud basics.
2. **Hands-on Practice**: Use labs, sample projects, and exercises.
3. **Review Official Documentation**: Always check the latest docs for your certification.
4. **Mock Exams**: Take practice tests to identify gaps.
5. **Join Communities**: Engage in forums and study groups.

### Recommended Certifications

- **GitHub Foundations**
- **AWS Certified Developer**
- **Azure Developer Associate**
- **Google Associate Cloud Engineer**
- **Certified Kubernetes Application Developer (CKAD)**

---

For questions or contributions, feel free to open an issue or pull request!
## 📚 Configuración Inicial de Git

### Configuración de Usuario
```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
git config --list  # Ver todas las configuraciones
```

## 🚀 Comandos Básicos de Git

### Inicialización y Clonación
```bash
git init                          # Inicializar repositorio local
git clone <url>                   # Clonar repositorio remoto
git clone <url> --branch <branch> # Clonar rama específica
```

### Estado y Staging
```bash
git status                        # Ver estado del repositorio
git add <archivo>                 # Agregar archivo al staging
git add .                         # Agregar todos los cambios
git add -A                        # Agregar todos (incluyendo eliminados)
git reset <archivo>               # Quitar del staging
```

### Commits
```bash
git commit -m "mensaje"           # Crear commit
git commit -am "mensaje"          # Add + commit (solo archivos tracked)
git commit --amend               # Modificar último commit
git log                          # Ver historial
git log --oneline                # Historial resumido
git log --graph --all --oneline  # Ver gráfico de ramas
```

## 🌿 Ramas (Branches)

### Gestión de Ramas
```bash
git branch                        # Listar ramas locales
git branch -a                     # Listar todas las ramas
git branch <nombre>               # Crear rama
git checkout <rama>               # Cambiar de rama
git checkout -b <nueva-rama>      # Crear y cambiar a nueva rama
git switch <rama>                 # Cambiar de rama (nuevo comando)
git switch -c <nueva-rama>        # Crear y cambiar (nuevo comando)
```

### Fusión y Rebase
```bash
git merge <rama>                  # Fusionar rama en la actual
git merge --no-ff <rama>          # Merge sin fast-forward
git rebase <rama>                 # Rebase de la rama actual
git cherry-pick <commit-hash>     # Aplicar commit específico
```

### Eliminar Ramas
```bash
git branch -d <rama>              # Eliminar rama local (merged)
git branch -D <rama>              # Forzar eliminación
git push origin --delete <rama>   # Eliminar rama remota
```

## 🔄 Trabajando con Remotos

### Gestión de Remotos
```bash
git remote -v                     # Ver remotos configurados
git remote add origin <url>       # Agregar remoto
git remote remove origin          # Eliminar remoto
git remote set-url origin <url>   # Cambiar URL remoto
```

### Push y Pull
```bash
git push                          # Subir cambios
git push -u origin <rama>         # Subir y establecer upstream
git push --force                  # Forzar push (¡cuidado!)
git pull                          # Traer y fusionar cambios
git fetch                         # Solo traer cambios
git fetch --all                   # Traer de todos los remotos
```

## 🔍 Inspección y Comparación

```bash
git diff                          # Ver cambios no staged
git diff --staged                 # Ver cambios staged
git diff <rama1>..<rama2>         # Comparar ramas
git show <commit>                 # Ver detalles de commit
git blame <archivo>               # Ver quién modificó cada línea
```

## ↩️ Deshacer Cambios

```bash
git checkout -- <archivo>         # Descartar cambios locales
git restore <archivo>             # Restaurar archivo (nuevo comando)
git reset HEAD~1                  # Deshacer último commit (soft)
git reset --hard HEAD~1           # Deshacer último commit (hard)
git revert <commit>               # Revertir commit con nuevo commit
git clean -fd                     # Eliminar archivos no tracked
```

## 🏷️ Tags

```bash
git tag                           # Listar tags
git tag <nombre>                  # Crear tag ligero
git tag -a <nombre> -m "mensaje"  # Crear tag anotado
git push origin <tag>             # Subir tag específico
git push --tags                   # Subir todos los tags
git tag -d <tag>                  # Eliminar tag local
```

## 📦 Stash (Guardado Temporal)

```bash
git stash                         # Guardar cambios temporalmente
git stash save "mensaje"          # Guardar con descripción
git stash list                    # Ver lista de stash
git stash apply                   # Aplicar último stash
git stash pop                     # Aplicar y eliminar stash
git stash drop                    # Eliminar último stash
```

## 🔧 GitHub CLI (gh)

### Autenticación
```bash
gh auth login                     # Iniciar sesión
gh auth status                    # Ver estado de autenticación
```

### Repositorios
```bash
gh repo create <nombre>           # Crear repositorio
gh repo clone <owner>/<repo>     # Clonar repositorio
gh repo fork                      # Hacer fork
gh repo view                      # Ver info del repo
```

### Pull Requests
```bash
gh pr create                      # Crear PR
gh pr list                        # Listar PRs
gh pr view <número>               # Ver PR específico
gh pr checkout <número>           # Checkout a PR
gh pr merge <número>              # Fusionar PR
```

### Issues
```bash
gh issue create                   # Crear issue
gh issue list                     # Listar issues
gh issue view <número>            # Ver issue
gh issue close <número>           # Cerrar issue
```

## 🔑 Conceptos Clave para el Examen

### Flujos de Trabajo
- **GitHub Flow**: main → feature branch → PR → merge
- **Git Flow**: main, develop, feature/*, release/*, hotfix/*

### Mejores Prácticas
1. **Commits atómicos**: Un commit = un cambio lógico
2. **Mensajes descriptivos**: Qué y por qué del cambio
3. **Branch naming**: feature/, bugfix/, hotfix/
4. **PR reviews**: Al menos 1 revisión antes de merge
5. **Protección de ramas**: Configurar reglas en main/master

### Archivos Importantes
- `.gitignore`: Archivos a ignorar
- `README.md`: Documentación principal
- `LICENSE`: Licencia del proyecto
- `.github/`: Configuraciones de GitHub
- `CONTRIBUTING.md`: Guía de contribución
- `CODE_OF_CONDUCT.md`: Código de conducta

### GitHub Actions (Básico)
```yaml
# .github/workflows/ejemplo.yml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test
```

### Markdown Esencial
```markdown
# Título H1
## Título H2
**Negrita**
*Cursiva*
`código inline`
[Enlace](url)
![Imagen](url)
- Lista
1. Lista numerada
> Cita
```

## 📊 Métricas y Insights
- **Contributors**: Quién contribuye al proyecto
- **Traffic**: Visitantes y clones
- **Commits**: Frecuencia de commits
- **Code frequency**: Adiciones/eliminaciones
- **Pulse**: Resumen de actividad

## 🛡️ Seguridad
- **Dependabot**: Actualizaciones automáticas
- **Secret scanning**: Detección de secretos
- **Code scanning**: Análisis de vulnerabilidades
- **Security policies**: SECURITY.md

## 💡 Tips para el Examen
1. Conocer diferencia entre Git y GitHub
2. Entender colaboración: forks, PRs, issues
3. Familiarizarse con GitHub.com UI
4. Conocer licencias open source comunes
5. Entender branch protection rules
6. Saber sobre GitHub Pages básico
7. Conocer GitHub Marketplace
8. Entender organizaciones y equipos

---
*Última actualización: Agosto 2025*