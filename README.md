# Useful things in my .gitconfig

## gitignore

I use a global .gitignore_global, which I store in my home folder. I populate it at [gitignore.io](https://gitignore.io), which now redirects to [toplevel](https://www.toptal.com/developers/gitignore/).

```
[core]
	editor = emacs
	excludesfile = ~/.gitignore_global
```

## I use the macOS Keychain

```
[credential]
	helper=osxkeychain
```

## My most-used aliases: `co`, `note`, `list`, `thisbranch`

I started from here. My `co` alias i now much more extensive.

```
[alias]
	co = "!f(){ git checkout \"$@\"; git note; };f"
```
	
If you notice, this uses `git note` as well as `checkout`. My `git note` and `git list` system are invaluable in adding context when all my branches are named after specific JIRA tickets.

Here is note and list:

```
	# Add or read branch annotations
	note = "!msg() { notes=${*}; patch=$(git symbolic-ref --short HEAD); \
		if test $# -eq 0; then git config branch.$patch.note; \
		else git config --replace-all branch.$patch.note \"$notes\"; \
		git config branch.$patch.note; fi; }; msg"

    # List all branches and show their annotations
	list = "!/bin/bash -c 'set -f; git branch | while read line; do \
		current=\"  \"; \
		name=${line##\\* }; \
		if [ \"$name\" != \"$line\" ]; then current=\"* \"; fi; \
		description=`git config branch.$name.note`; \
		echo \"$current\"$name: $description; \
		done'"
```

For example:

```
example% git list
  EDU-0255: Fix typo in term_description.md
* EDU-9511: Remove 2021_pricing.md
  main: Main branch. Don't forget to pull.
example% git co main
Switched to branch 'main'
Main branch. Don't forget to pull.
```

## `thisbranch` and `thiscommit`

Using `thisbranch` is invaluable for pushing commits from branches. It's also handy to get a quick ref to `thiscommit`.

```
[alias]
    thisbranch = symbolic-ref --short HEAD
	 thiscommit = rev-parse HEAD
```

```
example% git list
  EDU-0255: Fix typo in term_description.md
  EDU-9511: Remove 2021_pricing.md
* main: Main branch. Don't forget to pull.
example% git co EDU-9511
Switched to branch 'EDU-9511'
Remove 2021_pricing.md
example% git thisbranch
EDU-9511
example% git thiscommit
b484d7ae2b2d5e998eb454b5ca9f72c9d8872b20
example% git push origin `git thisbranch`
...
```

## Branch Management

I find Unix-y names to be very helpful and more memorable than 

```
[alias]
	mvbranch = rename
	rmbranch = branch -D
```

## Getting ready for emergencies

```
[alias]
    # What commit am I adding to?
	log1 = "!git log -1 --pretty=format:\"%h - %an, %ar : %s\""

 	# Pretty reflog
	saveme = reflog show --date=short --pretty=format:'%h %ad | %s%d [%an]'
```

# Advice to new git learners: early-stage `git` safety practices

Note: These approaches help users who are ramping up into a production environment to have better safety mechanisms at hand to fall back on. 

## Use `git commit` rather than `git commit -m`

**Why**: You have greater control using the editor.

**Supports back-out.** If you leave the message blank, you can back out without committing, unlike `git commit -m` which always commits (assuming the staging is ready for it.)

**Supports reviewing changes before committing:** You can review changes before finalizing. It helps to catch mistakes or oversights.

**Using the editor supports detailed commit messages.** Sometimes, a commit involves complex modifications. A longer, more detailed commit message is informative when tracking back the intent and reasoning in recording the history of the change.

## Naming commit messages

Pretend the words “This commit” or “This commit will” are standing before the message. “Adds missing validation checks” or “Updates README with installation instructions”. Keep the main message short (preferably under 60 characters) and readable. Using `git commit` over `git commit -m` gives you the space and time you need to explain your commit in more detail.

## Pull only from `main`

The main branch is the most stable and tested version of the project. For new users, who are less experienced with version control, pulling from the main branch provides a more stable environment to work with.

Pulling from branches other than main may introduce new features and material that conflicts with the changes you are workeing on. You might inadvertently pull in changes that break the codebase or cause unexpected behavior.

**Tips:**

- Always check which branch you’re in (`git branch`) before pulling.
- Always check your status before pulling

## Stage files individually

New users should add files individually over `git add .` for several reasons:

1. **Intentional actions**. Adding files individually lets you review and selectively stage changes. You choose which changes to include in the each commit. This ensures you only record relevant modifications.
2. **Prevent accidental inclusions:** Using `git add .` stages all changes in the working directory, including potentially unwanted or unintended modifications.
3. **Understanding impact:** When adding files one by one, you get to inspect and understand the specific change being made before commits. This supports deeper understanding of your file base, and encourages more thoughtful version control practices.
4. **Granular commits:** Especially when working on single documents, each commit represents a single logical change. This is easier to track as it isolates specific changes and makes it simpler to revert unwanted ones. 
5. **Avoid clutter:** Depending on the maturity of your `.gitignore` file, using `git add .` may include temporary files, build artifacts, or other extraneous content that doesn't belong in the repository. 
6. **Mindfulness:** Adding files one-by-one encourages intentional action and attention to detail. New users are prompted to consider each change individually, fostering good habits for effective version control.

## Always commit or stash before changing branches with modifications. (aka the “lost sheep” problem)

Uncommitted file changes can follow you when changing branches. Here are some pointers.

1. **Always use branch awareness:** Be mindful of the branch you’re working on with `git branch`. Switching branches changes your work context.
2. **Changes can carry over:** When changes or unstaged, uncommitted, or unstashed, files will be carried over when switching branches. This means that modifications in the working directory remain present, potentially causing conflicts or unexpected behavior in the new branch. This can be catastrophic, especially when you’re not using the “use the editor to  commit” and “add files intentionally” rules.
3. **Conflicts:** When changes made in one branch conflict with changes in another, Git will not allow you to switch branches until the conflicts are resolved.
4. **Commit or stash before changing branches:** A commit saves the diffs to your version history. Stashes let you temporarily store changes and re-apply them later.
5. **Check your status before you change the branch:** Regularly check`git status` before switching branches. You’ll see information about changes relative to your working directory.

## Abandon Corrupt Branches

Instead of using `git` tooling to fix corrupt changes for you, the following provides a faster quick and dirty work-around for efforts centered on a single file update.

1. Copy the updated file to a safe place, like TextEdit or your desktop.
2. Commit or stash on the corrupt branch.
3. Switch back to `main` and then cut a new branch `git checkout -b another-branch`
4. Use your saved information to complete your work.
5. Do not delete the corrupted branch until after your work is done. You may need it as a reference.

# Cloning a Branch

**Clone a Single Branch**

$ **git clone https://github.com/temporalio/documentation**

Cloning into 'documentation'...

remote: Enumerating objects: 42229, done.

…

$ **cd documentation**

$ **git fetch --all**

$ **git checkout --track origin/EDU-1781**

branch 'EDU-1781' set up to track 'origin/EDU-1781'.

Switched to a new branch 'EDU-1781'

$ **git branch**

- EDU-1781

main

$ **git status**

On branch EDU-1781

Your branch is up to date with 'origin/EDU-1781'.

nothing to commit, working tree clean

$

# Handy Git Aliases

| alias | what | how |
| --- | --- | --- |
| thisbranch | Current branch, e.g. push origin `git thisbranch` | thisbranch = symbolic-ref --short HEAD |
| thiscommit | Current commit hash | thiscommit = rev-parse HEAD |
| thishash | Short version | thishash = log --pretty=format:'%h' -n 1 |
| url | Origin URL | url = "!f() { upstream=$(git rev-parse --abbrev-ref \
  --symbolic-full-name @{u} | cut -d'/' -f1); \
  git remote get-url $upstream; }; f" |
| repo | Open the repo | repo = "!f() { git url | xargs open ;};  f" |
| jira | Open JIRA page for ticket (requires branch use ticket name) | jira = "!f() { open "https://temporalio.atlassian.net/browse/$(git thisbranch)"; }; f" |
| branchurl | URL for this branch | branchurl = "!f() { url=$(git url); branch=$(git thisbranch); \
  echo "${url}/tree/${branch}"; }; f" |
| clonebranch | Fetch branch and check out | clonebranch = "!f() { git fetch --all; \
  git checkout --track origin/$1; git branch; }; f" |
| owner | Repo owner | owner = "!f() { echo git url | cut -d/ -f4; }; f" |
| upstream | Set upstream | upstream = "!f() { git branch --set-upstream-to=origin/$(git thisbranch); }; f" |
| pr | URL for branch PR | pr = "!f() { pull_core=$(git ls-remote -q | \
  grep $(git hash) | grep pull | \
  sed 's/^.*pull/pull/'); \
  echo "$(git url)/${pull_core}"; }; f" |
| openpr | Open branch PR | openpr = "!f() { echo $(git pr) | xargs open; }; f" |
| logn | Log n entries, e.g. git logn 3 | logn = "!f() { git log -${1:-3} \
  --pretty=format:\"%h - %an, %ar : %s\"; }; f" |
| ls | List changed files. | ls = diff --name-only HEAD~1 |
| undo | Uncommit, call with a path, git undo <path> | undo = checkout HEAD^ --  |
| saveme | Pretty reflog | saveme = reflog show --date=short --pretty=format:'%h %ad | %s%d [%an]' |
| mvbranch | Rename | mvbranch = "!f() { git branch -m \"$@\"; }; f" |
| home | Home directory at root of repo, cd git home; | home = "!f(){ cd `git rev-parse --show-toplevel`; };f" |

# More to come...