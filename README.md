# ccc (Claude Code Chat)

A multi-platform bot that integrates Claude AI to help with software development tasks through chat commands. Supports both **Telegram** and **Lark (Feishu)**. This bot uses the Claude Agent SDK with `bypassPermissions` mode to perform complex development tasks like implementing features, fixing bugs, and more. Imagine you are having a software development team but they are not human. Give them instructions then review their work before merging their changes.

This project was fully written by Claude Code.

## Important Security Notice

**ALWAYS run this bot in an isolated environment such as:**

1. **Isolated Environment**: Always run in isolation
  - A dedicated VM
  - A Docker container
  - A separate development machine
  - A sandboxed environment
2. **DO NOT run this on:**
  - Your primary development machine
  - Production servers
  - Machines with sensitive data
  - Shared systems
3. **API Tokens**: Keep your Telegram/Lark tokens secret
4. **Authorization**: Regularly review authorized users and groups
5. **Repository Access**: Bot has full access to configured repositories
6. **File System**: Bot can modify files in project directories
7. **Command Execution**: Bot executes arbitrary commands via Claude

The bot uses the Claude Agent SDK with `bypassPermissions` mode, which allows it to make arbitrary file system changes, run commands, and potentially perform destructive operations.

## Development Status

This project is subject to changes as it is still in **active and heavy development**. Features, commands, and behavior may change without notice.

## Features

- **Multi-Platform Support**: Works with both Telegram and Lark (Feishu)
- **Parallel Job Execution**: Run multiple jobs on the same project simultaneously using git worktrees
- **Multiple Commands**: `/ask`, `/feat`, `/fix`, `/plan`, `/feedback`, `/init`, `/up`, `/stop`, `/status`, `/cancel`, `/log`, `/cost`, `/cleanup`, `/selfupdate`
- **Project Management**: Configure multiple projects via YAML
- **Session Continuity**: `/feedback` continues context from previous `/feat` or `/fix`
- **Thread Context**: Reply to bot in threads to continue conversations naturally
- **Process Management**: Spin up and stop project processes
- **Authorization**: User and group-based access control
- **Execution Tracking**: Automatic timing and logging
- **Git Integration**: Automatic repository cloning with worktree isolation
- **System Prompts**: Configurable rules per command type
- **MCP Integrations**: Connect external tools via MCP servers (e.g., Google Sheets)
- **Scheduled Reminders**: Configure automated prompts at specific times
- **Cost Monitoring**: View Claude API usage costs

## Prerequisites

- Python 3.10+
- Claude Code CLI installed and configured
- Telegram Bot API token (from @BotFather) and/or Lark App credentials
- Git (for repository cloning)
- claude-monitor (https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor) - optional, for /cost command
- The environment has to be configured to have network access and credentials ready (Claude Code, SSH for git, authentication for glab (gitlab), etc).

## Installation

### Option 1: Docker (Recommended)

Docker provides the isolated environment required for running this bot safely.

1. **Clone the repository:**
   ```bash
   git clone <your-repo-url>
   cd ccc
   ```

2. **Configure environment:**
   ```bash
   cp .env.example .env
   ```
   Edit `.env` and set your `TELEGRAM_BOT_TOKEN` (and Lark credentials if using Lark) and paths to SSH keys and Claude config.

3. **Configure projects:**
   Edit `config.yaml` to set up authorized users, groups, and projects.

4. **Build and run:**

   **Using Make (recommended):**
   ```bash
   make build
   make run
   make logs
   ```

   **Using Docker Compose:**
   ```bash
   docker-compose up -d
   docker-compose logs -f
   ```

5. **Manage the bot:**
   ```bash
   make stop       # Stop the bot
   make restart    # Restart the bot
   make logs       # View logs
   make status     # Check status
   ```

   Run `make help` to see all available commands.

6. **Stop the bot:**
   ```bash
   make down
   # or
   docker-compose down
   ```

**Building and pushing to Docker Hub:**
```bash
# Build the image
make build
# or
docker build -t herpiko/ccc:latest .

# Push to Docker Hub
make push
# or
docker push herpiko/ccc:latest
```

For detailed Docker setup instructions, troubleshooting, and advanced configuration, see [DOCKER.md](DOCKER.md).

### Option 2: Manual Installation

1. **Clone the repository:**
   ```bash
   git clone <your-repo-url>
   cd ccc
   ```

2. **Install the package:**
   ```bash
   pip install -e .
   ```

   Or install dependencies only:
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure config.yaml:**
   Edit `config.yaml` to set up:
   - Authorized users
   - Authorized groups/chats
   - Projects
   - Command rules
   - Platform-specific settings (Telegram/Lark)

