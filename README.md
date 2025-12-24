# 🤖 Claude Code PR Review

A [Claude Code](https://claude.ai/code) custom slash command that automatically reviews GitHub Pull Requests with inline comments, following your project's coding conventions and custom rules.

![Claude Code Review](https://img.shields.io/badge/Claude_Code-Skill-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## ✨ Features

- 🔍 **Automatic Code Analysis** - Reviews code for bugs, security issues, and best practices
- 💬 **Inline Comments** - Posts comments directly on specific lines of code
- 📏 **Custom Rules** - Define your own coding standards via `.claude-review.yml`
- 📖 **CLAUDE.md Support** - Reads project conventions from CLAUDE.md
- 🌐 **Multi-Language** - Works with Swift, TypeScript, Python, Go, Rust, and more
- 🚀 **One Command** - Just run `/review <PR_URL>` and let Claude do the rest

## 📦 Installation

### Quick Install (1-liner)

```bash
mkdir -p ~/.claude/commands && curl -o ~/.claude/commands/review.md \
  https://raw.githubusercontent.com/Nghicv/claude-code-pr-review/main/review.md
```

### Project-Local Installation

```bash
mkdir -p .claude/commands && curl -o .claude/commands/review.md \
  https://raw.githubusercontent.com/Nghicv/claude-code-pr-review/main/review.md
```

## 🔧 Prerequisites

1. **GitHub CLI** - Install and authenticate:
   ```bash
   # Install
   brew install gh        # macOS
   sudo apt install gh    # Ubuntu/Debian

   # Authenticate
   gh auth login
   ```

2. **Claude Code** - [Install Claude Code](https://claude.ai/code)

3. **(Optional) Auto-approve `gh` commands** - To avoid yes/no prompts:

   Add these patterns to your Claude Code permissions (run `/permissions` or edit settings):
   ```
   Bash(gh pr view:*)
   Bash(gh pr diff:*)
   Bash(gh api:*)
   ```

## 🚀 Usage

```bash
/user:review https://github.com/owner/repo/pull/123
```

Claude will:
1. ✅ Load your custom rules (`.claude-review.yml`)
2. ✅ Read project conventions (`CLAUDE.md`)
3. ✅ Analyze all code changes (silently, no prompts)
4. ✅ Show preview of all comments
5. ✅ **Ask once** → Post Review / Post Summary Only / Cancel
6. ✅ Submit review to GitHub

## 📏 Custom Rules

Create a `.claude-review.yml` file in your repository root to define custom coding standards:

```yaml
# .claude-review.yml

language: swift

rules:
  # Naming conventions
  - id: naming-camelcase
    name: "Use camelCase for variables"
    severity: warning
    examples:
      bad: "let MyVariable = 1"
      good: "let myVariable = 1"

  # Swift-specific
  - id: swift-weak-self
    name: "Use [weak self] in closures"
    severity: error
    description: "Prevent retain cycles in escaping closures"
    examples:
      bad: |
        api.fetch { result in
            self.handle(result)
        }
      good: |
        api.fetch { [weak self] result in
            self?.handle(result)
        }

  - id: swift-no-force-unwrap
    name: "Avoid force unwrap"
    severity: error
    exceptions:
      - "IBOutlets"
      - "Test files"

# Forbidden patterns (will trigger errors)
forbidden_patterns:
  - pattern: "password.*=.*[\"'].*[\"']"
    message: "Hardcoded password detected"

  - pattern: "api[_-]?key.*=.*[\"']"
    message: "Hardcoded API key detected"

# Patterns to warn about
required_patterns:
  - pattern: "// TODO:"
    action: warn
    message: "TODO found - address before merge"

  - pattern: "print\\("
    action: warn
    message: "Debug print statement found"

# Files to skip
ignore:
  - "*.generated.swift"
  - "Pods/**"
  - "node_modules/**"

# Documentation requirements
documentation:
  require_for:
    - "public func"
    - "public class"
```

### Severity Levels

| Level | Emoji | Effect |
|-------|-------|--------|
| `error` | 🔴 | Triggers REQUEST_CHANGES |
| `warning` | 🟡 | Highlighted but won't block |
| `info` | 🔵 | Suggestion only |

## 📖 CLAUDE.md Support

The reviewer also reads your project's `CLAUDE.md` file for additional context:

```markdown
# CLAUDE.md

## Coding Conventions
- Use MVVM architecture
- All ViewModels must be final classes
- Use `[weak self]` in all closures
- Maximum function length: 30 lines

## Naming
- ViewControllers: `*ViewController`
- ViewModels: `*ViewModel`
- Use camelCase for variables
```

## 📝 Example Output

### Inline Comments

```
🔴 **[swift-no-force-unwrap]** Force unwrap detected

Force unwrapping can cause crashes at runtime.

**Fix:**
guard let value = optional else { return }
```

### Review Summary

```markdown
## 🔍 Code Review

**PR:** #123 - Add user authentication
**Changes:** +500 / -20 lines across 8 files

### 📏 Rules Applied
- Project config: `.claude-review.yml` ✓
- CLAUDE.md conventions: ✓
- Default rules: ✓

### ✅ What's Good
- Clean MVVM architecture
- Proper error handling

### 🔍 Review Details
5 inline comment(s) added.

| Severity | Count |
|----------|-------|
| 🔴 Error | 2 |
| 🟡 Warning | 2 |
| 🔵 Info | 1 |

### 📊 Verdict: REQUEST_CHANGES 🔄
```

## 🏷️ Comment Types

| Emoji | Type | Description |
|-------|------|-------------|
| 🔴 | Error | Must fix (from rules with `severity: error`) |
| 🟡 | Warning | Should fix (from rules with `severity: warning`) |
| 🔵 | Info | Suggestion (from rules with `severity: info`) |
| 🐛 | Bug | Logic errors, crashes |
| ⚠️ | Security | Vulnerabilities |
| ⚡ | Performance | Memory leaks, inefficient code |
| 💡 | Suggestion | Improvements |
| 📝 | Convention | Style/naming issues |

## 📁 Project Structure

```
your-repo/
├── .claude-review.yml    # Custom review rules
├── CLAUDE.md             # Project conventions
└── src/
    └── ...
```

## ⚙️ Configuration Reference

See [`.claude-review.example.yml`](.claude-review.example.yml) for a complete example with all available options.

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## 📄 License

MIT License - See [LICENSE](LICENSE) for details.

## 🔗 Links

- [Claude Code](https://claude.ai/code)
- [GitHub CLI](https://cli.github.com/)
- [Example Config](.claude-review.example.yml)

---

Made with ❤️ for better code reviews
