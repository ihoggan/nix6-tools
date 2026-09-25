# nix6-tools

A new Python + GitHub project in one command, on an old PC running Ubuntu.

- **`nix6-new <name>`** makes the folder, a `venv` inside it, a starter set of files and the first commit, creates the GitHub repo and pushes.
- **`nix6-open <name>`** picks a project back up. If it isn't on this machine yet, it clones it from your GitHub first, then makes sure the venv and packages are in place.

Plus the full workflow they automate, written for someone coming back to Linux after years away: **[nix6_environments.md](nix6_environments.md)**.

Built on and for **nix6**, a 2015 Dell OptiPlex 3040 rescued from the skip and running Ubuntu 24.04. It's part of the same story as [localcast](https://github.com/IHoggan/localcast), persistent AI characters running on that machine.

## Before you start

The scripts need two things set up first. Both are in the workflow doc, with a checkpoint at every step:

1. **Part 1** of [nix6_environments.md](nix6_environments.md): git, your git identity, an SSH key on GitHub, and `~/.local/bin` on your PATH.
2. **Appendix A**: the GitHub CLI (`gh`) logged in.

If either is missing, the scripts stop straight away and tell you which step to go back to.

## Install

```bash
mkdir -p ~/code ~/.local/bin
git clone https://github.com/IHoggan/nix6-tools.git ~/code/nix6-tools
cp ~/code/nix6-tools/nix6-new ~/code/nix6-tools/nix6-open ~/.local/bin/
chmod +x ~/.local/bin/nix6-new ~/.local/bin/nix6-open
```

**Checkpoint:**
```bash
which nix6-new nix6-open
```
Prints two paths, `/home/<you>/.local/bin/nix6-new` and `/home/<you>/.local/bin/nix6-open`. If it prints nothing, open a new terminal and try again. If it still prints nothing, do step 1.4 of the workflow doc.

**To update later:**
```bash
cd ~/code/nix6-tools && git pull
cp nix6-new nix6-open ~/.local/bin/
```

## nix6-new

```bash
nix6-new mytool
```

This creates `~/code/mytool/`, gives it its own `venv`, commits a starter set of files on the `main` branch, creates a **public** GitHub repo called `mytool` under your account, and pushes.

Options:

```bash
nix6-new mytool --desc "A little CLI utility"      # description for the commit and the GitHub repo
nix6-new mytool --deps "requests,rich"             # install packages and list them in requirements.txt
nix6-new mytool --private                          # private repo instead of public
nix6-new mytool --from ~/Downloads/mytool-draft    # start from your own files instead of the starter set
```

Project names must be lowercase letters, digits and hyphens, starting with a letter.

The starter set:

```
~/code/mytool/
├── venv/               # this project's own Python packages (never committed)
├── .gitignore          # already ignores venv/
├── README.md           # the name and description
├── main.py             # a "hello" entry point
└── requirements.txt    # your --deps, one per line
```

When it finishes, it prints:

```
✓ mytool created at /home/<you>/code/mytool
✓ Pushed to https://github.com/<you>/mytool

To start working:
  cd /home/<you>/code/mytool && source venv/bin/activate
```

Copy that last line into your terminal and you're in.

## nix6-open

```bash
nix6-open mytool      # open a project (cloning it from your GitHub first if it isn't here)
nix6-open --list      # projects in ~/code, flagging any without a venv or git
nix6-open --remote    # your GitHub repos that aren't on this machine yet
```

Like `nix6-new`, it ends by printing the `cd ... && source venv/bin/activate` line to paste.

## Why don't they just put me in the folder?

A script runs in its own copy of the shell. Anything it does with `cd` or `source` disappears when it finishes, so it can't change *your* terminal. That's why both scripts print the line for you to paste.

If you'd like one command that really does drop you in, add this to the end of `~/.bashrc`, then open a new terminal:

```bash
nix6-cd() { cd "$HOME/code/$1" && { [ -f venv/bin/activate ] && source venv/bin/activate; true; }; }
```

Now `nix6-cd mytool` takes you into the project with its venv active.

## Troubleshooting

**`✗ gh CLI not authenticated`**: do Appendix A of the workflow doc.

**`✗ SSH to github.com failing`**: do step 1.3 of the workflow doc, and check that `ssh -T git@github.com` says "successfully authenticated".

**`✗ Repo <you>/<name> already exists on GitHub`**: pick another name, or use `nix6-open <name>` to work on the existing one.

**`✗ /home/<you>/code/<name> already exists`**: same choice. If you really want to start again, delete it with `rm -rf ~/code/<name>`. There's no undo, so check the name twice.

**The GitHub page is empty straight after pushing**: give it a few seconds and refresh. `gh repo view <you>/<name>` confirms from the terminal.

**`Branch is 'master', renaming to 'main'`**: not an error. The script fixed it, and set git's default so it won't happen again.

## Licence

MIT.
