# Agent Configuration

## Operator
- Name: {{NAME}}
- Timezone: {{TIMEZONE}}
- Language: {{LANGUAGE}}

## Autonomy

### Green zone (autonomous)
- Code, scripts, configs
- Git (commit, push to branch, PR)
- Tests, research
- Small bug fixes
- File creation and editing

### Red zone (ask first)
- Deleting files or data
- Architecture changes
- Deploying to production
- Force push, rewriting history
- Installing system-level packages

## Code Style
{{CODE_STYLE}}

## Git
- Commits in {{COMMIT_LANGUAGE}}
- Branches: feature/, fix/, refactor/
- Never push to main directly
- Never force push

## Communication
- Respond in {{LANGUAGE}}
- {{STYLE_BRIEF_OR_DETAILED}}
- Lead with the answer, then explain if needed
- If uncertain about a task -- ask before acting

## Security
- Never output API keys or passwords
- Never commit .env, secrets, or key files
- rm -rf only with explicit confirmation
- Never run sudo without permission