## Configuration

### config.yaml Structure

```yaml
# Shared configuration - Unified user list for both Telegram and Lark
authorized_users:
  - username: "telegram_username"      # Required: Telegram username
    lark_ouid: "ou_xxx"                # Optional: Lark open_id (for Lark auth)
    name: "John Doe"                   # Optional: for commit attribution
    email: "john@example.com"          # Optional: for commit attribution
  - username: "another_user"           # Minimal entry (no Lark, no identity)
# Backward compatible: plain strings still supported:
# authorized_users: ["username1", "username2"]

# Base directory for git worktrees (isolated workspaces for concurrent queries)
# Default: /tmp/ccc-worktrees
worktree_base: /tmp/ccc-worktrees

general_rules: |
  - General rules for all commands

ask_rules: |
  - Rules for /ask command

feat_rules: |
  - Rules for /feat command

fix_rules: |
  - Rules for /fix command

plan_rules: |
  - Rules for /plan command

feedback_rules: |
  - Rules for /feedback command

# Telegram configuration
telegram:
  bot_token: "your_telegram_bot_token"
  authorized_groups:
    # Format with optional sub (thread_id) for topics/threads:
    - group: "-1234567890"
      sub: "12345"  # Optional: thread_id for topic/subgroup

# Lark configuration
lark:
  app_id: "cli_xxx"
  app_secret: "xxx"
  verification_token: "xxx"
  encrypt_key: ""  # Optional, for encrypted events
  webhook_port: 8080
  authorized_chats: ["oc_xxx"]  # Lark chat_ids

# MCP integrations (optional, global scope - applies to all projects)
integrations:
  # Known integration shorthand (e.g. gspreadsheet)
  - mcp: gspreadsheet
    service_account: "/path/to/service-account.json"
    drive_folder_id: "optional_folder_id"       # optional
  # Generic/custom MCP server (stdio)
  - mcp: my-custom-server
    command: "npx"
    args: ["-y", "@example/mcp-server"]
    env:
      API_KEY: "xxx"

# Scheduled reminders (optional)
reminders:
  - time: "11:00+7"        # HH:MM with UTC offset (+N or -N)
    prompt: "The prompt to send to Claude"
    repeat: ['monday', 'tuesday', 'wednesday', 'thursday', 'friday']

# Project configuration
projects:
  - project_name: "my-project"
    project_repo: "git@github.com:user/repo.git"
    project_workdir: "/path/to/workdir"
    project_up: "make run"      # Optional: command to spin up the project
    project_reset: "make purge" # Optional: command to reset project
    project_endpoint_url: "https://myapp.example.com"  # Optional: URL shown when project starts
    project_ports: ["3000", "8080"]  # Optional: ports to free up before starting
```

## Usage

### Start the Bot

**With Docker Compose (recommended):**
```bash
docker-compose up -d
```

**Manual execution (after pip install -e .):**
```bash
# Run both Telegram and Lark bots (if configured)
ccc

# Run only Telegram bot
ccc --telegram

# Run only Lark bot
ccc --lark

# With custom config path
ccc --config /path/to/config.yaml
```

**Or run as a module:**
```bash
python -m ccc
```

### Platform-Specific Setup

#### Telegram Setup

1. Create a bot via @BotFather and get the token
2. Disable Privacy Mode (see below)
3. Add bot to your group
4. Configure `telegram` section in config.yaml

#### Lark Setup

1. Create an app in Lark Developer Console
2. Enable "Bot" capability
3. Subscribe to "Message received" event (im.message.receive_v1)
4. Set webhook URL to `http://your-server:8080/webhook`
5. Configure `lark` section in config.yaml

### Available Commands

#### `/ask [project-name] <query>`
Ask questions. With project-name: about that project. Without: casual conversation.

**Examples:**
```
/ask Explain how async/await works in Python
/ask my-project Explain the authentication flow
```

#### `/feat <project-name> <task>`
Implement new features in a specific project. Creates a new branch and merge request.

**Example:**
```
/feat my-project Add user authentication with JWT
```

#### `/feedback [project-name] [job-id] <feedback>`
Continue work on an existing branch with feedback. Optionally specify a job ID to continue a specific job (useful for parallel jobs). Can also be used by replying in a thread with context.

**Examples:**
```
/feedback my-project Fix the validation on the login form
/feedback my-project abc12345 Add more unit tests
```

Use `/status` to see available job IDs.

#### `/fix <project-name> <issue>`
Fix bugs in a specific project.

**Example:**
```
/fix my-project Resolve the memory leak in the cache module
```

