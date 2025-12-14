+++
title = "Why @ never works for me in Codex?!"
date = 2025-12-13
description = "I discovered why Codex's @ file search always returned 'no matches' for me. The culprit was a ~/.gitignore with * that I use for my dotfiles repo, which ripgrep's walker respects by default. With help from Codex itself, I found a better approach using git config status.showUntrackedFiles no."
+++

I have been recently experimenting with different LLM CLIs for my projects, as shown in my previous post about [wrangling multiple LLM CLIs](../2025-11-13-llm-clis/). However, I wasn't able to try much for Codex, for a very simple reason: `@` just never works and always just return “no matches”. I've found several issues on Github that people were submitting fixes for similar problems, so I thought at some point this would be resolved. But no, still the same a month later. Then I decided to do a bit digging myself, and oh well of course with some LLM help. The good thing is that Codex is [open source](https://github.com/openai/codex/), so I can just look into the codebase directly. In fact if you follow the [installation guide](https://github.com/openai/codex/blob/main/docs/install.md) you will run `cargo run --bin codex -- "explain this codebase to me"`, which serves [a great starting point](https://gist.github.com/ziyunli/d02df7a49846f4040cb76bb77e92d09c#file-codex_overview-md).


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

This made me realized that I have a `.gitignore` file in my home directory that ignores everything (with `*`). And because Codex uses ripgrep's walker that respects `.gitignore` by default, it effectively makes Codex think my home directory is empty!

But the reason why I have such a `.gitignore` file is that I put some of my dotfiles in Git repositories, and I don't want Git to track any other files in my home directory. After a couple more questions, Codex made some suggestions:

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
