# Pro: download and install

!!! note "Version 0.2.0-SNAPSHOT"
    Pro is available as a preview build, `0.2.0-SNAPSHOT`. Snapshots are published
    often until the 0.2.0 release, and the download link below always gives you the
    latest one.

Pro is the whole of Roadmap: contexts, the [Knowledge Base](../knowledge-base/index.md)
and its flows, git mining with jobs and a scheduler, Jira and Confluence, Bitbucket,
Trello, the AI providers, workflows, web testing, mockups, the file viewers and
speech to text.

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
4. Choose a password. On the first visit, an *Unlock Configuration* dialog asks for
   one under *First-time setup*. Roadmap encrypts the secrets you store with it,
   such as AI keys and access tokens. Keep it safe: you enter it again after every
   restart, unless you set `ROADMAP_KEY` (see
   [Environment variables](#environment-variables)).

The first run creates a `.venv` folder next to the launcher and installs the bundled
plugins into it, plus their dependencies from PyPI. That takes a few minutes and
needs internet access. Later runs reuse it and start at once. Stop the app with
Ctrl+C in its terminal.

!!! info "In a container, or on another machine"
    Roadmap only listens on `127.0.0.1` by default, so it can't be reached from
    outside a container or from another machine. Start it with
    `ROADMAP_HOST=0.0.0.0` (see [Environment variables](#environment-variables)), and
    publish the port when you run the container, for example
    `docker run -p 127.0.0.1:8090:8090 ...`. Read the warning there first: Roadmap
    has no login.

To force a clean reinstall, for example after unzipping a newer snapshot over the
old one, delete `.venv/.installed` and start again.

## Where your data lives

Roadmap keeps its configuration in `config.yaml`, with contexts and the rest of its
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

!!! danger "There is no login"
    Anyone who can reach port 8090 can use Roadmap and change its settings. Once
    the configuration is unlocked, the settings page also shows every stored secret
    in readable form: AI provider keys, Atlassian tokens, Git credentials.

    Keep the default, `127.0.0.1`, unless you need otherwise. Only use
    `ROADMAP_HOST=0.0.0.0` inside a container whose port you publish to your own
    machine only (`-p 127.0.0.1:8090:8090`), or on a network where you trust
    everyone who can reach it.

## Connect an AI client

Roadmap serves its tools over MCP, using SSE, at
`http://127.0.0.1:8090/mcp/sse`.

=== "Claude Code"

    ```bash
    claude mcp add-json -s user roadmap '{"type":"sse","url":"http://127.0.0.1:8090/mcp/sse"}'
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
        "roadmap": { "type": "sse", "url": "http://127.0.0.1:8090/mcp/sse" }
      }
    }
    ```

The [MCP tools](../reference/mcp-tools.md) page lists what the client can then do.

## Plugins

A fresh install has every Pro plugin installed and enabled. More plugins install
from the registry under *Settings → System → Admin*; restart Roadmap afterwards.

Which plugins a workspace uses is set under *Settings → System → Workspace*. Until
you save that list, every installed plugin is on, including ones you install later.
Once you have saved it, a newly installed plugin starts off in that workspace: turn
it on there.

Both screens work the same way:

- **Search** filters the plugins by name or id, and shows how many match.
- **List** and **Graph** switch between a list and a dependency graph. The graph
  asks before drawing more than 40 plugins, so narrow the search first.
- A plugin is marked **feature** when nothing depends on it, and **foundation** when
  it depends on nothing. Select one to see what it *depends on* and what it is
  *required by*.

Turning a plugin on in a workspace also turns on everything it depends on. Turning
one off also turns off everything that depends on it. A note names each plugin that
changed along with it.

## Next

- [Configure Git](git.md), so contexts and the Knowledge Base can read your
  repositories.
- [Configure the Knowledge Base](../reference/configuration.md).
