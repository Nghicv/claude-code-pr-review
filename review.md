# Review Pull Request

Automatically review a GitHub Pull Request with inline comments, following project-specific coding conventions and custom rules.

## Input
$ARGUMENTS = The Pull Request URL (e.g., https://github.com/owner/repo/pull/123)

## Prerequisites
- GitHub CLI (`gh`) must be installed and authenticated
- Run `gh auth login` if not already authenticated

## Workflow

### Step 1: Parse the PR URL
Extract `owner`, `repo`, and `pr_number` from the input URL.

Example: `https://github.com/facebook/react/pull/456`
→ owner=`facebook`, repo=`react`, pr_number=`456`

### Step 2: Fetch PR Information
```bash
gh pr view <PR_NUMBER> --repo <OWNER/REPO> --json title,body,files,additions,deletions,baseRefName,headRefName,headRefOid,state
```

Check if PR is open. If merged or closed, inform the user.

### Step 3: Load Custom Rules and Conventions

**IMPORTANT:** Before reviewing, check for project-specific rules:

#### 3.1 Check for `.claude-review.yml` or `.claude-review.json`
```bash
gh api repos/<OWNER>/<REPO>/contents/.claude-review.yml --jq '.content' | base64 -d
# or
gh api repos/<OWNER>/<REPO>/contents/.claude-review.json --jq '.content' | base64 -d
```

If found, parse and apply:
- `rules`: Custom rules to enforce
- `forbidden_patterns`: Patterns that must not appear
- `required_patterns`: Patterns to warn about
- `ignore`: Files/paths to skip
- `documentation`: Documentation requirements
- `architecture`: Layer dependency rules

#### 3.2 Check for `CLAUDE.md` in repository
```bash
gh api repos/<OWNER>/<REPO>/contents/CLAUDE.md --jq '.content' | base64 -d
```

If found, extract coding conventions and project-specific guidelines.

#### 3.3 Check for common config files
Also check for existing linter configs to understand project standards:
- `.swiftlint.yml` (Swift)
- `.eslintrc.*` (JavaScript/TypeScript)
- `pylint.rc` / `.flake8` (Python)
- `.golangci.yml` (Go)

### Step 4: Get the PR Diff
```bash
gh pr diff <PR_NUMBER> --repo <OWNER/REPO>
```

Read the full diff to understand all changes.

### Step 5: Analyze the Code

Review ALL changed files (except those in `ignore` list), applying:

#### 5.1 Custom Rules (from `.claude-review.yml`)
For each rule in config:
- Check if code violates the rule
- Use severity level (error/warning/info) from config
- Provide fix examples from config if available

#### 5.2 Forbidden Patterns
Scan for any `forbidden_patterns`:
- Hardcoded secrets
- Banned functions
- Security vulnerabilities

#### 5.3 Required Patterns
Check for `required_patterns` and warn if found:
- TODO/FIXME comments
- Debug statements

#### 5.4 Default Checks (always apply)

**Critical Issues (must report):**
- 🐛 **Bugs**: Logic errors, null pointer issues, off-by-one errors, race conditions
- ⚠️ **Security**: SQL injection, XSS, hardcoded secrets, improper input validation
- ⚡ **Performance**: Memory leaks, N+1 queries, inefficient algorithms

**Code Quality:**
- 💡 **Suggestions**: Better approaches, design patterns
- 📝 **Convention**: Violations of project coding standards
- ❓ **Questions**: Unclear logic, missing context

#### 5.5 Language-Specific Checks
- **Swift/iOS**: `[weak self]`, main thread UI, retain cycles, access control
- **JavaScript/TypeScript**: Async/await, type safety, memory leaks
- **Python**: Type hints, exception handling, resource cleanup
- **Go**: Error handling, goroutine leaks, defer usage
- **Rust**: Ownership, unsafe blocks, error handling

### Step 6: Determine Line Numbers

For each issue found:
1. Verify the line exists in the NEW version of the file
2. Use diff output to find correct line numbers
3. Only comment on lines that are added (+) or context in the diff

### Step 7: Post Review with Inline Comments

Use the GitHub API with JSON input:

```bash
gh api repos/<OWNER>/<REPO>/pulls/<PR_NUMBER>/reviews \
  --method POST \
  --input - << 'EOF'
{
  "event": "<EVENT_TYPE>",
  "body": "<SUMMARY_MARKDOWN>",
  "comments": [
    {
      "path": "relative/path/to/file.ext",
      "line": <LINE_NUMBER>,
      "body": "<COMMENT_WITH_EMOJI_PREFIX>"
    }
  ]
}
EOF
```

**Event Types:**
- `"COMMENT"` - Neutral review
- `"APPROVE"` - Approve PR (cannot approve own PR)
- `"REQUEST_CHANGES"` - Request changes (use for errors)

**Choose based on findings:**
- Only `info` level issues → `"APPROVE"` or `"COMMENT"`
- Has `warning` level issues → `"COMMENT"`
- Has `error` level issues → `"REQUEST_CHANGES"`

### Step 8: Format the Summary

```markdown
## 🔍 Code Review

**PR:** #<number> - <title>
**Branch:** `<head>` → `<base>`
**Changes:** +<additions> / -<deletions> lines across <file_count> files

### 📋 Summary
<1-2 sentence description of what this PR does>

### 📏 Rules Applied
- Project config: `.claude-review.yml` ✓/✗
- CLAUDE.md conventions: ✓/✗
- Default rules: ✓

### ✅ What's Good
- <Positive point 1>
- <Positive point 2>

### 🔍 Review Details
<N> inline comment(s) added to specific lines.

| Severity | Count |
|----------|-------|
| 🔴 Error | X |
| 🟡 Warning | X |
| 🔵 Info | X |

### 📊 Verdict: <APPROVE ✅ | REQUEST_CHANGES 🔄 | COMMENT 💬>

<Final recommendation>

---
*🤖 Reviewed by [Claude Code](https://claude.ai/code)*
```

## Comment Format

Include rule ID when from custom config:

```
🔴 **[swift-no-force-unwrap]** Force unwrap detected

Force unwrapping can cause crashes at runtime.

**Fix:**
\`\`\`swift
guard let value = optional else { return }
\`\`\`
```

## Comment Prefixes by Severity

| Severity | Emoji | From Config |
|----------|-------|-------------|
| Error | 🔴 | `severity: error` |
| Warning | 🟡 | `severity: warning` |
| Info | 🔵 | `severity: info` |
| Bug | 🐛 | Default check |
| Security | ⚠️ | Default check |
| Performance | ⚡ | Default check |
| Suggestion | 💡 | Default check |
| Convention | 📝 | From CLAUDE.md |
| Good | ✅ | Praise |

## Error Handling

- If config file not found → Use default rules only
- If PR URL invalid → Show correct format
- If `gh` not authenticated → Instruct to run `gh auth login`
- If PR merged/closed → Inform user
- If no issues found → Post approval with positive feedback

## Tips

1. **Check config first** - Always look for `.claude-review.yml`
2. **Respect ignore patterns** - Skip files in ignore list
3. **Use rule IDs** - Reference rule IDs in comments for traceability
4. **Match severity** - Use severity from config, not just default
5. **Show examples** - Use examples from config when available
6. **Be consistent** - Apply same rules across all files
