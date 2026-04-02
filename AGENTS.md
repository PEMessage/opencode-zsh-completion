# Writing Zsh Completions

Reference guide for writing zsh completion functions.

## Reference Examples

The best way to learn zsh completions is to study existing ones:

```
/usr/share/zsh/functions/Completion/Unix/
```

Recommended files to study:
- `_ack` - Simple completion with caching
- `_dput` - Completion with helper functions and caching policy
- `_gradle` - Complex completion with multiple states
- `_adb` - Extensive completion with many options
- `_subversion` - Complex state-based completion

## Basic Structure

Every completion file starts with:

```zsh
#compdef command-name
```

This tells zsh which command this completion is for.

### Simple Completion Function

```zsh
#compdef mycommand

_mycommand() {
  local curcontext="$curcontext" state line

  _arguments -C \
    '(-h --help)'{-h,--help}'[Show help message]' \
    '(-v --version)'{-v,--version}'[Show version]' \
    '(-o --output)'{-o,--output}'[Specify output file]:output file:_files' \
    '*:file:_files'
}

_mycommand "$@"
```

## The `_arguments` Function

`_arguments` is the main function for parsing command-line options.

### Option Syntax

```zsh
'(<exclusions>)<options>:<description>:<action>'
```

- **Exclusions**: Options that cannot be used together (optional)
- **Options**: The option flags, e.g., `-h` or `--help`
- **Description**: Help text shown in completion menu
- **Action**: What to complete after this option

### Common Actions

| Action | Meaning |
|--------|---------|
| `_files` | Complete files |
| `_directories` | Complete directories only |
| `_command_names` | Complete command names |
| `_values` | Complete from a list of values |
| `_describe` | Describe completion entries |
| `->state` | Jump to state handler |

### Option Patterns

```zsh
# Simple option
'-h[Show help]'

# Short and long form
'(-h --help)'{-h,--help}'[Show help]'

# Option with argument (required)
'-o+[Specify output]:output file:_files'

# Option with argument (optional)
'-o-[Specify output]:output file:_files'

# Mutually exclusive options
'(-v --quiet)'{-v,--verbose}'[Be verbose]'
'(-v --quiet)'{-q,--quiet}'[Be quiet]'

# Multiple arguments
'*-I[Add include path]:include path:_directories'

# Positional arguments (non-option)
'1:first arg:_files'      # First positional arg
'2:second arg:_files'     # Second positional arg
'*:remaining args:_files' # All remaining args
```

## State-Based Completion

For complex completions that need different behavior based on context:

```zsh
#compdef mycommand

_mycommand() {
  local curcontext="$curcontext" state line
  local -A opt_args

  _arguments -C \
    '(-h --help)'{-h,--help}'[Show help]' \
    '--format=[Output format]:format:->formats' \
    '1: :->first' \
    '*: :->rest' \
    && ret=0

  case "$state" in
    formats)
      local -a formats
      formats=(
        'json:JSON output'
        'yaml:YAML output'
        'xml:XML output'
      )
      _describe -t formats 'output format' formats
      ;;
    first)
      _files
      ;;
    rest)
      _files
      ;;
  esac
}

_mycommand "$@"
```

## Caching with `_store_cache`

Every Tab press in zsh starts a new process, so **global variables do NOT persist**. Use the official cache API:

