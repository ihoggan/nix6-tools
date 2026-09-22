# Installing nix6-new and nix6-open

Two Bash scripts that automate the nix6 project workflow. `nix6-new` bootstraps a fresh project (folder + venv + git + GitHub + push, all in one). `nix6-open` resumes an existing project (local or from GitHub).

## Prerequisites

Both scripts assume you've done Part 1 of `nix6_environments.md` AND Appendix A (gh CLI authenticated). If either isn't done, the scripts fail fast with a clear message telling you which step to go back to.

## Install (one-time)

Download `nix6-new` and `nix6-open` to `~/Downloads`, then run these commands (each line is one command):

```bash
# 1. Make sure the target directory exists (safe to run even if it does)
mkdir -p ~/.local/bin

# 2. Move both scripts in
mv ~/Downloads/nix6-new ~/.local/bin/
mv ~/Downloads/nix6-open ~/.local/bin/

# 3. Make them executable
chmod +x ~/.local/bin/nix6-new ~/.local/bin/nix6-open

# 4. Make sure ~/.local/bin is on PATH (adds to .bashrc if not)
echo $PATH | tr ':' '\n' | grep -q "$HOME/.local/bin" && echo "PATH OK" || {
    echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    echo "Added to .bashrc and sourced."
}

# 5. Verify
which nix6-new
which nix6-open
```

Both `which` calls should print `/home/iain/.local/bin/nix6-new` (and similar for nix6-open). If either prints nothing, open a fresh terminal and re-run `which nix6-new`.

### Installing OVER an existing version (updates)

If you've been given a fixed or updated script, the process is the same as install — the `mv` will overwrite the old one. Just re-run steps 2 and 3:

```bash
mv ~/Downloads/nix6-new ~/.local/bin/nix6-new
chmod +x ~/.local/bin/nix6-new
```

If your browser renamed the download (e.g. `nix6-new (1)` because the original was already in Downloads), use that filename in the `mv` source.

## Usage: nix6-new

### Simplest form
```bash
nix6-new mytool
```
Creates `~/code/mytool/` with a bare Python scaffold, initialises git on the `main` branch, creates a public GitHub repo under your account, and pushes.

### With a description
```bash
nix6-new mytool --desc "A little CLI utility"
```
The description goes into both the commit message and the GitHub repo's description field.

### With pip dependencies pre-installed
```bash
nix6-new mytool --desc "..." --deps "click,rich,requests"
```
Installs those packages into the venv AND adds them to `requirements.txt`.

### Private repo
```bash
nix6-new mytool --private
```

### With your own starter files
```bash
nix6-new mytool --from ~/Downloads/mytool-draft
```
Copies everything from `~/Downloads/mytool-draft/` into the new project folder instead of using the default scaffold. If that folder has a `requirements.txt` with real entries, they'll be installed automatically.

### What ends up in the project (default scaffold)

```
~/code/mytool/
├── venv/               # local virtualenv (gitignored)
├── .git/
├── .gitignore          # Python-focused
├── README.md           # just name + description
├── main.py             # trivial entry-point stub
└── requirements.txt    # empty (or your --deps, one per line)
```

### After it runs

Script prints:
```
✓ mytool created at /home/iain/code/mytool
✓ Pushed to https://github.com/IHoggan/mytool

To start working:
  cd /home/iain/code/mytool && source venv/bin/activate
```

Copy the last line, paste, and you're in the venv ready to code.

## Usage: nix6-open

### Open a project you already have locally
```bash
nix6-open mytool
```
Checks the venv exists (creates one if not), ensures `requirements.txt` deps are installed, then prints the cd + activate commands.

### Open a project from GitHub you haven't cloned yet
```bash
nix6-open mytool
```
Same command. If `~/code/mytool` doesn't exist, the script checks your GitHub account for a repo of that name and clones it in, creates the venv, installs `requirements.txt`.

### List all your local projects
```bash
nix6-open --list
```
Shows every project in `~/code/`, flagging any without a venv or without `.git/`.

### List your GitHub repos not yet cloned locally
```bash
nix6-open --remote
```
Handy for "what have I got on GitHub that I haven't touched on this machine?"

### After it runs

Same as `nix6-new` — prints the two commands to actually start work:
```
To start working:
  cd /home/iain/code/mytool && source venv/bin/activate
```

## Why the scripts don't just cd + activate for you

Bash scripts run in a child shell. When the script exits, any `cd` it did and any venv it activated die with the child shell — they can't affect your parent shell. So both scripts print the exact commands for you to run.

If you want a true "one command that lands you in the project with the venv active", you can add a shell function to your `~/.bashrc`:

```bash
# nix6-cd — actually cd into a project and activate its venv
nix6-cd() {
    local d="$HOME/code/$1"
    if [ ! -d "$d" ]; then
        echo "No such project: $1" >&2
        return 1
    fi
    cd "$d" || return 1
    if [ -f venv/bin/activate ]; then
        source venv/bin/activate
    fi
}
```

Then `nix6-cd mytool` really does drop you in the project with the venv active.

## Troubleshooting

### `Preflight...` then `✗ gh CLI not authenticated`
Do Appendix A of `nix6_environments.md`. Then retry.

### `✗ Repo <user>/<name> already exists on GitHub. Use nix6-open to resume.`
The repo exists on GitHub. Either:
- Pick a different name for the new project, OR
- Use `nix6-open <name>` to clone the existing one instead

### `✗ /home/iain/code/<name> already exists`
Local folder already exists. Either:
- Pick a different name, OR
- Delete it first: `rm -rf ~/code/<name>` (careful!), OR
- Use `nix6-open <name>` if it's a valid project you just want to resume

### Script says pushed successfully but GitHub page 404s
Sometimes GitHub takes a second or two to reflect a new push. Refresh the page. If still nothing after 30s, check with `gh repo view <user>/<name>`.

### `--from` copied files but not dotfiles like `.gitignore`
The script uses `shopt -s dotglob` before copying, so dotfiles ARE included. If yours weren't, they might not have been in the `--from` folder. Double-check with `ls -la <from-dir>`.

### `Branch is 'master', renaming to 'main'`
Not an error — script noticed your git installation defaults to `master` and renamed to `main` for you. It also sets `init.defaultBranch main` globally so it won't happen again. Take it as a hint that Part 1.2 might have been done wrong originally.
