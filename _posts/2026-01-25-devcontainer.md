In the past we'd have the problem of artifact mismatch between development and production. Containerization mitigated this, because we could build a Docker image once and promote it from environment to environment. The code would run as expected regardless of the host, since it lived inside the container.

Even so, we still develop and run the solution locally on our main machines, and that brings issues like incompatibility. A global library version can conflict with the one required by the project. It also adds another attack surface, because we might run malicious code without noticing it if a dependency gets poisoned (supply chain attack).

There are solutions that help mitigate this, and one of them is using a [devcontainer](https://containers.dev/). With it, we not only create the final image of the project, but we keep the whole development cycle inside a container. That isolates it from our OS, avoids unwanted incompatibilities, and prevents it from accessing sensitive data on our machine, like credentials.

This matters even more now with AI coding assistants like [Codex](https://openai.com/index/codex/), [Claude Code](https://www.anthropic.com/claude-code), and [OpenCode](https://opencode.ai/). They could potentially access our root repositories (especially if we run in yolo mode) and create catastrophic scenarios, like deleting important files and folders.