```zsh
#compdef mycommand

# Enable caching
zstyle ':completion:*' use-cache on
zstyle ':completion:*:mycommand:*' cache-path "${ZSH_CACHE_DIR:-$HOME/.cache/zsh}/mycommand"

# Caching policy function
_mycommand_caching_policy() {
  # $1 is the cache file path
  local config_file="$HOME/.mycommandrc"
  
  # Invalidate if config is newer than cache
  [[ -f "$config_file" && "$config_file" -nt "$1" ]] && return 0
  
  # Invalidate if cache is older than 1 week
  local -a oldp
  oldp=( "$1"(Nmw+1) )
  (( $#oldp )) && return 0
  
  # Cache is valid
  return 1
}

_mycommand() {
  local curcontext="$curcontext" state line

  _arguments -C \
    '--list=[Select from list]:item:_mycommand_list' \
    '*:file:_files'
}

_mycommand_list() {
  local update_policy
  
  # Set up caching policy
  zstyle -s ":completion:${curcontext}:" cache-policy update_policy
  [[ -z "$update_policy" ]] && zstyle ":completion:${curcontext}:" cache-policy _mycommand_caching_policy
  
  # Cache API usage
  if _cache_invalid mycommand-list || ! _retrieve_cache mycommand-list; then
    # Cache miss - fetch fresh data
    local -a _mycommand_items
    _mycommand_items=()
    
    # Populate array (expensive operation)
    _mycommand_items=(
      'item1:First item description'
      'item2:Second item description'
      'item3:Third item description'
    )
    
    # Store in cache (persists across Tab presses)
    [[ ${#_mycommand_items} -gt 0 ]] && _store_cache mycommand-list _mycommand_items
  fi
  
  # Use cached or retrieved data
  _describe -t items 'available items' _mycommand_items
}

_mycommand "$@"
```

### Cache API Functions

| Function | Purpose |
|----------|---------|
| `_cache_invalid <name>` | Returns 0 if cache needs regeneration |
| `_retrieve_cache <name>` | Loads cached variables from file |
| `_store_cache <name> <var>...` | Saves variables to cache file |

### Cache Storage

- Default location: `~/.zcompcache/`
- Custom path via `zstyle ':completion:*:cmd:*' cache-path <path>`
- Cache files are plain text that can be inspected

## Helper Completion Functions

### `_describe`

Shows completion entries with descriptions:

```zsh
local -a items
items=(
  'start:Start the service'
  'stop:Stop the service'
  'restart:Restart the service'
)
_describe -t actions 'service action' items
```

### `_values`

For completing values without descriptions:

```zsh
_values -s ',' 'keywords' foo bar baz
```

### `_guard`

Restricts what can be entered:

```zsh
# Only allow non-option arguments
'1: :_guard "^-*" argument'
```

### `compset`

Manipulates completion context:

```zsh
# Remove prefix up to '='
compset -P 1 '*='

# Remove suffix from space onward
compset -S ' *'
```

## Testing Completions

1. **Add to fpath**:
   ```zsh
   fpath+=(/path/to/your/completion/dir)
   ```

2. **Reload completions**:
   ```zsh
   unfunction _yourcommand
   autoload -U _yourcommand
   ```
   
   Or open a new shell/terminal window.

3. **Test completion**:
   ```zsh
   yourcommand <TAB>
   yourcommand -<TAB>
   yourcommand --opt<TAB>
   ```

4. **Enable debugging** (optional):
   ```zsh
   # Add to completion function
   setopt xtrace  # Debug mode
   ```

## Common Patterns

### Reading from external command

```zsh
local -a items
local data

# Fetch and parse data
data=$(mycommand --list 2>/dev/null)
while IFS= read -r line; do
  [[ -n "$line" ]] && items+=("${line}")
done <<< "$data"

_describe -t items 'available items' items
```

### File type filtering

```zsh
# Only .txt files
':input file:_files -g "*.txt"'

# Any file except specific pattern
':output file:_files -g "^*.bak"'
```

### Exclusive options

```zsh
_arguments -C \
  '(-v --verbose -q --quiet)'{-v,--verbose}'[Verbose output]' \
  '(-v --verbose -q --quiet)'{-q,--quiet}'[Quiet mode]' \
  '*:file:_files'
```

## Best Practices

1. **Always use the cache API** for expensive operations - global variables don't persist across Tab presses
2. **Set a caching policy** to invalidate cache when source data changes
3. **Use `2>/dev/null`** when calling external commands to avoid errors in completion
4. **Use absolute paths** for external commands if PATH might not be set
5. **Document your caching policy** - when and why cache is invalidated
6. **Test in a fresh shell** to ensure completion works without your personal config

## Resources

- `man zshcompsys` - Zsh completion system manual
- `man zshcompwid` - Completion widgets
- `/usr/share/zsh/functions/Completion/` - Built-in completions (best reference)
- `which _store_cache` - View cache API implementation
