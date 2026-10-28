# Tools

Drop tool directories here. Each tool needs:

```
tools/
  my-tool/
    pyproject.toml   # [tool.centaur] section with module path
                     # [project.scripts] entry for sandbox CLI shims
    .env.example     # Document required secrets
    __init__.py
    client.py        # API client class + _client() factory
    cli.py           # typer CLI for standalone use
```

## Writing a tool

```python
# client.py
from centaur_sdk.tool_sdk import secret


class MyClient:
    def search(self, query: str, limit: int = 10) -> dict:
        """Search something."""
        token = secret("MY_API_TOKEN")
        # ... use token, return results ...
        return {"results": [...]}


def _client() -> MyClient:
    return MyClient()
```

## Secrets

Use `secret("KEY")` to access a credential. In a sandbox it returns a
placeholder (the key name itself), and the per-sandbox iron-proxy swaps in the
real value on outbound requests to the hosts the secret is bound to. If the key
is set in the environment, its value is returned instead, which is how
`pg_dsn` secrets reach a tool.

Declare each secret, with its hosts and allowed request locations, under
`[tool.centaur] secrets` in `pyproject.toml`. Never read `os.environ` or load
`.env` files in tool code; ruff rejects both.

## Sandbox CLI shims

Agent sandboxes install tool CLIs from `[project.scripts]` at startup through
`services/sandbox/install_tool_shims.py`. To make a tool visible to agents,
declare a script and verify it appears in the sandbox catalog:

```toml
[project.scripts]
my-tool = "my_tool.cli:app"
```

```bash
centaur-tools list
my-tool --help
```

## Available Plugins

The open-source tool inventory lives in this `tools/` tree and changes over time. To see what ships in a running sandbox, use `centaur-tools list`; private deployments may mount additional overlay tool directories.

- `centaur-console`: inspect sandbox permissions and configured OAuth apps, and
  manage the linked user's scheduled tasks.
- `centaur-skills`: discover merged repository and overlay skills alongside
  private and public Console-authored skills, and manage Console skills.
- `datadog`: query Datadog logs, metrics, monitors, hosts, and dashboards
  read-only with `DD_API_KEY` and `DD_APP_KEY`.
- `preqin`: query Preqin Operational API fund and fund-manager data, with
  redacted auth diagnostics for `PREQIN_*` credentials.

## Sandbox Tool Paths

Sandbox startup accepts `TOOLS_PATH` and `TOOLS_OVERLAY_PATH` and appends them to
`TOOL_DIRS` for `centaur-tools` discovery. Pass them from the API process with
`SESSION_SANDBOX_PASSTHROUGH_ENV=TOOLS_PATH,TOOLS_OVERLAY_PATH`.

`TOOLS_PATH` should point at the base mounted tool directory. `TOOLS_OVERLAY_PATH`
is appended after it so overlay tools can replace base tools with the same name.

For repo-cache overlays, set `REPOS_PATH` on the API process to the host path
where the repo-cache syncs repositories. The sandbox mounts that path read-only
at `/home/agent/github`, so overlay tools can come from the cached repo:

```bash
REPOS_PATH=/var/lib/centaur/repos
TOOLS_PATH=/home/agent/github/paradigmxyz/centaur/tools
TOOLS_OVERLAY_PATH=/home/agent/github/acme/centaur-overlay/tools
SESSION_SANDBOX_PASSTHROUGH_ENV=TOOLS_PATH,TOOLS_OVERLAY_PATH
```

The repo-cache is responsible for syncing `acme/centaur-overlay`; sandbox
startup only points `centaur-tools` at the already-mounted source tree.
