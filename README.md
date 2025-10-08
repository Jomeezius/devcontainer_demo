# Demo DevContainer — Portable Development Environments

This repository provides a small, example DevContainer configuration designed to be easy to reuse across different devcontainer solutions: Visual Studio Code Remote - Containers, GitHub Codespaces, Gitpod, and local Docker / Docker Compose workflows. The goal is to offer a friendly, documented starting point you can adapt to your project and team.

Why this repo exists
- Demonstrate a simple, reproducible developer environment.
- Show compatibility patterns for multiple devcontainer providers.
- Provide clear, quick-start instructions so contributors can start coding quickly.

Contents
- devcontainer.json — VS Code / Codespaces devcontainer definition (may reference a Dockerfile or image).
- Dockerfile — (optional) base image built for the devcontainer.
- docker-compose.yml — (optional) brings up multi-service environments.
- .gitpod.yml — (optional) Gitpod configuration.
- README.md — this file.

Supported providers / workflows
- Visual Studio Code Remote - Containers (local)
- GitHub Codespaces
- Gitpod
- Docker CLI (build & run)
- Docker Compose

Quick start (pick one)

1) VS Code (Remote - Containers)
- Prerequisites: Docker, VS Code, Remote - Containers extension.
- From VS Code: Open the repository folder, then use "Remote-Containers: Reopen in Container".
- VS Code will build (or pull) the container and open your workspace inside it.

2) GitHub Codespaces
- If your repository is enabled for Codespaces, create a new Codespace from the GitHub UI.
- Codespaces will read `devcontainer.json` and `Dockerfile` (or prebuilt image) to build the environment.

3) Gitpod
- If `/.gitpod.yml` exists, you can start a workspace by prefixing your repo URL with https://gitpod.io/# and opening the result in your browser.
- Example: https://gitpod.io/#https://github.com/owner/repo

4) Local Docker (simple image)
- Build the image (if a Dockerfile is provided):
  docker build -t demo-devcontainer -f Dockerfile .
- Run a container and mount your workspace:
  docker run --rm -it -v "$(pwd):/workspace" -w /workspace demo-devcontainer /bin/bash

5) Docker Compose (multi-service)
- Start services:
  docker-compose up --build
- Attach an interactive shell to the workspace container:
  docker-compose exec <service-name> /bin/bash

Customization
- devcontainer.json
  - edit settings, extensions, and forwardPorts to match your project.
  - choose between "image" (prebuilt) and "build" (Dockerfile) depending on CI/policy.
- Dockerfile
  - install language runtimes, linters, debugging tools, or system packages here.
  - keep image layers small and cache-friendly.
- docker-compose.yml
  - useful when your dev environment requires databases, cache, or other services.
  - make sure to map volumes for source code and persist data if needed.

Recommended devcontainer.json settings (examples)
- `"forwardPorts": [3000, 8000]` — common development ports
- `"extensions": ["ms-vscode.cpptools", "dbaeumer.vscode-eslint"]` — editor extensions to preinstall
- `"settings": { "terminal.integrated.shell.linux": "/bin/bash" }`

Common patterns and tips
- Use a non-root user inside the container for better parity with CI and to avoid accidental root-owned files.
- Put repeated setup (toolchain installs) in the Dockerfile to leverage layer caching.
- If building in Codespaces, consider publishing a prebuilt image to reduce Codespace startup time.
- Keep environment-specific secrets out of the repo. Use provider secrets (Codespaces secrets, Gitpod env, .env files not in repo).

Ports, volumes, and workspace
- Ports: document which ports the app uses and ensure they are forwarded in devcontainer.json / compose.
- Volumes: mount your source directory into the container (usually /workspace).
- Workspace folder: set `"workspaceFolder"` in devcontainer.json to the expected working directory (e.g., /workspace).

Security and best practices
- Avoid embedding secrets or long-lived tokens in images or config.
- Use minimal base images and only install required packages.
- Regularly update base images and dependencies to pick up security fixes.

Troubleshooting
- If builds fail: check Docker daemon status, network access (firewall/proxy), and available disk space.
- If VS Code cannot reopen in container: check the Remote - Containers extension logs (View → Output → Remote - Containers).
- If extensions fail to install: verify that the container has network access and the marketplace is reachable.

Contributing
- Suggest improvements via issues or pull requests.
- If you add a new devcontainer provider workflow (e.g., a prebuilt image or cloud workspace), update this README with usage notes.

License
- Add an appropriate LICENSE file for your project. If you don't include one, the repository content is not automatically licensed.

Where to go from here
- Replace the example Dockerfile and devcontainer.json with language, framework, and tooling specific to your project.
- Add CI jobs to build and test the devcontainer setup (optional but recommended for team projects).

Acknowledgements
- Inspired by the Dev Containers documentation from Visual Studio Code and common community patterns for Codespaces and Gitpod.

If you want, I can:
- Tailor this README to your repo by including specific file paths, port numbers, or sample commands found in your config files.
- Generate a CONTRIBUTING.md and a basic LICENSE file next.
