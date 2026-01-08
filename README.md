# gh-wt

A GitHub CLI extension for managing Git worktrees with ease.

## What are Git Worktrees?

Git worktrees allow you to have multiple working directories for a single repository. This is incredibly useful when you need to:

- Work on multiple branches simultaneously
- Quickly switch between features without stashing
- Run tests on one branch while developing on another
- Review PRs without disrupting your current work

`gh-wt` makes working with worktrees simple and intuitive, with added features like templates and safety checks.

## Installation

```bash
gh extension install vansdevcode/gh-wt
```

## Quick Start

```bash
# Initialize a repository for worktree management
gh wt init myorg/myrepo

# Create a new worktree for a feature
cd myrepo
gh wt create main feature-awesome-thing

# Work in your new worktree
cd feature-awesome-thing
# ... make changes ...

# List all worktrees
gh wt list

# Delete a worktree when done
gh wt delete feature-awesome-thing
```

## Commands

### `gh wt init`

Initialize a new worktree-managed repository.

```bash
gh wt init <repo> [dir] [--new]
```

**Arguments:**
- `repo` - Repository in format `org/repo` or full git URL
- `dir` - Directory name (optional, defaults to repo name)
- `--new` - Create a new repository instead of cloning

**Examples:**

```bash
# Clone an existing repository
gh wt init myorg/myrepo

# Clone with custom directory name
gh wt init myorg/myrepo my-project

# Clone using full git URL
gh wt init git@github.com:myorg/myrepo.git

# Create a new repository
gh wt init myorg/newproject --new
```

**What it does:**
1. Creates a directory structure with a bare repository in `.bare/`
2. Clones the repository (or creates a new one with `--new`)
3. Automatically creates a worktree for the default branch
4. Initializes submodules if present
5. Applies templates if `.wt-templates/` exists

### `gh wt create`

Create a new worktree from a base branch.

```bash
gh wt create <base-branch> <new-branch-name>
```

**Arguments:**
- `base-branch` - Branch, tag, or commit to base from (e.g., `main`, `origin/develop`, `v1.0.0`)
- `new-branch-name` - Name for the new branch and directory

**Examples:**

```bash
# Create from main branch
gh wt create main feature-123

# Create from remote branch
gh wt create origin/develop fix-bug-456

# Create from a tag
gh wt create v1.0.0 hotfix-security

# Create from a specific commit
gh wt create a1b2c3d experiment
```

**What it does:**
1. Creates a new branch from the specified base
2. Creates a new directory with the branch name
3. Initializes submodules automatically
4. Processes and copies template files with variable replacement
5. Ready to start working immediately

### `gh wt delete`

Delete a worktree and its branch.

```bash
gh wt delete [branch-name] [--force]
```

**Arguments:**
- `branch-name` - Name of the branch/worktree to delete (optional, uses current directory if not specified)
- `--force`, `-f` - Force deletion even with uncommitted changes

**Examples:**

```bash
# Delete a specific worktree
gh wt delete feature-123

# Force delete with uncommitted changes
gh wt delete old-branch --force

# Delete current worktree (when inside a worktree directory)
gh wt delete
```

**Safety features:**
- Checks for uncommitted changes
- Warns about untracked files
- Prevents deleting worktree you're currently in
- Prompts for confirmation when there are untracked files

### `gh wt list`

List all worktrees in the current repository.

```bash
gh wt list
```

**Example output:**
```
Worktrees in /Users/you/projects/myrepo:

/Users/you/projects/myrepo/.bare           (bare)
/Users/you/projects/myrepo/main            a1b2c3d [main]
/Users/you/projects/myrepo/feature-123     d4e5f6g [feature-123]
```

## Template Support

One of the most powerful features of `gh-wt` is template support. Templates allow you to automatically set up configuration files for each worktree.

### Setting Up Templates

Create a `.wt-templates/` directory in your repository root:

```bash
mkdir .wt-templates
```

Any files you place in this directory will be copied to new worktrees with variable replacement.

### Available Variables

- `${WT_BRANCH}` - The branch name of the worktree
- `${WT_DIRECTORY}` - The absolute path to the worktree directory
- `${WT_ROOT_DIRECTORY}` - The absolute path to the repository root (where `.bare` is located)

### Example Templates

**`.wt-templates/.env`**
```bash
APP_URL=${WT_BRANCH}.myapp.test
APP_NAME=${WT_BRANCH}
APP_ENV=local
DATABASE_NAME=myapp_${WT_BRANCH}
```

