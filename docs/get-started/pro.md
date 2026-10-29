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

- **Python 3.13 or later on your PATH.** The launcher calls `python` on Windows
  and `python3` on macOS and Linux, so that command must be 3.13 or later.
- **Internet access on the first run**, to install the dependencies from PyPI.
- **Git on your PATH**, for the [Git features](git.md).
- **unzip**, or any other way to extract a zip file.

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

    Ubuntu 24.04 ships Python 3.12, which is too old. Install 3.13 from the
    deadsnakes PPA, along with git and unzip:

    ```bash
    sudo apt-get update
    sudo apt-get install -y software-properties-common
    sudo add-apt-repository -y ppa:deadsnakes/ppa
    sudo apt-get install -y python3.13 python3.13-venv git unzip
    ```

    In a container where you are root, leave out `sudo`.

    This leaves the system `python3` at 3.12, which Ubuntu itself depends on, so
    don't replace it. Instead, give the launcher its own `python3` that points at
    3.13, ahead of the system one on your PATH:

    ```bash
    mkdir -p ~/.roadmap-bin
    ln -sf "$(command -v python3.13)" ~/.roadmap-bin/python3
    export PATH=~/.roadmap-bin:$PATH
    python3 --version
    ```

    The `export` lasts for this terminal only. Run it again in every new terminal
    before you start Roadmap, or add it to your `~/.bashrc`.

!!! tip "Started it with an older Python by mistake?"
    The first start fixes the Python version into the `.venv` folder it creates.
    Delete `.venv` and start again with 3.13 on your PATH.

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

The first run creates a `.venv` folder next to the launcher and installs the bundled
plugins into it, plus their dependencies from PyPI. That takes a few minutes and
needs internet access. Later runs reuse it and start at once. Stop the app with
Ctrl+C in its terminal.

!!! info "In a container, or on another machine"
    Roadmap only listens on `127.0.0.1` by default, so it can't be reached from
    outside a container or from another machine. Start it with
    `ROADMAP_HOST=0.0.0.0` (see [Environment variables](#environment-variables)), and
    publish the port when you run the container, for example
    `docker run -p 8090:8090 ...`.

To force a clean reinstall, for example after unzipping a newer snapshot over the
old one, delete `.venv/.installed` and start again.

## Where your data lives

Roadmap keeps its configuration in `config.yaml`, with contexts and the rest of its
data next to it. Without further settings that is `backend/config.yaml` inside the
unzipped folder. Other workspaces get their own `workspaces/<name>/config.yaml`
beside it.

To keep your data outside the install folder, so a newer snapshot can replace the
folder without touching it, point `ROADMAP_CONFIG_PATH` at a `config.yaml` of your
choosing, in a folder that exists.

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

!!! warning
    With `ROADMAP_HOST=0.0.0.0`, anyone who can reach the machine on port 8090 can
    use Roadmap. Only do this on a network you trust.

## Connect an AI client

Roadmap serves its tools over MCP, using SSE. Add it to your client's MCP
configuration:

```json
{
  "mcpServers": {
    "roadmap": { "type": "sse", "url": "http://127.0.0.1:8090/mcp/sse" }
  }
}
```

The [MCP tools](../reference/mcp-tools.md) page lists what the client can then do.

## Plugins

Every Pro plugin is installed and enabled out of the box. A plugin you install later,
from the registry under *Settings → Admin*, starts disabled. Enable it per workspace
under *Settings → System → Workspace*.

## Next

- [Configure Git](git.md), so contexts and the Knowledge Base can read your
  repositories.
- [Configure the Knowledge Base](../reference/configuration.md).
