# Configuring Git

Roadmap works with Git in two separate ways:

- **Git Repos** are working copies that are already on the machine Roadmap runs on.
  Contexts, the MCP tools and Knowledge Base flows read them. Roadmap never clones,
  pulls or changes them.
- **Git Mining** clones remote repositories itself, keeps them up to date on a
  schedule, and turns their history into searchable work items: branches and
  commits, linked to the Jira issues their messages mention.

Both are configured per workspace, under *Settings → Information Sources*.

## Prerequisites

- **Git on the PATH of the Roadmap server.** Without it, file reads still work, but
  branches, commits, diffs and Git Mining don't. Check with `git --version` in the
  terminal you start Roadmap from.
- For Git Mining: network access from the server to your Git host.

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

## Git Mining

*Settings → Information Sources → Git Mining* configures the repositories Roadmap
mines. Click **Edit** on the tab to change it, **Done** when you're finished, then
**Save** on the settings page.

### Add a repository to mine

Under *Mining repositories*, click **Add repository** and fill in:

| Field | What it is |
|---|---|
| Name | A short name, for example `my-service`. Use letters, digits and dashes. Roadmap keeps its copy at `gitmining/repos/<name>.git` next to `config.yaml`. |
| Remote URL | The URL to clone, for example `https://git.example.com/team/my-service.git`. |
| Credential | *(public / none)*, or one of the credentials below. |
| Tracked branches | Comma-separated, for example `main, develop`. Empty means every branch. |
| Enabled | Only enabled repositories are scanned. |

### Credentials for private repositories

Under *Credentials*, click **Add credential**, give it a **Name**, and enter a
**Username** and a **Secret**: an access token or password. Then pick it as the
repository's *Credential*.

- Credentials only work with **HTTPS** remote URLs. SSH URLs are passed to Git as
  they are, without a credential.
- Secrets are only stored encrypted if you have set a password for Roadmap's
  encryption. Without one, they are stored in `config.yaml` as plain text.
- Use a token with read-only access.

### Scan

Click **Scan now** next to a repository to fetch it and mine new commits straight
away. The first scan clones the whole repository, which can take a while for a large
one.

To scan on a schedule, go to *Settings → System → Scheduler* and enable the task
*Scan git-mining repositories*. It is off by default. Its schedule is a standard
5-field cron expression in the server's local time; the default, `0 * * * *`,
scans every hour. Scheduled tasks only run while Roadmap is unlocked.

### The workbench

**Git Mining** in the main navigation lists the mined repositories. Open one to see:

- what has been mined: branches and commits, with author, subject line, changed
  files, and the Jira keys (such as `PROJ-123`) found in the subject;
- each item's details, processor results and relations to other items;
- **Scan now** to mine what's new, and **Full re-mine** to start over from scratch;
- the ingest jobs, with their status.

### In the configuration file

```yaml
gitmining:
  persistence: sql              # sql (built-in SQLite) | filesystem (JSON files)
  repos:
    - name: my-service
      remote_url: https://git.example.com/team/my-service.git
      branches: [main, develop] # [] = every branch
      credential: ci            # "" = public
      enabled: true
  credentials:
    - name: ci
      username: build-bot
      secret: ...               # encrypted when encryption is set up
scheduler:
  tasks:
    - id: gitmining.scan
      cron: "0 * * * *"
      enabled: true
```
