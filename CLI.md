# GitHub CLI Documentation

The GitHub CLI (`gh`) is a command-line tool for interacting with GitHub from your terminal.

---

## Installation
- **Linux/macOS:** `brew install gh` or download from [GitHub CLI Releases](https://github.com/cli/cli/releases)
- **Windows:** `winget install GitHub.cli`

## Authentication
```bash
gh auth login
```

## Common Commands
- **Repository Management:**
  - `gh repo clone <owner>/<repo>`
  - `gh repo create`
- **Issues & PRs:**
  - `gh issue list`
  - `gh pr create`
  - `gh pr merge`
- **Codespaces:**
  - `gh codespace list`
  - `gh codespace create`
- **Actions:**
  - `gh run list`
  - `gh run view <run-id>`

## Help & Docs
- `gh help`
- [GitHub CLI Docs](https://cli.github.com/manual/)

---

> The CLI is extensible and supports scripting for automation.
