# codebase-memory-mcp

Manual install of [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)
for `mypi`. It connects over stdio, and pi starts it with each session.

Don't use the project's installer. `install.sh`, `curl | bash`, and
`codebase-memory-mcp install` configure every coding agent they detect, including
hooks and skills. `npm install -g` runs a postinstall script. These steps put one
binary in one folder and change nothing else.

## Locations

| What | Where |
|---|---|
| Binary | `<mcp-servers-dir>/codebase-memory-mcp/0.12.1/codebase-memory-mcp` |
| Index and logs | `.pi-agent/cbm-cache/` (ignored by git) |
| Server config | `.pi-agent/mcp.json.tmpl`, rendered by `./setup` |

`<mcp-servers-dir>` is the base directory you gave `./setup`. The commands below use
the default, `~/tools`, so change it if you chose another one.

The version is part of the binary path, so an upgrade means editing `command` in
`.pi-agent/mcp.json.tmpl` and running `./setup` again.

## Install

### 1. Choose your platform

Set the archive name in your shell, and keep the same shell open for the next steps:

| Platform | Archive |
|---|---|
| macOS, Apple Silicon | `codebase-memory-mcp-darwin-arm64.tar.gz` |
| macOS, Intel | `codebase-memory-mcp-darwin-amd64.tar.gz` |
| Linux, x86-64 | `codebase-memory-mcp-linux-amd64.tar.gz` |
| Linux, ARM64 | `codebase-memory-mcp-linux-arm64.tar.gz` |

```sh
ARCHIVE=codebase-memory-mcp-darwin-arm64.tar.gz
```

Other builds are listed in the
[release assets](https://github.com/DeusData/codebase-memory-mcp/releases/tag/v0.12.1).

### 2. Download

```sh
mkdir -p ~/tools/codebase-memory-mcp/0.12.1
cd ~/tools/codebase-memory-mcp/0.12.1
gh release download v0.12.1 -R DeusData/codebase-memory-mcp \
  -p "$ARCHIVE" -p checksums.txt
```

### 3. Verify

Stop if either check fails.

```sh
shasum -a 256 -c --ignore-missing checksums.txt
gh attestation verify "$ARCHIVE" \
  --repo DeusData/codebase-memory-mcp \
  --signer-workflow DeusData/codebase-memory-mcp/.github/workflows/_build.yml
```

On Linux without `shasum`, use `sha256sum -c --ignore-missing checksums.txt`.

### 4. Extract

This extracts only the binary and leaves out the archive's `install.sh`. The
macOS Apple Silicon binary is about 335 MB.

```sh
tar xzf "$ARCHIVE" codebase-memory-mcp
```

### 5. Check the connection

This needs `mypi` on your `PATH`, for example a symlink in `~/.local/bin`.

```sh
mypi mcp list
```

`codebase-memory` should show as connected with 17 tools.

## File watching

Watching is on by default. It re-indexes git projects registered by active
sessions, so the agent doesn't get outdated answers.

If it causes CPU or disk load, for example on a very large repo or slow storage,
turn it off. Run this from the root of this repo, because the cache path must
match `CBM_CACHE_DIR` in `.pi-agent/mcp.json`:

```sh
CBM_CACHE_DIR="$PWD/.pi-agent/cbm-cache" \
  ~/tools/codebase-memory-mcp/0.12.1/codebase-memory-mcp config set watcher_enabled false
```

With watching off, ask the agent to run `index_repository` again after changes.

## Stop the background daemon

The server shares a per-account background daemon across sessions. Run this from
the root of this repo, because the cache path must match `CBM_CACHE_DIR` in
`.pi-agent/mcp.json`:

```sh
CBM_CACHE_DIR="$PWD/.pi-agent/cbm-cache" \
  ~/tools/codebase-memory-mcp/0.12.1/codebase-memory-mcp daemon stop
```

## Remove

Stop the daemon, then:

```sh
mypi mcp remove codebase-memory
rm -rf ~/tools/codebase-memory-mcp .pi-agent/cbm-cache
```
