# Google Workspace MCP Server

MCP server for Google Workspace: Gmail, Calendar, Drive, Docs, Sheets, Slides, Forms, Tasks, Chat.

## Quick Reference

```bash
# Run locally
uv run main.py --tool-tier extended

# Tool tiers: basic, standard, extended
# Use extended for full functionality
```

## Project Structure

```
google_workspace_mcp/
├── auth/           # OAuth2 authentication
│   ├── google_auth.py   # Auth flow and token management
│   └── scopes.py        # Google API scope definitions
├── core/
│   └── tool_tiers.yaml  # Tool availability by tier
├── gmail/          # Gmail tools
├── calendar/       # Calendar tools
├── drive/          # Drive tools
├── docs/           # Docs tools
├── sheets/         # Sheets tools
└── main.py         # Entry point
```

## Authentication

Uses Google Desktop OAuth client (no redirect URIs needed):
1. Set `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`
2. First run opens browser for OAuth consent
3. Tokens cached in `~/.google_workspace_mcp/`

## Adding New Tools

1. Create tool function with `@server.tool()` decorator
2. Add scope to `auth/scopes.py` if new permission needed
3. Use `@require_google_service("service_name", SCOPE)` decorator
4. Add to appropriate tier in `core/tool_tiers.yaml`

Example:
```python
@server.tool()
@handle_http_errors("tool_name", is_read_only=True, service_type="gmail")
@require_google_service("gmail", GMAIL_READONLY_SCOPE)
async def my_tool(service, user_google_email: str) -> str:
    # service is the authenticated Google API client
    result = await asyncio.to_thread(
        service.users().messages().list(userId="me").execute
    )
    return format_result(result)
```

## Gmail Tools

Available Gmail operations:
- **Messages**: list, get, send, reply, forward, trash, batch operations
- **Labels**: list, create, modify, apply to messages
- **Filters**: list, get, create, delete (requires GMAIL_SETTINGS_BASIC_SCOPE)
- **Threads**: list, get, modify labels
- **Drafts**: create, update, send

## Scopes Reference

```python
GMAIL_READONLY_SCOPE = "gmail.readonly"
GMAIL_COMPOSE_SCOPE = "gmail.compose"
GMAIL_MODIFY_SCOPE = "gmail.modify"
GMAIL_LABELS_SCOPE = "gmail.labels"
GMAIL_SETTINGS_BASIC_SCOPE = "gmail.settings.basic"  # For filters
```

## Common Patterns

**Async execution of sync Google APIs:**
```python
result = await asyncio.to_thread(
    service.users().messages().get(userId="me", id=msg_id).execute
)
```

**Error handling:**
```python
@handle_http_errors("operation_name", is_read_only=True, service_type="gmail")
```