**`.wt-templates/docker-compose.override.yml`**
```yaml
version: '3.8'
services:
  app:
    container_name: myapp_${WT_BRANCH}
    ports:
      - "8000:8000"
    environment:
      - BRANCH=${WT_BRANCH}
```

**`.wt-templates/.vscode/settings.json`**
```json
{
  "terminal.integrated.cwd": "${WT_DIRECTORY}",
  "git.defaultBranchName": "${WT_BRANCH}"
}
```

### Directory Structure in Templates

You can create subdirectories in `.wt-templates/` and they'll be preserved:

```
.wt-templates/
├── .env
├── .vscode/
│   └── settings.json
└── config/
    └── local.yml
```

All of these will be copied to each new worktree with variables replaced.

## Typical Workflow

Here's a typical development workflow using `gh-wt`:

```bash
# 1. Initialize your repository once
gh wt init myorg/myrepo

# 2. Set up templates (one-time)
cd myrepo
mkdir -p .wt-templates
echo 'APP_URL=${WT_BRANCH}.myapp.test' > .wt-templates/.env
git add .wt-templates
git commit -m "Add worktree templates"
git push

# 3. Create worktrees for each feature/fix
gh wt create main feature-user-auth
gh wt create main feature-payment
gh wt create main fix-login-bug

# 4. Work on each feature independently
cd feature-user-auth
# ... develop, commit, push ...

cd ../feature-payment
# ... develop, commit, push ...

# 5. List what you're working on
gh wt list

# 6. Clean up when done
gh wt delete feature-user-auth
gh wt delete feature-payment
```

## Tips & Best Practices

### 1. Use Descriptive Branch Names

Since the branch name becomes the directory name, use clear, descriptive names:

```bash
gh wt create main feature-user-authentication
gh wt create main fix-memory-leak-in-parser
gh wt create main refactor-payment-module
```

### 2. Keep the Root Clean

Work inside the worktree directories, not in the root. The root directory should only contain:
- `.bare/` - The bare repository
- `.wt-templates/` - Template files
- `main/` (or your default branch)
- Other worktree directories

### 3. Template Everything

Use templates for:
- Environment files (`.env`)
- Docker configurations
- IDE settings
- Database connection strings
- Local configuration files

### 4. Regular Cleanup

Delete worktrees you're no longer using:

```bash
# List to see what you have
gh wt list

# Clean up old branches
gh wt delete old-feature-1
gh wt delete old-feature-2
```

### 5. Submodules Work Automatically

If your repository has submodules, they're automatically initialized in each worktree. No extra steps needed!

## Repository Structure

After running `gh wt init`, your repository structure will look like this:

```
myrepo/
├── .bare/              # Bare repository (your .git folder)
├── .wt-templates/      # Template files (optional)
│   ├── .env
│   └── config/
│       └── local.yml
├── main/               # Default branch worktree
│   ├── .git            # Points to ../.bare
│   └── ... your code ...
├── feature-123/        # Feature branch worktree
│   ├── .git            # Points to ../.bare
│   ├── .env            # Generated from template
│   └── ... your code ...
└── fix-bug-456/        # Bug fix worktree
    ├── .git            # Points to ../.bare
    ├── .env            # Generated from template
    └── ... your code ...
```

## Troubleshooting

### "Not in a worktree-managed repository"

Make sure you've initialized the repository with `gh wt init` and you're in the repository root or one of its worktree directories.

### Worktree Already Exists

If you get an error that a worktree already exists, you can:

```bash
# List existing worktrees
gh wt list

# Delete the existing one
gh wt delete existing-branch-name

# Or use a different branch name
gh wt create main feature-123-v2
```

### Uncommitted Changes on Delete

If you try to delete a worktree with uncommitted changes:

```bash
# Commit or stash your changes first
cd feature-branch
git stash

# Or force delete
gh wt delete feature-branch --force
```

## Requirements

- [GitHub CLI](https://cli.github.com/) (`gh`) installed
- Git 2.5+ (for worktree support)
- macOS (Linux support coming soon)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

See [ROADMAP.md](ROADMAP.md) for planned features and ideas.

## License

MIT

## Credits

Created by [Vanderlei](https://github.com/vanderlei)

Inspired by the powerful but often underutilized Git worktrees feature.
