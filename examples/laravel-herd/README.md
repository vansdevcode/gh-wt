# Laravel Herd Example

This example demonstrates how to use Worktree Manager with Laravel Herd.

- `files` and `hooks` directories must be inside the `.worktree/` folder where `wtm` is initialized.
- `post-create` hook will install dependencies and link the worktree to Laravel Herd. It will make a database copy for the specific branch.
- `post-delete` will delete the branch database and unlink from Laravel Herd.
