# 🤖 Claude Code Review

A [Claude Code](https://claude.ai/code) custom slash command that automatically reviews GitHub Pull Requests and posts inline comments on specific code lines.

![Claude Code Review Demo](https://img.shields.io/badge/Claude_Code-Skill-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## ✨ Features

- 🔍 **Automatic Code Analysis** - Reviews code for bugs, security issues, and best practices
- 💬 **Inline Comments** - Posts comments directly on specific lines of code
- 🌐 **Multi-Language Support** - Works with Swift, TypeScript, Python, Go, Rust, and more
- 📊 **Structured Summary** - Provides organized review with categorized findings
- 🚀 **One Command** - Just run `/review <PR_URL>` and let Claude do the rest

## 📦 Installation

### Option 1: Global Installation (Recommended)

Copy the skill file to your Claude Code commands directory:

```bash
# Create the commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Download the review skill
curl -o ~/.claude/commands/review.md \
  https://raw.githubusercontent.com/user/claude-code-review/main/review.md
```

### Option 2: Project-Local Installation

Add to a specific project:

```bash
# In your project root
mkdir -p .claude/commands
curl -o .claude/commands/review.md \
  https://raw.githubusercontent.com/user/claude-code-review/main/review.md
```

## 🔧 Prerequisites

1. **GitHub CLI** - Install and authenticate:
   ```bash
   # Install (macOS)
   brew install gh

   # Install (Ubuntu/Debian)
   sudo apt install gh

   # Authenticate
   gh auth login
   ```

2. **Claude Code** - Make sure you have [Claude Code](https://claude.ai/code) installed

## 🚀 Usage

In Claude Code, simply run:

```
/user:review https://github.com/owner/repo/pull/123
```

Or for project-local installation:

```
/project:review https://github.com/owner/repo/pull/123
```

Claude will:
1. Fetch the PR details and diff
2. Analyze all code changes
3. Identify bugs, security issues, and improvements
4. Post inline comments on specific lines
5. Submit a comprehensive review summary

## 📝 Example Output

### Inline Comments
![Inline Comment Example](docs/inline-comment.png)

Comments are posted directly on the relevant lines:

```
🐛 **Bug:** This will crash if `user` is nil.

**Fix:**
guard let user = user else { return }
```

### Review Summary

```markdown
## 🔍 Code Review

**PR:** #123 - Add user authentication
**Changes:** +500 / -20 lines across 8 files

### ✅ What's Good
- Clean separation of concerns
- Proper error handling

### 🔍 Review Details
5 inline comment(s) added.

| Type | Count |
|------|-------|
| 🐛 Bugs | 2 |
| ⚠️ Security | 1 |
| 💡 Suggestions | 2 |

### 📊 Verdict: REQUEST_CHANGES 🔄
```

## 🏷️ Comment Types

| Emoji | Type | Description |
|-------|------|-------------|
| 🐛 | Bug | Logic errors, crashes, incorrect behavior |
| ⚠️ | Security | Vulnerabilities, injection, secrets exposure |
| ⚡ | Performance | Memory leaks, inefficient algorithms |
| 💡 | Suggestion | Improvements, refactoring opportunities |
| 📝 | Style | Naming, formatting, documentation |
| ❓ | Question | Clarification needed |
| ✅ | Good | Praise for well-written code |

## ⚙️ Customization

You can modify `review.md` to:
- Add language-specific checks
- Change comment format
- Adjust review criteria
- Add custom rules for your team

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## 📄 License

MIT License - See [LICENSE](LICENSE) for details.

## 🔗 Related

- [Claude Code](https://claude.ai/code) - The AI-powered CLI
- [GitHub CLI](https://cli.github.com/) - GitHub's official CLI
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)

---

Made with ❤️ by the community
