# AGENTS.md - Developer Guide for gh-wt

This guide is for AI coding agents and developers working on the gh-wt GitHub CLI extension.

## Project Overview

**Type:** GitHub CLI Extension (Bash Script)  
**Purpose:** Simplifies Git worktree management with templates and safety features  
**Language:** Bash (requires bash 4.0+)  
**Dependencies:** git (2.5+), gh CLI, standard Unix utilities (find, sed, mktemp, basename, dirname)

## Build, Test & Lint Commands

### Running the Extension
```bash
# Install locally for testing
gh extension install .

# Run commands
gh wt help
gh wt init myorg/myrepo
gh wt create main feature-branch
gh wt list
gh wt delete feature-branch
```

### Testing
**Current Status:** No test framework configured yet

**Planned:** bats (Bash Automated Testing System)
```bash
# When tests are added, run with:
bats test/
bats test/test_init.bats  # Run single test file
```

### Linting
**Current Status:** No linter configured

**Recommended:** shellcheck for bash linting
```bash
# Manual linting (when shellcheck is added):
shellcheck gh-wt
shellcheck -x gh-wt  # Follow sourced files
```

### Manual Testing
```bash
# Test the script directly
./gh-wt help
./gh-wt init test/repo test-dir --new

# Check script syntax
bash -n gh-wt  # Parse without execution
```

## Code Style Guidelines

### File Structure
1. **Shebang & Settings** (lines 1-2): `#!/usr/bin/env bash` and `set -e`
2. **Constants** (lines 4-9): Color codes in UPPERCASE
3. **Helper Functions** (lines 11-56): Output helpers and utilities
4. **Command Functions** (lines 59-490): One function per command
5. **Main Dispatcher** (lines 492-518): Command routing in `main()`

### Naming Conventions

**Variables:**
- Use `snake_case` for all variables
- Use `local` keyword for function-scoped variables
- Use descriptive names: `worktree_dir`, `branch_name`, `create_new`
- Boolean flags: `create_new=true`, `force=false`

**Constants:**
- Use `UPPERCASE` for constants: `RED`, `GREEN`, `NC`
- Define at top of script

**Functions:**
- Use `snake_case`: `find_root()`, `get_default_branch()`
- Command functions: `cmd_<name>()` pattern (e.g., `cmd_init()`, `cmd_create()`)
- Helper functions: descriptive verb+noun (e.g., `convert_gh_format()`)

### Bash Style

**Indentation:**
- 4 spaces (no tabs)
- Consistent throughout

**Conditionals:**
- Use `[[ ]]` (modern bash), never `[ ]` or `test`
- Always quote variables: `[[ -z "$var" ]]`
- Regex matching: `[[ "$repo" =~ ^pattern$ ]]`

**Functions:**
- K&R brace style: `function_name() {`
- Always use local variables: `local var="value"`
- Return status codes: `return 0` for success, `return 1` for failure

**Loops:**
- While loops for argument parsing:
```bash
while [[ $# -gt 0 ]]; do
    case $1 in
        --flag) flag=true ;;
        *) positional="$1" ;;
    esac
    shift
done
```

**Case Statements:**
- Use for command dispatch and flag parsing
- Support multiple aliases: `delete|remove|rm)`
- Always include default case: `*)`

### Quoting Rules

**Always quote:**
- Variable expansions: `"$variable"` or `"${variable}"`
- Command substitutions: `"$(command)"`
- File paths and user input

**Exception (no quotes):**
- Arithmetic: `(( count++ ))`
- When word splitting is intentionally needed (rare)

### String Operations

```bash
# Preferred patterns:
dir="$(basename "$repo" .git)"          # Extract basename
current_dir="$(dirname "$current_dir")"  # Get parent directory
content="${content//pattern/replacement}" # String replacement

# Use sed for complex operations:
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
```

### Error Handling

**Exit on Errors:**
- Script uses `set -e` - any command failure exits
- Override when needed: `command || true` or `command || handle_error`

**Error Messages:**
- Use `error()` helper for all errors
- Provide actionable messages with usage examples
- Always exit with status 1: `exit 1`

**Validation Pattern:**
```bash
if [[ -z "$required_arg" ]]; then
    error "Argument required. Usage: gh wt command <arg>"
fi

if [[ -d "$dir" ]]; then
    error "Directory '$dir' already exists"
fi
```

