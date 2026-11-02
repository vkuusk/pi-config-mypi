# Pi agent setup "mypi"

Opinionated [Pi coding agent](https://pi.dev) config for DevOps and app development:
local LLMs, sandboxing, and a minimal set of MCP servers.

`mypi` is a small wrapper around `pi`. It runs pi in your current directory with the
config from this repo, and leaves your global `~/.pi/agent` setup untouched.

## Requirements

- [Pi coding agent](https://pi.dev) 1.4 or later
- An [Ollama](https://ollama.com) server with a tool-capable model
- [GitHub CLI](https://cli.github.com) (`gh`), to download MCP servers

## Activate

### 1. Clone the repo

```sh
git clone <repo-url> ~/work/pi-config-mypi
cd ~/work/pi-config-mypi
```

### 2. Run setup

```sh
./setup
```

It asks for two values and renders `.pi-agent/models.json` and `.pi-agent/mcp.json`
from their `.tmpl` files:

| Question | Example |
|---|---|
| Ollama API URL | `http://localhost:11434` |
| Base directory for stdio MCP servers | `~/tools` |

Your answers are saved in `.mypi.env`, which git ignores, and become the defaults
next time. Run `./setup` again after changing a template.

### 3. Choose your models

To change the models, edit `.pi-agent/models.json.tmpl`, not the rendered file, and
run `./setup` again. List models your server has, and set `contextWindow` to the
context size your Ollama server uses.

### 4. Put `mypi` on your `PATH`

```sh
ln -s "$PWD/mypi" ~/.local/bin/mypi
```

### 5. Install the MCP servers

Follow the steps for each server:

- [codebase-memory-mcp](dependencies/codebase-memory-mcp/README.md)

### 6. Start it

```sh
cd <your-project>
mypi
```

The first start installs the packages listed in `.pi-agent/settings.json`.

## Linux

`mypi` and `setup` work the same way on Linux. These commands are for Ubuntu and other
Debian-based systems.

Install pi with Node.js 22.19 or later, and the tools the sandbox needs:

```sh
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
sudo apt install -y bubblewrap socat ripgrep
```

Install `gh` from [GitHub's apt repository](https://github.com/cli/cli/blob/trunk/docs/install_linux.md),
because the MCP server steps need a recent version.

Ubuntu 24.04 and newer restrict unprivileged user namespaces, so every sandboxed command
fails with `bwrap: ... Operation not permitted`. Nothing runs unsandboxed, but commands
won't work until you add an
[AppArmor exception](https://github.com/wynainfo/pi-permission-modes/blob/main/docs/troubleshooting.md#bubblewrap-namespaces).

If `mypi` isn't found after step 4, create `~/.local/bin` and log in again.
