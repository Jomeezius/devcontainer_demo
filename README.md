# devcontainer_demo

A minimal Dev Container blueprint for Python projects, with [OpenCode](https://opencode.ai) pre-installed and wired to a self-hosted, OpenAI-compatible LLM endpoint (vLLM / NVIDIA NIM).

This repository contains no application code. Copy `.devcontainer/` into your own project and adapt it.

## Contents

```
.devcontainer/
├── Containerfile.dev     # Image definition (Ubuntu 26.04 "Resolute")
├── devcontainer.json     # Dev Container definition
└── opencode.json         # OpenCode config (model + custom provider)
```

The built image includes Python 3 (`pip`, `pipx`), Node.js 20, a current Git from `ppa:git-core/ppa`, `build-essential`, common CLI tools (`curl`, `jq`, `vim`, `htop`, `rsync`, `unzip`), the OpenCode CLI, and a non-root user `coder` (UID/GID 3001) with passwordless `sudo`. Workspace root is `/workspaces`.

## Prerequisites

- Docker
- VS Code with the [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension, or the [devcontainer CLI](https://github.com/devcontainers/cli)

## Quick start

```bash
git clone https://github.com/Jomeezius/devcontainer_demo.git
cd devcontainer_demo
printf 'API_KEY=your-api-key\n' > .env      # required, see below
```

Then either open the folder in VS Code and run **Dev Containers: Reopen in Container**, or:

```bash
devcontainer up --workspace-folder .
devcontainer exec --workspace-folder . bash
```

### The `.env` file is mandatory

`devcontainer.json` passes `--env-file .env` straight to `docker run`. Without that file in the repository root, the container fails to start:

```
docker: open .env: no such file or directory
```

It must define `API_KEY`, which `opencode.json` reads via `{env:API_KEY}`. There is no `.gitignore` in this repository yet — add one before committing:

```bash
printf '.env\n' >> .gitignore
```

## Configuration

**`devcontainer.json`** — builds from `Containerfile.dev`, connects as `remoteUser: coder`, installs the `code-spell-checker` extension, and publishes ports via `appPort: ["8080:8080", "4000:3000"]` (host 8080 → container 8080, host 4000 → container 3000). `features`, `forwardPorts`, `postCreateCommand` and a `kubectl port-forward` for MLflow are present but commented out.

**`Containerfile.dev`** — Ubuntu base (`UBUNTU_VERSION=resolute`), system packages, Node.js 20 from NodeSource, `en_US.UTF-8` locale, then it deletes the default `ubuntu` user and creates `coder`. Everything after `USER coder` runs unprivileged, including the OpenCode install. Overridable build args: `UBUNTU_VERSION`, `USERNAME`, `USER_UID`, `USER_GID`.

**`opencode.json`** — disables the built-in OpenCode provider and defines a custom `DEV-NIM` provider via `@ai-sdk/openai-compatible`, pointing at `http://192.168.34.122:8000/v1` with model `nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4` (128k context, 8k output). `autoupdate` is off.

## OpenCode setup

The `COPY opencode.json …` line in `Containerfile.dev` is commented out, so the config does not end up in the image. Either uncomment it (the build context is `.devcontainer/`, so the path works as written), or copy it after start:

```bash
cp .devcontainer/opencode.json ~/.config/opencode/opencode.json
opencode
```

## Known gotchas

| Issue | Impact | Fix |
|---|---|---|
| No `.env` in repo, but `--env-file .env` in `runArgs` | Container won't start | Create `.env` |
| No `.gitignore` | Secrets can be committed | Add `.env` to `.gitignore` |
| `COPY opencode.json` commented out | OpenCode starts without provider config | Uncomment or copy manually |
| `ARG OPENCODE_VERSION=1.18.4` is never used | Install script always pulls latest; builds aren't reproducible | Use the arg or drop it |
| `ubuntu:resolute` tag, not pinned by digest | Builds drift over time | Pin `@sha256:…` |
| `baseURL` points at `192.168.34.122` | Only reachable on that internal network | Make it configurable via env var |
| `USER_UID=3001` vs. typical host UID 1000 | Bind-mount file ownership issues on Linux | Pass `USER_UID`/`USER_GID` as build args |
| `appPort` instead of `forwardPorts` | Ignored in Codespaces; host ports can collide | Switch to `forwardPorts` |
| `systemd` installed but no init process | Dead weight in the image | Remove, or run the container with a real init |
| Port `9000` has `portsAttributes` but isn't forwarded | Label never applies | Enable `forwardPorts: [9000]` |
| No `LICENSE` | Not legally reusable by others | Add MIT / Apache-2.0 |

## Troubleshooting

- **`open .env: no such file or directory`** — the file belongs in the repository root, not in `.devcontainer/`.
- **`address already in use`** — remap the host port in `appPort` (e.g. `"8081:8080"`) or switch to `forwardPorts`.
- **Build fails at the PPA or NodeSource step** — check proxy/firewall access to `launchpad.net` and `deb.nodesource.com`.
- **OpenCode connection or auth errors** — verify the config was copied, `API_KEY` is set, and the endpoint is reachable: `curl -v http://192.168.34.122:8000/v1/models`.
- **Container won't open in VS Code** — check **View → Output → Dev Containers** for the actual `docker` error.

## License

None specified. Without a `LICENSE` file, default copyright applies and the content is not licensed for reuse.

## Links

- [containers.dev](https://containers.dev) · [`devcontainer.json` reference](https://containers.dev/implementors/json_reference/) · [Features](https://containers.dev/features)
- [VS Code Dev Containers docs](https://code.visualstudio.com/docs/devcontainers/containers)
- [OpenCode docs](https://opencode.ai/docs/)