#### `/plan <project-name> <task>`
Plan and explore a task. Creates a new branch with `plan-` prefix. Good for design, exploration, and documentation before implementation.

**Example:**
```
/plan my-project Design the new authentication system architecture
```

#### `/init <project-name>`
Initialize CLAUDE.md for a project.

**Example:**
```
/init my-project
```

#### `/up <project-name> [branch]`
Spin up a project using the configured `project_up` command. Optionally specify a branch.

**Examples:**
```
/up my-project
/up my-project feat-123
```

#### `/stop [project-name]`
Stop a running project process. Without project-name: uses thread context.

**Example:**
```
/stop my-project
```

#### `/status`
Show all running project processes, Claude queries, and completed jobs with their details (query ID, command, elapsed time).

**Example:**
```
/status
```

#### `/cancel [project-name] [query-id]`
Cancel running Claude queries. If query-id is specified, cancels only that query. Otherwise cancels all queries for the project. Without args: uses thread context.

**Example:**
```
/cancel my-project           # Cancel all queries for project
/cancel my-project abc123    # Cancel specific query
```

#### `/log [project-name] [lines]`
Show the last N lines of a running project's logs (default: 50 lines). Without project-name: uses thread context.

**Example:**
```
/log my-project
/log my-project 100
```

#### `/cleanup`
Clean up orphan worktrees (those without running/completed jobs).

**Example:**
```
/cleanup
```

#### `/cost`
Display Claude API usage costs via claude-monitor.

**Example:**
```
/cost
```

#### `/selfupdate`
Update bot from GitHub and restart.

## How It Works

1. User sends a command in an authorized Telegram group or Lark chat
2. Bot validates user and chat authorization
3. Bot clones repository if needed (for project commands)
4. Bot creates an isolated git worktree for the query (enables parallel execution)
5. Bot executes Claude via the Agent SDK with:
   - `bypassPermissions` mode for full access
   - `system_prompt` with command-specific rules
   - MCP servers if configured
6. Bot tracks execution time and query status
7. Bot sends results back to the chat (in thread for Lark)
8. Thread context is maintained for follow-up conversations

## Parallel Job Execution

The bot supports running multiple jobs on the same project simultaneously using **git worktrees**. Each query gets its own isolated workspace cloned from the main repository, allowing concurrent development tasks without conflicts.

- Worktrees are created in the `worktree_base` directory (default: `/tmp/ccc-worktrees`)
- Worktrees are kept for potential feedback/continuation
- Use `/status` to see all running queries with their IDs
- Use `/cancel project-name query-id` to cancel a specific query
- Use `/cleanup` to remove orphan worktrees

## Thread Context

The bot maintains conversation context per thread:
- In Lark: Reply in the same thread to continue a conversation naturally
- Mention the bot without a command prefix to continue in existing context
- If no context exists, mentions are treated as `/ask`

## Authorization

The bot enforces **dual authorization**:
- User must be in `authorized_users` list (by `username` for Telegram, by `lark_ouid` for Lark)
- Chat must be in `authorized_groups` (Telegram) or `authorized_chats` (Lark)

Both conditions must be met for the bot to respond.

## Disable Privacy Mode (Telegram)

For the bot to work in Telegram groups, you must disable Privacy Mode:

1. Open @BotFather
2. Send `/mybots`
3. Select your bot
4. Go to **Bot Settings** -> **Group Privacy**
5. Click **Turn off**
6. Remove and re-add the bot to your group

## Troubleshooting

### Bot doesn't respond in group
- **Telegram**: Check that Privacy Mode is disabled in @BotFather
- **Lark**: Check webhook URL is accessible and event subscription is active
- Verify the chat ID is in authorized groups/chats
- Verify your username/user_id is in authorized users
- Check bot logs for authorization messages

### Unauthorized user message
- Check your username matches exactly (case-sensitive)
- Ensure you're in the correct group/chat
- Verify config.yaml is loaded correctly (check startup logs)

### Git clone fails
- Verify SSH keys are configured if using SSH URLs
- Ensure the bot has network access
- Check repository URL is correct

### Command times out
- Increase timeout if needed (currently 30 minutes)
- Check Claude Code is installed and accessible
- Verify the task isn't too complex

### Lark messages not threading
- Ensure `reply_in_thread: true` is working (check Lark SDK version)
- Verify webhook is receiving events correctly

## Contributing

This project is in active development. Please test thoroughly before contributing and document any changes.

## License

MIT

## Disclaimer

This bot executes AI-generated code with elevated permissions. Use at your own risk. Always review outputs and changes made by the bot. The authors are not responsible for any damage or data loss caused by using this bot.

---

**Built with Claude Code**
