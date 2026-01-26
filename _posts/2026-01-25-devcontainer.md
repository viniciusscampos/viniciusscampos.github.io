In the past we'd have the problem of artifact mismatch between development and production. Containerization mitigated this, because we could build a Docker image once and promote it from environment to environment. The code would run as expected regardless of the host, since it lived inside the container.

Even so, we still develop and run the solution locally on our main machines, and that brings issues like incompatibility. A global library version can conflict with the one required by the project. It also adds another attack surface, because we might run malicious code without noticing it if a dependency gets poisoned (supply chain attack).

There are solutions that help mitigate this, and one of them is using a [devcontainer](https://containers.dev/). With it, we not only create the final image of the project, but we keep the whole development cycle inside a container. That isolates it from our OS, avoids unwanted incompatibilities, and prevents it from accessing sensitive data on our machine, like credentials.

This matters even more now with AI coding assistants like [Codex](https://openai.com/index/codex/), [Claude Code](https://www.anthropic.com/claude-code), and [OpenCode](https://opencode.ai/). They could potentially access our root repositories (especially if we run in yolo mode) and create catastrophic scenarios, like deleting important files and folders.

As a simple example, we can create a devcontainer in the repository and add OpenCode to it, so the AI agent won't have access to anything outside the repository folder, since the mounted volume contains only the repository files. It's important to note that the Docker container still has access to the network, so if we want to make it 100% isolated we should restrict that as well, but that's not our case.

To configure it, create a `.devcontainer/` folder in the repository root directory and add the `devcontainer.json` and `Dockerfile` files to it.

The `devcontainer.json` specifies the name of the sandbox devcontainer environment, how it can be built, the user, and the workspaceFolder, which guarantees it will only have access to this mounted volume, not to all OS folders. Here's a minimal example.

```json
{
  "name": "opencode-sandbox-alpine",
  "build": { "dockerfile": "Dockerfile" },
  "remoteUser": "dev",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "runArgs": [
    "--cap-drop=ALL",
    "--security-opt=no-new-privileges",
    "--pids-limit=256"
  ]
}
```


The Dockerfile just represents the development environment, so it must install the necessary libraries to do development, including any additional AI coding agent like Claude Code, Codex, and OpenCode.

```Dockerfile
FROM alpine:3.20

# Basics + Node + npm
RUN apk add --no-cache bash ca-certificates git nodejs npm

# Install opencode (adjust package name if yours differs)
RUN npm i -g opencode-ai

# Non-root user
RUN adduser -D -u 1000 dev
USER dev

WORKDIR /workspaces
CMD ["bash"]
```


Next, install the devcontainers/cli in our system and use it to build and enter the development container as shown below.

```sh
npm i -g @devcontainers/cli
devcontainer up --workspace-folder .
devcontainer exec --workspace-folder . bash
```

Once it's running, you'll have a simple development environment where you can make all the changes safely inside a container that doesn't have access to other files, avoiding the risk of corrupting any OS related files, for example.
