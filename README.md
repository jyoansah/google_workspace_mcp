# Google Workspace MCP Server (Drapes Fork)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![PyPI](https://img.shields.io/pypi/v/workspace-mcp.svg)](https://pypi.org/project/workspace-mcp/)

Full natural language control over Google Calendar, Drive, Gmail, Docs, Sheets, Slides, Forms, Tasks, and Chat through MCP clients, AI assistants, and developer tools.

**Upstream:** [taylorwilsdon/google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp)
**This fork:** [drapesinc/google-mcp](https://github.com/drapesinc/google-mcp)

---

## Overview

A production-ready MCP server integrating all major Google Workspace services with AI assistants. Supports both single-user operation and multi-user authentication via OAuth 2.1. Built with FastMCP for optimal performance, featuring service caching, advanced authentication, and streamlined development patterns.

This fork adds a `list_authenticated_accounts` tool for discovering which Google accounts have cached credentials, and maintains the auth tools in a dedicated `auth` section in tool tiers.

---

## Features

| Service | Capabilities |
|---------|-------------|
| **Gmail** | Search, read, send, draft, threads, labels, filters (list/get/create/delete) |
| **Google Drive** | Search, read, create, share, permissions, batch sharing, ownership transfer |
| **Google Calendar** | List calendars, events CRUD, attachments, reminders |
| **Google Docs** | Create, read, edit, find/replace, tables, images, headers/footers, comments, PDF export |
| **Google Sheets** | Create, read, write, spreadsheet info, sheet management, comments |
| **Google Slides** | Create, read, batch updates, thumbnails, comments |
| **Google Forms** | Create, read, publish settings, responses |
| **Google Tasks** | Full task and task list CRUD, hierarchy, move, clear |
| **Google Chat** | Spaces, messages, search |
| **Google Custom Search** | Web search, site-restricted search |
| **Authentication** | Account discovery, manual auth flow, multi-account support |

---

## Quick Start

### Prerequisites

- **Python 3.10+**
- **[uv](https://github.com/astral-sh/uv)** (for running and dependency management)
- **Google Cloud Project** with OAuth 2.0 Desktop Application credentials

### 1. Clone and Run

```bash
git clone git@github.com:drapesinc/google-mcp.git
cd google-mcp
uv run main.py --tool-tier extended
```

### 2. Set Credentials

```bash
export GOOGLE_OAUTH_CLIENT_ID="your-client-id"
export GOOGLE_OAUTH_CLIENT_SECRET="your-secret"
export OAUTHLIB_INSECURE_TRANSPORT=1  # Development only
```

Or place a `client_secret.json` in the project root (downloaded from Google Cloud Console).

### 3. Enable Google APIs

In your Google Cloud project, enable the APIs you need:

- [Google Calendar API](https://console.cloud.google.com/flows/enableapi?apiid=calendar-json.googleapis.com)
- [Google Drive API](https://console.cloud.google.com/flows/enableapi?apiid=drive.googleapis.com)
- [Gmail API](https://console.cloud.google.com/flows/enableapi?apiid=gmail.googleapis.com)
- [Google Docs API](https://console.cloud.google.com/flows/enableapi?apiid=docs.googleapis.com)
- [Google Sheets API](https://console.cloud.google.com/flows/enableapi?apiid=sheets.googleapis.com)
- [Google Slides API](https://console.cloud.google.com/flows/enableapi?apiid=slides.googleapis.com)
- [Google Forms API](https://console.cloud.google.com/flows/enableapi?apiid=forms.googleapis.com)
- [Google Tasks API](https://console.cloud.google.com/flows/enableapi?apiid=tasks.googleapis.com)
- [Google Chat API](https://console.cloud.google.com/flows/enableapi?apiid=chat.googleapis.com)
- [Google Custom Search API](https://console.cloud.google.com/flows/enableapi?apiid=customsearch.googleapis.com)

---

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GOOGLE_OAUTH_CLIENT_ID` | Yes | OAuth client ID from Google Cloud |
| `GOOGLE_OAUTH_CLIENT_SECRET` | Yes | OAuth client secret |
| `OAUTHLIB_INSECURE_TRANSPORT` | Dev only | Set to `1` to allow HTTP redirect URIs |
| `USER_GOOGLE_EMAIL` | No | Default email for single-user auth |
| `GOOGLE_PSE_API_KEY` | No | API key for Custom Search |
| `GOOGLE_PSE_ENGINE_ID` | No | Search Engine ID for Custom Search |
| `GOOGLE_MCP_CREDENTIALS_DIR` | No | Custom credentials storage directory |
| `GOOGLE_CLIENT_SECRET_PATH` | No | Path to `client_secret.json` if not in project root |

### Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_MCP_BASE_URI` | `http://localhost` | Base server URI (no port) |
| `WORKSPACE_MCP_PORT` | `8000` | Server listening port |
| `WORKSPACE_EXTERNAL_URL` | None | External URL for reverse proxy setups |
| `GOOGLE_OAUTH_REDIRECT_URI` | Auto | Override OAuth callback URL |

### OAuth 2.1 Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_ENABLE_OAUTH21` | `false` | Enable OAuth 2.1 multi-user support |
| `EXTERNAL_OAUTH21_PROVIDER` | `false` | Use external OAuth flow with bearer tokens |
| `WORKSPACE_MCP_STATELESS_MODE` | `false` | No file system writes (container-friendly) |

### Credential Loading Priority

1. Environment variables (`export VAR=value`)
2. `.env` file in project root
3. `client_secret.json` via `GOOGLE_CLIENT_SECRET_PATH`
4. Default `client_secret.json` in project root

### Multi-Account Credentials

Credentials are stored per-user in `~/.google_workspace_mcp/credentials/` (or the path set by `GOOGLE_MCP_CREDENTIALS_DIR`). Use `list_authenticated_accounts` to discover which accounts are available.

---

## Launch Commands

```bash
# Default stdio mode (for Claude Desktop, Claude Code, etc.)
uv run main.py

# HTTP mode (for web interfaces and debugging)
uv run main.py --transport streamable-http

# Single-user mode (skip session mapping)
uv run main.py --single-user

# Load specific services only
uv run main.py --tools gmail drive calendar

# Tool tier selection
uv run main.py --tool-tier core       # Essential tools only
uv run main.py --tool-tier extended   # Core + management tools
uv run main.py --tool-tier complete   # Full API access

# Combine tier with service filter
uv run main.py --tools gmail drive --tool-tier core

# Via uvx (no clone needed)
uvx workspace-mcp --tool-tier core
```

### Docker

```bash
docker build -t workspace-mcp .
docker run -p 8000:8000 -v $(pwd):/app workspace-mcp --transport streamable-http
```

---

## Available Tools

All tools support automatic authentication via `@require_google_service()` decorators with 30-minute service caching.

### Auth

| Tool | Tier | Description |
|------|------|-------------|
| `list_authenticated_accounts` | Core | List all Google accounts with cached credentials |
| `start_google_auth` | Complete | Manually initiate OAuth authentication flow |

### Gmail

| Tool | Tier | Description |
|------|------|-------------|
| `search_gmail_messages` | Core | Search with Gmail query operators |
| `get_gmail_message_content` | Core | Retrieve message content |
| `get_gmail_messages_content_batch` | Core | Batch retrieve message content |
| `send_gmail_message` | Core | Send emails (supports attachments, Send As) |
| `get_gmail_thread_content` | Extended | Get full thread content |
| `modify_gmail_message_labels` | Extended | Modify message labels |
| `list_gmail_labels` | Extended | List available labels |
| `manage_gmail_label` | Extended | Create/update/delete labels |
| `draft_gmail_message` | Extended | Create drafts |
| `list_gmail_filters` | Extended | List Gmail filters |
| `get_gmail_filter` | Extended | Get filter details |
| `create_gmail_filter` | Extended | Create new filters |
| `delete_gmail_filter` | Extended | Delete filters |
| `get_gmail_threads_content_batch` | Complete | Batch retrieve thread content |
| `batch_modify_gmail_message_labels` | Complete | Batch modify labels |

### Google Drive

| Tool | Tier | Description |
|------|------|-------------|
| `search_drive_files` | Core | Search files with query syntax |
| `get_drive_file_content` | Core | Read file content (Office formats supported) |
| `get_drive_file_download_url` | Core | Get download URL for Drive files |
| `create_drive_file` | Core | Create files or fetch from URLs |
| `share_drive_file` | Core | Share with users/groups/domains/anyone |
| `get_drive_shareable_link` | Core | Get shareable links |
| `list_drive_items` | Extended | List folder contents |
| `update_drive_file` | Extended | Update metadata, move between folders |
| `batch_share_drive_file` | Extended | Share with multiple recipients |
| `update_drive_permission` | Extended | Modify permission role |
| `remove_drive_permission` | Extended | Revoke file access |
| `transfer_drive_ownership` | Extended | Transfer ownership |
| `get_drive_file_permissions` | Complete | Get detailed permissions |
| `check_drive_file_public_access` | Complete | Check public sharing status |

### Google Calendar

| Tool | Tier | Description |
|------|------|-------------|
| `list_calendars` | Core | List accessible calendars |
| `get_events` | Core | Retrieve events with time range filtering |
| `create_event` | Core | Create events with attachments and reminders |
| `modify_event` | Core | Update existing events |
| `delete_event` | Extended | Remove events |

### Google Docs

| Tool | Tier | Description |
|------|------|-------------|
| `get_doc_content` | Core | Extract document text |
| `create_doc` | Core | Create new documents |
| `modify_doc_text` | Core | Modify document text |
| `export_doc_to_pdf` | Extended | Export document to PDF |
| `search_docs` | Extended | Find documents by name |
| `find_and_replace_doc` | Extended | Find and replace text |
| `list_docs_in_folder` | Extended | List docs in folder |
| `insert_doc_elements` | Extended | Add tables, lists, page breaks |
| `insert_doc_image` | Complete | Insert images from Drive/URLs |
| `update_doc_headers_footers` | Complete | Modify headers and footers |
| `batch_update_doc` | Complete | Execute multiple operations |
| `inspect_doc_structure` | Complete | Analyze document structure |
| `create_table_with_data` | Complete | Create data tables |
| `debug_table_structure` | Complete | Debug table issues |
| `read_document_comments` | Complete | Read comments |
| `create_document_comment` | Complete | Create comments |
| `reply_to_document_comment` | Complete | Reply to comments |
| `resolve_document_comment` | Complete | Resolve comments |

### Google Sheets

| Tool | Tier | Description |
|------|------|-------------|
| `create_spreadsheet` | Core | Create new spreadsheets |
| `read_sheet_values` | Core | Read cell ranges |
| `modify_sheet_values` | Core | Write/update/clear cells |
| `list_spreadsheets` | Extended | List accessible spreadsheets |
| `get_spreadsheet_info` | Extended | Get spreadsheet metadata |
| `create_sheet` | Complete | Add sheets to existing files |
| `read_spreadsheet_comments` | Complete | Read comments |
| `create_spreadsheet_comment` | Complete | Create comments |
| `reply_to_spreadsheet_comment` | Complete | Reply to comments |
| `resolve_spreadsheet_comment` | Complete | Resolve comments |

### Google Slides

| Tool | Tier | Description |
|------|------|-------------|
| `create_presentation` | Core | Create new presentations |
| `get_presentation` | Core | Retrieve presentation details |
| `batch_update_presentation` | Extended | Apply multiple updates |
| `get_page` | Extended | Get specific slide info |
| `get_page_thumbnail` | Extended | Generate slide thumbnails |
| `read_presentation_comments` | Complete | Read comments |
| `create_presentation_comment` | Complete | Create comments |
| `reply_to_presentation_comment` | Complete | Reply to comments |
| `resolve_presentation_comment` | Complete | Resolve comments |

### Google Forms

| Tool | Tier | Description |
|------|------|-------------|
| `create_form` | Core | Create new forms |
| `get_form` | Core | Retrieve form details and URLs |
| `list_form_responses` | Extended | List all responses with pagination |
| `set_publish_settings` | Complete | Configure form settings |
| `get_form_response` | Complete | Get individual responses |

### Google Tasks

| Tool | Tier | Description |
|------|------|-------------|
| `list_tasks` | Core | List tasks with filtering |
| `get_task` | Core | Retrieve task details |
| `create_task` | Core | Create tasks with hierarchy |
| `update_task` | Core | Modify task properties |
| `delete_task` | Extended | Remove tasks |
| `list_task_lists` | Complete | List task lists |
| `get_task_list` | Complete | Get task list details |
| `create_task_list` | Complete | Create task lists |
| `update_task_list` | Complete | Update task lists |
| `delete_task_list` | Complete | Delete task lists |
| `move_task` | Complete | Reposition tasks |
| `clear_completed_tasks` | Complete | Hide completed tasks |

### Google Chat

| Tool | Tier | Description |
|------|------|-------------|
| `send_message` | Core | Send messages to spaces |
| `get_messages` | Core | Retrieve space messages |
| `search_messages` | Core | Search across chat history |
| `list_spaces` | Extended | List chat spaces/rooms |

### Google Custom Search

| Tool | Tier | Description |
|------|------|-------------|
| `search_custom` | Core | Perform web searches |
| `search_custom_siterestrict` | Extended | Search within specific domains |
| `get_search_engine_info` | Complete | Retrieve search engine metadata |

### Tool Tier Summary

- **Core** -- Essential tools for everyday tasks. Minimal API quotas. Start here.
- **Extended** -- Core plus management tools: labels, folders, filters, batch operations.
- **Complete** -- Full API access including comments, headers/footers, admin functions.

Tiers are cumulative: each includes all previous tiers.

---

## MCP Client Configuration

### Claude Desktop (stdio)

**Option 1: DXT installer** -- Download `google_workspace_mcp.dxt` from the Releases page and double-click to install.

**Option 2: Manual JSON config** -- Edit `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "google_workspace": {
      "command": "uvx",
      "args": ["workspace-mcp"],
      "env": {
        "GOOGLE_OAUTH_CLIENT_ID": "your-client-id",
        "GOOGLE_OAUTH_CLIENT_SECRET": "your-secret",
        "OAUTHLIB_INSECURE_TRANSPORT": "1"
      }
    }
  }
}
```

Config file locations:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

### Claude Code

```bash
claude mcp add --transport http workspace-mcp http://localhost:8000/mcp
```

### VS Code

```json
{
  "servers": {
    "google-workspace": {
      "url": "http://localhost:8000/mcp/",
      "type": "http"
    }
  }
}
```

### LM Studio

Same JSON format as Claude Desktop, added via Settings > MCP Servers.

### Development Setup (for contributors)

```json
{
  "mcpServers": {
    "google_workspace": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/path/to/google-mcp",
        "main.py"
      ],
      "env": {
        "GOOGLE_OAUTH_CLIENT_ID": "your-client-id",
        "GOOGLE_OAUTH_CLIENT_SECRET": "your-secret",
        "OAUTHLIB_INSECURE_TRANSPORT": "1"
      }
    }
  }
}
```

---

## Authentication

The server uses **Google Desktop OAuth** for simplified authentication:

1. Call any Google Workspace tool
2. Server returns an authorization URL
3. Open the URL in a browser and authorize
4. Google provides an authorization code
5. Paste the code when prompted (or handled automatically)
6. Server completes authentication and retries the request

Credentials are cached in `~/.google_workspace_mcp/credentials/` for reuse across sessions.

### OAuth 2.1 Multi-User Mode

For multi-user deployments, enable OAuth 2.1:

```bash
export MCP_ENABLE_OAUTH21=true
uv run main.py --transport streamable-http
```

OAuth 2.1 reuses your existing `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET`. No additional configuration needed. FastMCP's built-in `GoogleProvider` handles Dynamic Client Registration and CORS.

### Stateless Mode (Containers)

For container deployments where no filesystem writes should occur:

```bash
export MCP_ENABLE_OAUTH21=true
export WORKSPACE_MCP_STATELESS_MODE=true
uv run main.py --transport streamable-http
```

- No credentials written to disk
- No file-based logging
- Memory-only sessions via OAuth 2.1
- Each request must include a valid Bearer token

### External OAuth Provider Mode

For scenarios where authentication is handled by an external system:

```bash
export MCP_ENABLE_OAUTH21=true
export EXTERNAL_OAUTH21_PROVIDER=true
uv run main.py --transport streamable-http
```

- Protocol-level auth disabled (no auth needed for `initialize` / `tools/list`)
- Tool calls require `Authorization: Bearer <token>` header
- Tokens validated via Google's tokeninfo API

### OAuth Proxy Storage Backends

For OAuth 2.1, the server supports pluggable storage backends:

| Backend | Config | Persistence | Multi-Server |
|---------|--------|-------------|--------------|
| Memory | `WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=memory` | No | No |
| Disk | `WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=disk` | Yes | No |
| Valkey/Redis | `WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=valkey` | Yes | Yes |

Disk and Valkey backends are encrypted with Fernet. Install `workspace-mcp[valkey]` for Valkey support.

<details>
<summary>Valkey/Redis configuration variables</summary>

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_HOST` | localhost | Valkey/Redis host |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_PORT` | 6379 | Port (6380 auto-enables TLS) |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_DB` | 0 | Database number |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_USE_TLS` | auto | Enable TLS |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_USERNAME` | - | Auth username |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_PASSWORD` | - | Auth password |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_REQUEST_TIMEOUT_MS` | 5000 | Request timeout (remote) |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_CONNECTION_TIMEOUT_MS` | 10000 | Connection timeout (remote) |

</details>

### Reverse Proxy Setup

When behind nginx, Apache, Cloudflare, etc.:

```bash
# Set external URL for all OAuth endpoints
export WORKSPACE_EXTERNAL_URL="https://your-domain.com"

# Or override just the callback URL
export GOOGLE_OAUTH_REDIRECT_URI="https://your-domain.com/oauth2callback"
```

Additional options: `OAUTH_CUSTOM_REDIRECT_URIS` (comma-separated), `OAUTH_ALLOWED_ORIGINS` (comma-separated CORS origins).

---

## Project Structure

```
google_workspace_mcp/
├── auth/                  # Authentication system
│   ├── google_auth.py     # Auth flow and token management
│   ├── credential_store.py  # Multi-user credential storage
│   ├── scopes.py          # Google API scope definitions
│   ├── oauth_config.py    # Centralized OAuth configuration
│   └── ...
├── core/
│   ├── server.py          # MCP server, auth tools, health endpoint
│   ├── tool_tiers.yaml    # Tool availability by tier
│   ├── tool_tier_loader.py  # Tier resolution logic
│   ├── tool_registry.py   # Tool filtering and registration
│   └── ...
├── gcalendar/             # Google Calendar tools
├── gchat/                 # Google Chat tools
├── gdocs/                 # Google Docs tools
├── gdrive/                # Google Drive tools
├── gforms/                # Google Forms tools
├── gmail/                 # Gmail tools
├── gsearch/               # Google Custom Search tools
├── gsheets/               # Google Sheets tools
├── gslides/               # Google Slides tools
├── gtasks/                # Google Tasks tools
├── main.py                # Server entry point with CLI args
├── pyproject.toml         # Dependencies (v1.7.1)
└── CLAUDE.md              # Claude Code project context
```

## Development

### Setup

```bash
git clone git@github.com:drapesinc/google-mcp.git
cd google-mcp
uv sync --group dev
```

### Run Tests

```bash
uv run pytest
```

### Lint

```bash
uv run ruff check .
```

### Adding New Tools

```python
from auth.service_decorator import require_google_service

@require_google_service("drive", "drive_read")
async def your_new_tool(service, param1: str, param2: int = 10):
    """Tool description"""
    result = service.files().list().execute()
    return result
```

1. Create tool function with `@server.tool()` decorator
2. Add scope to `auth/scopes.py` if new permission needed
3. Use `@require_google_service("service_name", SCOPE)` decorator
4. Add to appropriate tier in `core/tool_tiers.yaml`

### Architecture Highlights

- **Service Caching**: 30-minute TTL reduces authentication overhead
- **Scope Management**: Centralized in `SCOPE_GROUPS` for easy maintenance
- **Error Handling**: `@handle_http_errors` decorator with retry logic
- **Multi-Service Support**: `@require_multiple_services()` for complex tools
- **Tool Tiers**: YAML-based tier configuration for progressive feature loading
- **Credential Store**: Abstract interface with local file storage backend

---

## Security

- Never commit `.env`, `client_secret.json`, or `.credentials/` to source control
- OAuth callback uses `http://localhost:8000/oauth2callback` for development (requires `OAUTHLIB_INSECURE_TRANSPORT=1`)
- Use HTTPS and OAuth 2.1 in production
- Tools request only necessary permissions (scope minimization)
- Fernet encryption for Disk and Valkey storage backends

---

## License

MIT License - see `LICENSE` file for details.
