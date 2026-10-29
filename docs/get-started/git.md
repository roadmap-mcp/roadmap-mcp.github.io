# Configuring Git

Roadmap reads Git through **Git Repos**: working copies that are already on the
machine Roadmap runs on. Contexts, the MCP tools and Knowledge Base flows read them.
Roadmap never clones, pulls or changes them: keep them up to date yourself, the way
you normally would.

Git Repos are configured per workspace, under *Settings → Information Sources → Git
Repos*.

## Prerequisites

- **Git on the PATH of the Roadmap server.** Without it, file reads still work, but
  branches and commits don't, and Knowledge Base flows can't record which commit
  they read. Check with `git --version` in the terminal you start Roadmap from.

## Git Repos

*Settings → Information Sources → Git Repos* lists the working copies Roadmap may
read.

!!! important "Path is a folder on the server"
    **Path** is a local working copy on the machine the Roadmap server runs on, not
    on the machine your browser runs on, and not a remote URL. When Roadmap runs in a
    container, it is a path inside the container: mount your repositories into it.
    Roadmap reads the folder as it is on disk, including changes you haven't
    committed.

### Add a repository

1. Click **Add repo**.
2. Fill in:

    | Field | What it is |
    |---|---|
    | Name | How contexts, the MCP tools and flows refer to the repo. Leave it empty to use the folder name. Keep names unique: lookups use the first match. |
    | Path | The working copy's folder on the server, for example `/home/me/src/my-service` or `C:/src/my-service`. |
    | Tags | Optional labels to filter the list by. Type one and press Enter. |

3. Click **Save** under the fields. That writes the repository to `config.yaml`
   straight away; there is nothing else to save. **Remove** and **Import** are
   stored straight away too.

Roadmap doesn't check the path when you save it. If you mistype it, you'll find out
when something reads the repo.

### Import a folder of repositories

**Import from folder** asks for a parent folder and lists its subfolders, one level
deep. Folders with a `.git` are marked *git* and selected; folders already in the
list are marked *already added*. Each one you add is named after its folder, without
tags.

The **Browse** and **Import** folder pickers open a dialog on the machine the server
runs on. That works when Roadmap runs on your own desktop. On a server or in a
container, type the path into **Path** instead.

### In the configuration file

Git Repos are stored in the workspace's `config.yaml` (see
[where your data lives](pro.md#where-your-data-lives)):

```yaml
repositories:
  - name: my-service
    path: /home/me/src/my-service
    tags: [backend]
  - name: legacy-app
    path: C:/src/legacy-app
    tags: []
```

### What uses them

- **Contexts.** A *Git Repo* item adds a repository to a context; repo files and
  commits can be added from it too. Files are read from the working copy.
- **MCP tools.** `list_git_repos` lists the configured repositories.
  `request_git_repo_info` adds one to a context and returns its recent branches and
  commits. See [MCP tools](../reference/mcp-tools.md).
- **Knowledge Base flows.** A flow with a `repo` source names a Git Repo in its
  `ref`, and reads the files matching `scope.paths` from the working copy:

    ```yaml
    source: {kind: repo, ref: my-service, scope: {paths: ["src/**"]}}
    ```

    The `ref` must match a Git Repo's **Name** exactly. If it doesn't, the run fails
    with *No Git Repo called '...'*. See [Flows](../knowledge-base/flows.md).
