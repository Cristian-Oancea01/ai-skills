# GitHub Workflows

## `opencode.yml`

Enables the [OpenCode GitHub integration](https://opencode.ai/docs/github) for this repository. It allows anyone with access to trigger OpenCode directly from GitHub issue or pull request comments.

### How it works

Mention `/opencode` or `/oc` in any issue or PR comment and OpenCode will:

- Read the full thread context
- Execute the requested task inside a GitHub Actions runner
- Commit changes, open PRs, or reply — all from within GitHub

### Trigger examples

```
/opencode fix this bug
/oc review this PR and suggest improvements
/oc explain why this test is failing
```

### Requirements

- **GitHub App**: Install [github.com/apps/opencode-agent](https://github.com/apps/opencode-agent) on this repository
- **Secret**: Add `ANTHROPIC_API_KEY` to the repository secrets (Settings → Secrets and variables → Actions)

### Supported events

| Event | Trigger |
|---|---|
| `issue_comment` | Comment on an issue or PR |
| `pull_request_review_comment` | Inline comment on a PR diff |