**Git Command Error Handling:**
```bash
# Multiple fallback attempts:
git command arg1 2>/dev/null || \
    git command arg2 2>/dev/null || \
    git command arg3 || \
    error "Operation failed"
```

### Output Patterns

**User Messages:**
- Use color helpers: `info()`, `success()`, `warning()`, `error()`
- Info: Blue for progress updates
- Success: Green for completed operations
- Warning: Yellow for cautions
- Error: Red for failures (to stderr)

**Examples:**
```bash
info "Creating worktree for branch: $branch_name"
success "✓ Worktree created successfully"
warning "⚠ This worktree has uncommitted changes"
error "Directory '$dir' already exists"
```

### Git Operations

**All git commands use bare repository:**
```bash
git --git-dir="$bare_dir" worktree add "$branch" "$branch"
git --git-dir="$bare_dir" worktree list
git --git-dir="$bare_dir" branch -D "$branch"
```

**Never use:**
- Relative git operations without `--git-dir`
- Bare `git` commands (must specify bare directory)

### Command Structure

**Standard Pattern:**
```bash
cmd_command_name() {
    local required_arg=""
    local optional_arg=""
    local flag=false
    
    # Parse arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            --flag) flag=true; shift ;;
            *) 
                if [[ -z "$required_arg" ]]; then
                    required_arg="$1"
                else
                    error "Too many arguments"
                fi
                shift
                ;;
        esac
    done
    
    # Validate
    if [[ -z "$required_arg" ]]; then
        error "Required argument missing"
    fi
    
    # Execute
    info "Performing operation..."
    # ... implementation ...
    success "✓ Operation completed"
}
```

## Common Patterns

### Directory Traversal
```bash
local current_dir="$PWD"
while [[ "$current_dir" != "/" ]]; do
    if [[ -d "$current_dir/.bare" ]]; then
        echo "$current_dir"
        return 0
    fi
    current_dir="$(dirname "$current_dir")"
done
```

### Template Processing
```bash
find "$templates_dir" -type f | while read -r template_file; do
    content="$(<"$template_file")"
    content="${content//\$\{VAR\}/$value}"
    echo "$content" > "$output_file"
done
```

### Safety Checks
```bash
# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
    warning "Worktree has uncommitted changes"
    if [[ "$force" != true ]]; then
        error "Use --force to proceed anyway"
    fi
fi
```

## Special Considerations

### Template Variables
When adding new template variables, update:
1. The `process_templates()` function
2. README.md documentation
3. Template examples

### Command Aliases
Support common aliases in case statement:
```bash
case "$command" in
    delete|remove|rm)
        cmd_delete "$@"
        ;;
esac
```

### Environment Variables
- `WT_ROOT`: Set to repository root, used by templates
- Template vars: `${WT_BRANCH}`, `${WT_DIRECTORY}`, `${WT_ROOT_DIRECTORY}`

## Testing Checklist

When adding features, manually test:
- [ ] Command with valid arguments
- [ ] Command with missing required arguments
- [ ] Command with invalid arguments
- [ ] Command with existing/conflicting resources
- [ ] Command with --force flag (if applicable)
- [ ] Error messages are clear and actionable
- [ ] Success messages confirm action taken
- [ ] Templates are processed correctly (if applicable)

## Documentation Requirements

When adding/modifying commands:
1. Update `cmd_help()` function in gh-wt
2. Update README.md with examples
3. Add to ROADMAP.md if it's a planned feature
4. Include usage examples in error messages

## Commit Message Style

Based on the initial commit in the repository:
- Use clear, descriptive messages
- Start with action verb: "Add", "Update", "Fix", "Refactor"
- Reference issues when applicable: "Fix #123: ..."

## Safety & Security

**Never:**
- Execute arbitrary user input
- Remove directories without validation
- Force push to main/master
- Skip safety checks without explicit `--force` flag
- Hardcode credentials or tokens

**Always:**
- Validate file paths are within expected directories
- Check for uncommitted changes before destructive operations
- Provide --force flag for risky operations
- Quote all variables to prevent injection
- Use `set -e` to fail fast on errors

## Future Proofing

When contributing, consider:
- Bash version compatibility (target 4.0+)
- Platform compatibility (macOS, Linux)
- Git version compatibility (2.5+)
- Graceful degradation when optional features unavailable
- Clear error messages when dependencies missing
