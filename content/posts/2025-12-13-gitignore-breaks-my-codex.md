+++
title = "Why @ never works for me in Codex?!"
date = 2025-12-13
+++

I have recently been experimenting with different LLM CLIs for my projects, as shown in my previous post about [wrangling multiple LLM CLIs](../2025-11-13-llm-clis/). But I could not use Codex much for a simple reason: `@` file search never worked for me and always returned “no matches”. I found several issues on GitHub where people were submitting fixes for similar problems, so I assumed it would get resolved. It did not, so I decided to dig in myself (with some LLM help).

Codex is [open source](https://github.com/openai/codex/), so I could read the code. If you follow the [installation guide](https://github.com/openai/codex/blob/main/docs/install.md), you will run `cargo run --bin codex -- "explain this codebase to me"`, which is [a great starting point](https://gist.github.com/ziyunli/d02df7a49846f4040cb76bb77e92d09c#file-codex_overview-md).

Then I asked:

```
› File search/match using @ never works for me - it always just show "no matches". Any idea why? Help me find the related code
```

The model responded with:

```
Related code path

- tui/src/bottom_pane/chat_composer.rs:1696: detects the current @token and emits AppEvent::StartFileSearch(query) (query excludes the @).
- tui/src/file_search.rs:167: debounces and calls codex_file_search::run(..., respect_gitignore=true).
- tui/src/bottom_pane/file_search_popup.rs:139: renders "no matches" when results come back empty.
- file-search/src/lib.rs:157: uses ignore::WalkBuilder (ripgrep’s walker) and by default respects .gitignore, global gitignore, .ignore, parent ignores.
- file-search/src/lib.rs:216: drops any path that fails path.strip_prefix(search_directory).

Why you might always get “no matches” (most likely)

- You launched Codex from a directory that’s effectively “empty” to the walker because ignore rules exclude everything (global gitignore is the classic footgun). This repo’s file search always sets
respect_gitignore=true (tui/src/file_search.rs:167).
- Your cwd path is a symlink-y path and the walker yields paths that don’t share a lexical prefix with search_directory, so strip_prefix fails and every file is discarded (file-search/src/lib.rs:227). This can
make it look like “search is broken” while traversal is happening.
```

This made me realize the problem was self-inflicted. I had a `~/.gitignore` file that ignores everything (just `*`). Codex uses a ripgrep-style filesystem walker (`ignore::WalkBuilder`) and runs it with `respect_gitignore=true`, which means it will apply ignore rules from parent directories. So if you run Codex anywhere under your home directory, `~/.gitignore` can make the walker filter out almost every file, and `@` search comes back empty.

The reason I had this file is that I keep some dotfiles in Git, and I did not want Git to show every other file in my home directory as untracked. After a couple more questions, Codex made some suggestions:

```
Why ~/.gitignore="*" is toxic

- Tools that implement gitignore semantics by walking parent dirs (ripgrep/ignore crate, fd, etc.) can treat ~/.gitignore as an ignore file for everything under your home directory, not just your dotfiles repo.
- That breaks file pickers/search in unrelated projects.

Bare dotfiles repo pattern

- You store the git database in a hidden dir, and set the work tree to $HOME.
- You then only track files you explicitly add; untracked noise in $HOME stays untracked.

Setup (one-time)

git init --bare "$HOME/.dotfiles"
alias dotfiles='git --git-dir="$HOME/.dotfiles" --work-tree="$HOME"'
dotfiles config status.showUntrackedFiles no

Ignore-everything for this repo only
Use the repo’s excludes file (applies only to this repo):

# this path is inside the bare repo
printf "*\n!.gitignore\n" >> "$HOME/.dotfiles/info/exclude"

Then whitelist what you want by force-adding:

dotfiles add -f "$HOME/.zshrc" "$HOME/.gitconfig"
dotfiles commit -m "Track zshrc and gitconfig"

Notes / gotchas

- info/exclude is like .gitignore but not versioned; it’s local-only. If you want the ignore rule shared across machines, commit a ~/.gitignore inside the repo (i.e., $HOME/.gitignore) but keep it repo-scoped by
ensuring it’s not accidentally treated as a global ignore file by other tooling. The safest is still info/exclude.
- With status.showUntrackedFiles no, your dotfiles status won’t spam you with every file in $HOME.
```

I felt the whole setup is a bit too much for my use case, but `config status.showUntrackedFiles no` seems to be a good enough solution. The only other issue I found is that it doesn't quite work with VS Code, so I turned to Codex again for help:

```
  VS Code’s Git extension does not rely on your status.showUntrackedFiles=no default. It typically runs git status --porcelain … -uall (or equivalent), and -uall overrides status.showUntrackedFiles.

  What to do instead:

  - Set VS Code to hide untracked:
      - Settings → search “untracked” → set Git: Untracked Changes to hidden
      - Or in settings.json: "git.untrackedChanges": "hidden"
```

...and now I am unblocked to use Codex with the help from Codex itself!
