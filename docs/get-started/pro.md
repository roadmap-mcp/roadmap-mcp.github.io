# Pro

!!! note "Version 0.2.0-SNAPSHOT"
    Pro is available as a preview build, `0.2.0-SNAPSHOT`. Snapshots are published
    often until the 0.2.0 release, and the download link below always gives you the
    latest one.

Pro is the [Knowledge Base](../knowledge-base/index.md) and the sources that feed
it: your [Git repositories](git.md), Confluence and Jira, and Bitbucket. It also
has the AI providers, the jobs and scheduler that run Knowledge Base flows, and an
MCP server through which your AI client reads and writes the Knowledge Base. Trello is an optional extra: install it from the registry (see
[Plugins](plugins.md#managing-plugins)).

## Requirements

- **Python 3.13 or later on your PATH.** On macOS and Linux the launcher looks for
  `python3.14`, `python3.13` and `python3`, in that order; on Windows for
  `py -3.14`, `py -3.13` and `python`. It uses the first that is 3.13 or later, and
  stops with a message if there is none.
- **Internet access on the first run**, to install the dependencies from PyPI.
- **Git on your PATH**, for the [Git features](git.md).
- **unzip**, or any other way to extract a zip file, and **curl** if you download
  from the command line.

### Getting Python 3.13

=== "Windows"

    Install Python 3.13 from [python.org](https://www.python.org/downloads/windows/)
    and tick **Add python.exe to PATH** in the installer. Or, from a terminal:

    ```bat
    winget install Python.Python.3.13
    ```

    Open a new terminal and check:

    ```bat
    python --version
    ```

    If `python` opens the Microsoft Store instead, Python is not on your PATH yet:
    run the installer again with **Add python.exe to PATH** ticked.

=== "macOS"

    Install Python from [python.org](https://www.python.org/downloads/macos/), or
    with Homebrew:

    ```bash
    brew install python
    ```

    Check that `python3` is 3.13 or later:

    ```bash
    python3 --version
    ```

=== "Ubuntu 24.04"

    Ubuntu 24.04 ships Python 3.12, which is too old; the bare `ubuntu:24.04`
    container image has no Python at all. Install 3.13 from the deadsnakes PPA,
    along with git, curl and unzip. `python3.13-venv` is required:

    ```bash
    export DEBIAN_FRONTEND=noninteractive
    sudo -E apt-get update
    sudo -E apt-get install -y software-properties-common
    sudo -E add-apt-repository -y ppa:deadsnakes/ppa
    sudo -E apt-get install -y python3.13 python3.13-venv git curl unzip
    ```

    `DEBIAN_FRONTEND=noninteractive` stops the installation from halting to ask
    for your time zone. In a container where you are root, leave out `sudo -E`.
    Check:

    ```bash
    python3.13 --version
    ```

    If your system has a `python3` at 3.12, leave it: Ubuntu itself depends on it,
    and the launcher finds `python3.13` by itself.

## Download

[Download Pro (.zip)](https://raw.githubusercontent.com/roadmap-mcp/roadmap-dist/main/install/roadmap-pro.zip){ .md-button .md-button--primary }
[All downloads on GitHub](https://github.com/roadmap-mcp/roadmap-dist){ .md-button }

## Install and start

1. Unzip the bundle into a folder of its own, anywhere on your machine. The zip has
   no folder inside it, so extracting it in place scatters its files. On the
   command line:

    ```bash
    curl -LO https://raw.githubusercontent.com/roadmap-mcp/roadmap-dist/main/install/roadmap-pro.zip
    unzip roadmap-pro.zip -d roadmap-pro
    cd roadmap-pro
    ```

2. Start it:

    === "Windows"

        Double-click `start.bat`, or run it from a terminal.

    === "macOS / Linux"

        ```bash
        bash start.sh
        ```

3. Open <http://localhost:8090>.
4. Create the root user. The first visit shows **Create the root user**: keep the
   user name `root` or type another, choose a password of at least 8 characters,
   enter it again under **Confirm password**, and click **Create and sign in**. The
   root user is the admin who adds everyone else (see
   [Users and roles](users.md)).
5. Choose the encryption password. Next, an *Unlock Configuration* dialog asks for
   one under *First-time setup*, as **Encryption password**; enter it twice to
   confirm it. Roadmap encrypts the
   secrets you store with it, such as AI keys and access tokens. It is not the root
   user's password, and it can't be recovered: keep it safe. An admin enters it
   again after every restart, unless you set `ROADMAP_KEY` (see
   [Environment variables](#environment-variables)).

From then on, every visit starts at the sign-in screen.

The first run creates a `.venv` folder next to the launcher and installs the bundled
plugins into it, plus their dependencies from PyPI. That takes a few minutes and
needs internet access. Later runs reuse it and start at once. Stop the app with
Ctrl+C in its terminal.

!!! info "In a container, or on another machine"
    Roadmap only listens on `127.0.0.1` by default, so it can't be reached from
    outside a container or from another machine. Start it with
    `ROADMAP_HOST=0.0.0.0` (see [Environment variables](#environment-variables)), and
    publish the port when you run the container, for example
    `docker run -p 127.0.0.1:8090:8090 ...`. Read the warning there first. To let Roadmap read your Git repositories, mount the folder that
    holds them into the container, and set that path as the checkout folder on the
    server (see [Configuring Git](git.md#set-the-checkout-folders)).

To force a clean reinstall, for example after unzipping a newer snapshot over the
old one, delete `.venv/.installed` and start again.

## Where your data lives

Roadmap keeps its configuration in `config.yaml`, with the Knowledge Base and the rest of its
data next to it. Without further settings that is `backend/config.yaml` inside the
unzipped folder. Other workspaces get their own `workspaces/<name>/config.yaml`
beside it.

To keep your data outside the install folder, so a newer snapshot can replace the
folder without touching it, point `ROADMAP_CONFIG_PATH` at a `config.yaml` of your
choosing. Its folder is created if it doesn't exist. Use an absolute path: a
relative one is taken relative to the `backend` folder, because the launcher starts
Roadmap from there.

## Environment variables

| Variable | Default | What it does |
|---|---|---|
| `ROADMAP_CONFIG_PATH` | `backend/config.yaml` | The configuration file; data is kept next to it. |
| `ROADMAP_HOST` | `127.0.0.1` | The address to listen on. `0.0.0.0` makes Roadmap reachable from other machines, or from outside a container. |
| `ROADMAP_KB_PACK_REGISTRY_URL` | roadmap-dist's `kb-packs/index.json` | Where the app finds [Knowledge Base packs](../knowledge-base/packs.md). |
| `ROADMAP_KEY` | none | Your password. Secrets in the configuration are encrypted with it; set it to unlock them at startup instead of in the browser. |

Set them in the terminal before you start the launcher:

=== "Windows"

    ```bat
    set ROADMAP_CONFIG_PATH=D:\roadmap-data\config.yaml
    set ROADMAP_HOST=0.0.0.0
    start.bat
    ```

=== "macOS / Linux"

    ```bash
    export ROADMAP_CONFIG_PATH=~/roadmap-data/config.yaml
    export ROADMAP_HOST=0.0.0.0
    bash start.sh
    ```

!!! danger "Before you open Roadmap to other machines"
    Everyone signs in (see [Users and roles](users.md)), but Roadmap serves plain
    HTTP: passwords, sessions and API tokens cross the network unencrypted. Once the
    configuration is unlocked, an admin's settings page also shows every stored
    secret in readable form: AI provider keys, Atlassian tokens, Git credentials.

    Keep the default, `127.0.0.1`, unless you need otherwise. Only use
    `ROADMAP_HOST=0.0.0.0` inside a container whose port you publish to your own
    machine only (`-p 127.0.0.1:8090:8090`), on a network you trust, or behind a
    reverse proxy that adds HTTPS.

## Connect an AI client

Roadmap serves its tools over MCP, using SSE, at
`http://127.0.0.1:8090/mcp/sse`. A client needs an **API token** of a user with the
editor role or higher:

1. Under *Settings → System → Account*, give a new token a name and click **New
   token** (see [API tokens](account.md#api-tokens)).
2. Copy the token: it is shown once. Next to it is the Claude Code command below,
   with the token filled in.
3. Add Roadmap to your client with the token in place of `<token>`.

=== "Claude Code"

    ```bash
    claude mcp add-json -s user roadmap '{"type":"sse","url":"http://127.0.0.1:8090/mcp/sse","headers":{"Authorization":"Bearer <token>"}}'
    ```

    `-s user` makes Roadmap available in every project. Check it with
    `claude mcp list`, which should show `roadmap` as *Connected*. If you put the
    server in a project's `.mcp.json` instead, Claude Code shows it as *Pending
    approval* until you start `claude` in that project and approve it.

=== "Other clients"

    Add it to the client's MCP configuration file; where that file lives depends
    on the client:

    ```json
    {
      "mcpServers": {
        "roadmap": {
          "type": "sse",
          "url": "http://127.0.0.1:8090/mcp/sse",
          "headers": { "Authorization": "Bearer <token>" }
        }
      }
    }
    ```

!!! warning "Failed to connect?"
    Claude Code shows this, for SSE and for HTTP alike, when the configuration has
    no token or a wrong one, for example one that was revoked:

    ```text
    ✘ Failed to connect — Dynamic Client Registration rejected (HTTP 404): {"detail":"Not Found: Roadmap takes API tokens (Authorization: Bearer), not OAuth"}
    ```

    This is always the case for a client set up before snapshot 12, which had no
    tokens. Remove the client, make a token, and add it again with the `headers`
    part: `claude mcp remove roadmap -s user`, then the command above. A
    `403 forbidden` means the token's user is a viewer: an admin has to make them an
    editor.

Ask the client to run `kb_read` with `action="status"` to check that it works; the
[Pro edition](../knowledge-base/index.md) pages describe what it can do from there.

### The stateless endpoint

Roadmap also serves MCP at `http://127.0.0.1:8090/mcp`, over plain HTTP: every call
is one request, so it survives a restart of Roadmap without reconnecting. It offers a
single tool, `roadmap`, that runs any of the tools by name: call it with a
`command` and its `args`, such as `{"command": "kb_read", "args": {"action": "status"}}`,
and `{"command": "help"}` lists them all. Use it when your client
keeps losing the SSE connection, or when you'd rather not have every tool in the
client's tool list.

=== "Claude Code"

    ```bash
    claude mcp add-json -s user roadmap '{"type":"http","url":"http://127.0.0.1:8090/mcp","headers":{"Authorization":"Bearer <token>"}}'
    ```

=== "Other clients"

    ```json
    {
      "mcpServers": {
        "roadmap": {
          "type": "http",
          "url": "http://127.0.0.1:8090/mcp",
          "headers": { "Authorization": "Bearer <token>" }
        }
      }
    }
    ```

Add one of the two endpoints, not both.

## Next

- [Add users](users.md#managing-users), and give each one a role.

- [Configure Git](git.md): set the checkout folders and add your repositories, so the
  Knowledge Base can read them.
- [Set up personas and node types](../knowledge-base/setup.md), or install a
  [pack](../knowledge-base/packs.md).
