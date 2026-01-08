# gh-wt Roadmap

Future features and enhancements for gh-wt.

## Planned Features

### High Priority

- **Switch Command** (`gh wt switch <branch>`)
  - Quick navigation between worktrees
  - Opens a new shell in the target worktree directory
  - Could integrate with tools like `zoxide` for fuzzy finding

- **Status Command** (`gh wt status`)
  - Show status of all worktrees at once
  - Display uncommitted changes, untracked files, and branch status
  - Useful for getting a bird's-eye view of work in progress

### Medium Priority

- **Sync Command** (`gh wt sync [branch]`)
  - Fetch latest changes from origin
  - Optionally rebase or merge current branch
  - Sync all worktrees or a specific one

- **Archive Command** (`gh wt archive <branch>`)
  - Archive a worktree instead of deleting it
  - Create a tar.gz of the worktree before deletion
  - Useful for preserving work-in-progress that might be needed later

- **Clone Command** (`gh wt clone <existing-branch> <new-branch>`)
  - Create a new worktree based on an existing worktree's branch
  - Useful for experimenting with variations

### Low Priority

- **Interactive Mode**
  - Use `fzf` or similar for interactive worktree selection
  - Commands like `gh wt switch` could show a menu

- **Hooks Support**
  - Pre/post hooks for create, delete operations
  - Allow custom scripts to run (e.g., installing dependencies, starting services)
  - Could be defined in `.wt-hooks/` directory

- **Worktree Configuration**
  - Per-worktree configuration file (`.wt-config`)
  - Store metadata like description, tags, related issues
  - Show in status/list commands

- **Branch Naming Conventions**
  - Support branch naming patterns (e.g., `feature/`, `fix/`, `hotfix/`)
  - Auto-prefix branches based on configuration

## Template Enhancements

- **Additional Template Variables**
  - `${WT_DATE}` - Current date
  - `${WT_USER}` - Git user name
  - `${WT_EMAIL}` - Git user email
  - `${WT_BASE_BRANCH}` - The branch the worktree was created from

- **Conditional Templates**
  - Apply different templates based on branch naming patterns
  - E.g., feature branches get different `.env` than hotfix branches

- **Template Functions**
  - Simple functions like `${WT_BRANCH_UPPER}` for uppercase branch name
  - `${WT_BRANCH_SLUG}` for URL-safe version

## Integration Ideas

- **GitHub Integration**
  - Auto-create draft PR when creating worktree
  - Link worktree to GitHub issue
  - Show PR status in `gh wt status`

- **Docker/Container Support**
  - Auto-start Docker containers per worktree
  - Use worktree name for container naming
  - Clean up containers on worktree delete

- **IDE Integration**
  - Generate workspace/project files for VSCode, JetBrains IDEs
  - Auto-open IDE when creating worktree

## Performance & Quality

- **Testing**
  - Add comprehensive test suite using `bats` or similar
  - Test all commands with various scenarios
  - CI/CD integration

- **Error Handling**
  - Improve error messages with suggestions
  - Add recovery mechanisms for failed operations
  - Validate prerequisites (git version, gh CLI availability)

- **Documentation**
  - Add man pages
  - Create video tutorials
  - More examples in README

## Community Requests

This section will be populated based on user feedback and feature requests.

---

Have an idea? Open an issue on GitHub!
