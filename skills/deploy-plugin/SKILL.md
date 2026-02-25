---
name: deploy-plugin
description: Deploys a Claude Code plugin to a GitHub marketplace. Triggers when the user wants to deploy, publish, release, or share a plugin to a marketplace. Responds to phrases like "플러그인 배포해줘", "마켓플레이스에 올려줘", "플러그인 공유하고 싶어", "deploy this plugin", "publish to marketplace".
---

# Plugin Deployer Skill

You help users deploy Claude Code plugins to GitHub-hosted marketplaces so anyone can install them.

## When to Activate

Activate when the user wants to:
- Deploy/publish/release a plugin to a marketplace
- Share a plugin on GitHub so others can install it
- Upload a plugin to a marketplace
- "배포", "퍼블리시", "마켓플레이스에 올려", "공유" related to plugins

## Workflow

### Step 1: Identify the Plugin

Determine which plugin to deploy:
- If the user points to a specific path, use that
- If they just created a plugin in this session, offer to deploy that one
- If unclear, ask which plugin they want to deploy

Validate the plugin:
1. Check `.claude-plugin/plugin.json` exists with name, description, version
2. Check component directories are at plugin root (not inside .claude-plugin/)
3. Show a summary of what will be deployed

### Step 2: Gather Deploy Config

Collect deployment info with sensible defaults. Ask once, not multiple times:

| Setting | Default | How to detect |
|---------|---------|---------------|
| GitHub username | Auto-detect via `gh api user --jq '.login'` | Fall back to asking |
| Repo name | Same as plugin name | Confirm with user |
| Marketplace name | `<username>-plugins` | Confirm with user |
| Visibility | public | Confirm with user |
| License | MIT | Confirm with user |

Present all defaults at once: "이대로 진행할까요?" — one confirmation, not five separate questions.

### Step 3: Build Marketplace Structure

Create the marketplace repo structure:

```
<repo-name>/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace catalog
├── plugins/
│   └── <plugin-name>/        # Copy of the plugin
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       ├── skills/
│       └── ...
└── README.md                 # Install instructions
```

**marketplace.json:**
```json
{
  "name": "<marketplace-name>",
  "owner": { "name": "<github-username>" },
  "metadata": {
    "description": "...",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "<plugin-name>",
      "source": "./<plugin-name>",
      "description": "<from plugin.json>",
      "version": "<from plugin.json>"
    }
  ]
}
```

**README.md** must include:
- Plugin name and description
- Install instructions with exact commands
- Usage examples
- License info

### Step 4: Deploy to GitHub

Execute in order:
1. `gh auth status` — check auth, guide login if needed
2. `git init` + initial commit in the marketplace directory
3. `gh repo create <repo> --public --source=. --push` (or --private)
4. Show the result URL

### Step 5: Show Results

```
✅ 배포 완료!

📦 레포: https://github.com/<username>/<repo>

🔧 설치 방법:
  /plugin marketplace add <username>/<repo>
  /plugin install <plugin-name>@<marketplace-name>

📝 업데이트 방법:
  plugins/<plugin-name>/ 수정 → git commit → git push
  사용자: /plugin marketplace update
```

## Handling Existing Marketplaces

If the user already has a marketplace repo:
1. Clone or navigate to the existing repo
2. Copy the new plugin into `plugins/`
3. Add a new entry to `marketplace.json` plugins array
4. Commit and push
5. Do NOT overwrite existing plugins or marketplace metadata

## Prerequisites Check

Before starting deployment, verify:
- `gh` CLI is installed → if not, guide: `brew install gh`
- `gh auth status` passes → if not, guide: `gh auth login`
- `git` is available and configured → if not, help set up `git config`

If any prerequisite fails, stop and help the user fix it before proceeding.

## Safety Rules

- NEVER force push
- NEVER overwrite existing repos without asking
- Warn about sensitive files (.env, credentials, tokens)
- Always show what will be created/pushed before doing it
- If the repo name already exists on GitHub, ask before proceeding
