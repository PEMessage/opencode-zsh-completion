<h1 align="center">Opencode Zsh Completion</h1>

<p align="center">
  <strong>Smart shell completions for opencode CLI</strong><br>
  Intelligent tab completion for sessions, models, agents, and all subcommands
</p>

<p align="center">
  <a href="#installation"><img src="https://img.shields.io/badge/Shell-Zsh-89E051?logo=zsh" alt="Zsh"></a>
  <a href="#installation"><img src="https://img.shields.io/badge/Zinit-Supported-orange" alt="Zinit"></a>
  <a href="#installation"><img src="https://img.shields.io/badge/Oh_My_Zsh-Supported-yellow" alt="Oh My Zsh"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-AGPL--3.0-blue" alt="AGPL-3.0"></a>
</p>

<p align="center">
  <a href="#why-this-completion">Why?</a> •
  <a href="#features">Features</a> •
  <a href="#installation">Installation</a> •
  <a href="#usage">Usage</a> •
  <a href="#configuration">Configuration</a>
</p>

---

Zsh completions for [opencode](https://github.com/opencode-ai/opencode), an AI-powered CLI tool.

## Why This Completion?

The built-in `opencode completion zsh` only provides **subcommand names** (e.g., `mcp`, `debug`, `providers`), but no completion for:

- **Flags/Options** — typing `opencode -<TAB>` or `opencode --<TAB>` shows nothing
- **Argument values** — `opencode -s <TAB>` or `opencode -m <TAB>` has no completion

| What | Built-in | This Project |
|------|----------|--------------|
| Subcommand names | ✅ | ✅ |
| Flags & options (`-m`, `--model`, etc.) | ❌ | ✅ |
| Session values (`-s <TAB>`) | ❌ | ✅ with metadata |
| Model values (`-m <TAB>`) | ❌ | ✅ cached |
| Agent values (`--agent <TAB>`) | ❌ | ✅ |
| MCP server names | ❌ | ✅ |

## Features

- **Complete option completion** for all `opencode` flags (`-m`, `-c`, `-s`, `--model`, `--session`, `--agent`, etc.)
- **Subcommand completion** with descriptions for all commands:
  - `completion`, `mcp`, `debug`, `providers`, `agent`, `models`
  - `export`, `import`, `session`, `github`, `pr`, `run`, `serve`, `upgrade`
  - and more...
- **Smart session selection** with `opencode -s <TAB>` — shows recent sessions with titles, directories, and timestamps
- **Model completion** — caches and completes available AI models
- **Agent completion** — suggests available agents (`--agent <TAB>`)
- **MCP server management** — completion for `mcp add/list/auth/logout/debug`
- **Provider management** — completion for `providers list/login/logout`
- **Cached completions** for sessions and models (auto-invalidated when data changes)
- **Configurable session limit** — control how many recent sessions to show (default: 30)

## Installation

### Using [zinit](https://github.com/zdharma-continuum/zinit)

#### Option 1: Using snippet (lightweight, no clone)

```zsh
zinit ice as"completion"
zinit snippet https://raw.githubusercontent.com/PEMessage/opencode-zsh-completion/main/_opencode
```

#### Option 2: Clone as completion

```zsh
zinit ice as"completion" blockf
zinit light PEMessage/opencode-zsh-completion
```

### Using [oh-my-zsh](https://ohmyz.sh/)

Clone this repository into your custom plugins directory:

```zsh
git clone https://github.com/PEMessage/opencode-zsh-completion.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/opencode
```

Then add `opencode` to your plugins array in `~/.zshrc`:

```zsh
plugins=(... opencode)
```

### Manual Installation

You can manually install the completion file to any directory in your `$fpath`.

#### Option 1: User-specific installation (recommended)

```zsh
# Create completions directory if it doesn't exist
mkdir -p ~/.zsh/completions

# Download the completion file
curl -L https://raw.githubusercontent.com/PEMessage/opencode-zsh-completion/main/_opencode -o ~/.zsh/completions/_opencode

# Add to fpath in your ~/.zshrc (before calling compinit)
echo 'fpath+=(~/.zsh/completions)' >> ~/.zshrc
```

#### Option 2: System-wide installation

```zsh
# Download to system completions directory (may require sudo)
# Common paths:
# - macOS (Homebrew): /usr/local/share/zsh/site-functions or /opt/homebrew/share/zsh/site-functions
# - Linux: /usr/share/zsh/site-functions or /usr/local/share/zsh/site-functions

sudo curl -L https://raw.githubusercontent.com/PEMessage/opencode-zsh-completion/main/_opencode \
  -o /usr/local/share/zsh/site-functions/_opencode
```

#### Option 3: Using local clone

```zsh
# Clone the repository
git clone https://github.com/PEMessage/opencode-zsh-completion.git ~/opencode-zsh-completion

# Add to fpath in ~/.zshrc
echo 'fpath+=(~/opencode-zsh-completion)' >> ~/.zshrc
```

#### Reload completions

After installation, reload your shell configuration or run:

```zsh
# Remove cached function (if any) and reload
unfunction _opencode 2>/dev/null
autoload -U _opencode
# Reinitialize completions
compinit
```

Or simply open a new terminal window.

## Usage

After installation, use tab completion with the `opencode` command:

```zsh
# Complete global options
opencode --<TAB>

# Select a model
opencode -m <TAB>

# Select from recent sessions
opencode -s <TAB>

# List subcommands
opencode <TAB>

# Complete subcommand options
opencode mcp <TAB>
opencode debug <TAB>
opencode providers <TAB>

# Select provider for models command
opencode models <TAB>

# Complete MCP server names
opencode mcp auth <TAB>

# Export a session
opencode export <TAB>
```

## Configuration

You can customize the behavior of the completions using zstyle.

### Session Limit

The maximum number of recent sessions shown in the completion list (default: 30):

```zsh
# Show up to 50 recent sessions
zstyle ':completion:*:opencode:*' session-limit 50

# Show all sessions (no limit)
zstyle ':completion:*:opencode:*' session-limit 9999
```

### Cache Path

The location where session data is cached (auto-configured by default):

```zsh
zstyle ':completion:*:opencode:*' cache-path "${ZSH_CACHE_DIR:-$HOME/.cache/zsh}/opencode"
```

## License

GNU Affero General Public License v3.0 (AGPL-3.0) - see [LICENSE](LICENSE) file for details.
