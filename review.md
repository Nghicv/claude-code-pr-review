# Review Pull Request

Automatically review a GitHub Pull Request and post inline comments on specific code lines.

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

### Step 3: Get the PR Diff
```bash
gh pr diff <PR_NUMBER> --repo <OWNER/REPO>
```

Read the full diff to understand all changes.

### Step 4: Analyze the Code

Review ALL changed files in the diff, focusing on:

**Critical Issues (must report):**
- 🐛 **Bugs**: Logic errors, null pointer issues, off-by-one errors, race conditions
- ⚠️ **Security**: SQL injection, XSS, hardcoded secrets, improper input validation
- ⚡ **Performance**: Memory leaks, N+1 queries, inefficient algorithms, missing indexes

**Code Quality (report if significant):**
- 💡 **Suggestions**: Better approaches, design patterns, refactoring opportunities
- 📝 **Style**: Naming conventions, code organization, missing documentation
- ❓ **Questions**: Unclear logic, missing context, potential edge cases

**Language-Specific Checks:**
- **Swift/iOS**: `[weak self]` in closures, main thread UI updates, retain cycles
- **JavaScript/TypeScript**: Async/await errors, memory leaks, type safety
- **Python**: Type hints, exception handling, resource cleanup
- **Go**: Error handling, goroutine leaks, defer usage
- **Rust**: Ownership issues, unsafe blocks, error handling

### Step 5: Determine Line Numbers

For each issue found, you MUST verify the exact line number:

1. The line must exist in the NEW version of the file (right side of diff)
2. Use the diff output to find the correct line number
3. Look for `@@ -old,count +new,count @@` headers to calculate line numbers
4. Only comment on lines that are added (+) or unchanged (context) in the diff

### Step 6: Post Review with Inline Comments

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
- `"COMMENT"` - Neutral review with comments
- `"APPROVE"` - Approve the PR (cannot approve your own PR)
- `"REQUEST_CHANGES"` - Request changes before merge

**Choose event type based on findings:**
- No issues found → `"APPROVE"` (or `"COMMENT"` if own PR)
- Minor suggestions only → `"COMMENT"`
- Bugs or security issues → `"REQUEST_CHANGES"`

### Step 7: Format the Summary

```markdown
## 🔍 Code Review

**PR:** #<number> - <title>
**Branch:** `<head>` → `<base>`
**Changes:** +<additions> / -<deletions> lines across <file_count> files

### 📋 Summary
<1-2 sentence description of what this PR does>

### ✅ What's Good
- <Positive point 1>
- <Positive point 2>

### 🔍 Review Details
<N> inline comment(s) added to specific lines.

| Type | Count |
|------|-------|
| 🐛 Bugs | X |
| ⚠️ Security | X |
| ⚡ Performance | X |
| 💡 Suggestions | X |

### 📊 Verdict: <APPROVE ✅ | REQUEST_CHANGES 🔄 | COMMENT 💬>

<Final recommendation>

---
*🤖 Reviewed by [Claude Code](https://claude.ai/code)*
```

## Comment Format Examples

### Bug Report
```
🐛 **Bug:** This will crash if `user` is nil.

**Problem:** Force unwrapping optional without checking.

**Fix:**
\`\`\`swift
guard let user = user else { return }
\`\`\`
```

### Security Issue
```
⚠️ **Security:** SQL injection vulnerability.

**Problem:** User input is directly concatenated into SQL query.

**Fix:**
\`\`\`python
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
\`\`\`
```

### Performance Issue
```
⚡ **Performance:** Potential memory leak due to retain cycle.

**Problem:** Strong reference to `self` in escaping closure.

**Fix:**
\`\`\`swift
api.fetch { [weak self] result in
    guard let self else { return }
    self.handleResult(result)
}
\`\`\`
```

### Suggestion
```
💡 **Suggestion:** Consider using `guard` for early return.

This improves readability by reducing nesting:
\`\`\`swift
guard let data = data else {
    completion(.failure(.noData))
    return
}
// Continue with data...
\`\`\`
```

## Error Handling

- If PR URL is invalid, inform the user of the correct format
- If `gh` is not authenticated, instruct user to run `gh auth login`
- If PR is already merged/closed, inform the user
- If no issues found, still post a positive review with approval

## Tips for Quality Reviews

1. **Read the PR description** - Understand the intent before reviewing
2. **Check the full context** - Don't comment on code without understanding it
3. **Be constructive** - Suggest fixes, not just point out problems
4. **Prioritize** - Focus on bugs and security over style
5. **Acknowledge good code** - Mention what's done well
6. **Be specific** - Include line numbers and code examples
7. **Consider edge cases** - Think about error conditions and boundaries
