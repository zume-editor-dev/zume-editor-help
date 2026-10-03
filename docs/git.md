# Git

A built-in Git client, opened as its own tab (**Git → Open Git Tab**).

- **Local Changes** - stage / unstage files (with multi-select), see per-file diffs,
  and commit (subject + description, amend). Resolve conflicts inline.
- **Commit graph** - a lane graph of all branches, with per-commit file lists and
  diffs; filter or search commits.
- **Branches / tags / stashes / remotes** in a navigator, with checkout, create,
  rename, delete, merge, rebase and more from context menus.
- **Fetch / Pull / Push / Clone**, worktrees and LFS - these run your own `git`.
- The file panel follows the repository while the Git tab is active.

!!! info
    Local operations use libgit2; network and merge/rebase operations run the
    `git` already installed on your system.

!!! note "TODO"
    Add a screenshot here.
