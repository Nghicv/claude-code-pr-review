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

**Check for project-specific rules (do all checks in one step):**

#### 3.1 Check for `.claude-review.yml`
```bash
gh api repos/<OWNER>/<REPO>/contents/.claude-review.yml --jq '.content' 2>/dev/null | base64 -d
```

If found, parse and apply:
- `rules`: Custom rules to enforce
- `forbidden_patterns`: Patterns that must not appear
- `required_patterns`: Patterns to warn about
- `ignore`: Files/paths to skip

#### 3.2 Check for `CLAUDE.md`
```bash
gh api repos/<OWNER>/<REPO>/contents/CLAUDE.md --jq '.content' 2>/dev/null | base64 -d
```

If found, extract coding conventions and project-specific guidelines.

### Step 4: Get the PR Diff
```bash
gh pr diff <PR_NUMBER> --repo <OWNER/REPO>
```

### Step 5: Analyze the Code (NO USER INTERACTION)

Review ALL changed files silently, applying:

#### 5.1 Custom Rules (from `.claude-review.yml`)
- Check if code violates each rule
- Note severity level (error/warning/info)
- Collect fix examples from config

#### 5.2 Default Checks
- 🐛 **Bugs**: Logic errors, null pointer issues, race conditions
- ⚠️ **Security**: SQL injection, XSS, hardcoded secrets
- ⚡ **Performance**: Memory leaks, retain cycles, N+1 queries
- 💡 **Suggestions**: Better approaches, design patterns
- 📝 **Convention**: Violations of project coding standards

#### 5.3 Language-Specific Checks
- **Swift/iOS**: `[weak self]`, main thread UI, retain cycles, access control
- **JavaScript/TypeScript**: Async/await, type safety, memory leaks
- **Python**: Type hints, exception handling, resource cleanup
- **Go**: Error handling, goroutine leaks, defer usage

### Step 6: Prepare Review (COLLECT ALL COMMENTS)

**DO NOT post yet.** Collect all findings into a structured preview:

```
═══════════════════════════════════════════════════════════════
                    📋 REVIEW PREVIEW
═══════════════════════════════════════════════════════════════

PR: #<number> - <title>
Files: <count> | Changes: +<add> / -<del>

───────────────────────────────────────────────────────────────
                    INLINE COMMENTS (<count>)
───────────────────────────────────────────────────────────────

1. [<SEVERITY>] <file>:<line>
   <comment body preview - first 100 chars>

2. [<SEVERITY>] <file>:<line>
   <comment body preview>

...

───────────────────────────────────────────────────────────────
                    SUMMARY
───────────────────────────────────────────────────────────────

✅ Good: <count points>
🔴 Errors: <count>
🟡 Warnings: <count>
🔵 Info: <count>

Verdict: <APPROVE/REQUEST_CHANGES/COMMENT>

═══════════════════════════════════════════════════════════════
```

### Step 7: ASK USER FOR CONFIRMATION (ONLY ONCE)

**Use the AskUserQuestion tool** to ask user:

```
Ready to post this review to GitHub?

Options:
- Post Review (post all comments)
- Post Summary Only (no inline comments)
- Cancel (don't post anything)
```

### Step 8: Post Review (ONLY AFTER USER CONFIRMS)

If user confirms, post using GitHub API:

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
      "body": "<COMMENT_BODY>"
    }
  ]
}
EOF
```

**Event Types:**
- `"COMMENT"` - Neutral review
- `"APPROVE"` - Approve PR (cannot approve own PR)
- `"REQUEST_CHANGES"` - Request changes (use for errors)

### Step 9: Report Result

After posting, show:
- ✅ Review posted successfully
- Link to the review on GitHub
- Summary of what was posted

---

## Summary Format

```markdown
## 🔍 Code Review

**PR:** #<number> - <title>
**Branch:** `<head>` → `<base>`
**Changes:** +<additions> / -<deletions> lines across <file_count> files

### 📋 Summary
<1-2 sentence description>

### 📏 Rules Applied
- Project config: `.claude-review.yml` ✓/✗
- CLAUDE.md conventions: ✓/✗
- Default rules: ✓

### ✅ What's Good
- <Positive point 1>
- <Positive point 2>

### 🔍 Review Details
<N> inline comment(s) added.

| Severity | Count |
|----------|-------|
| 🔴 Error | X |
| 🟡 Warning | X |
| 🔵 Info | X |

### 📊 Verdict: <APPROVE ✅ | REQUEST_CHANGES 🔄 | COMMENT 💬>

---
*🤖 Reviewed by [Claude Code](https://claude.ai/code)*
```

## Comment Format

```
<EMOJI> **[<rule-id>]** <Title>

<Description of the issue>

**Fix:**
\`\`\`<language>
<code example>
\`\`\`
```

## Severity Mapping

| Severity | Emoji | Event |
|----------|-------|-------|
| Error | 🔴 | REQUEST_CHANGES |
| Warning | 🟡 | COMMENT |
| Info | 🔵 | COMMENT |
| Bug | 🐛 | REQUEST_CHANGES |
| Security | ⚠️ | REQUEST_CHANGES |
| Performance | ⚡ | COMMENT |
| Suggestion | 💡 | COMMENT |
| Good | ✅ | - |

## Error Handling

- Config not found → Use default rules only
- PR URL invalid → Show correct format
- `gh` not authenticated → Instruct to run `gh auth login`
- PR merged/closed → Inform user
- No issues found → Post approval with positive feedback

## Key Principle

**ASK USER ONLY ONCE** - at the end, before posting. All analysis should happen silently without user interaction.
