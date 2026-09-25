# nix6 Environments: v3

How I set up an old PC for Python and GitHub work, written so that someone who hasn't used Linux in years (me, a week ago) can follow it without guessing.

My machine is **nix6**: a 2015 Dell OptiPlex 3040 (Intel i5-6500, 16 GB RAM) running **Ubuntu 24.04 LTS**. Anything similar will do.

By the end you'll have:

- git and GitHub working over SSH, set up once
- a repeatable pattern for every new Python project: its own folder under `~/code/`, its own `venv` inside that folder, pushed to GitHub
- optionally, two scripts ([`nix6-new` and `nix6-open`](README.md)) that do a whole new project in one command

**How to use this doc:** run one block at a time. Every step ends with a **checkpoint**, a command that proves it worked. Don't move on until the checkpoint matches. Where a command asks you a question, the answer is written down. Don't guess.

Wherever you see `<angle brackets>`, replace them, brackets and all, with your own value.

---

## Part 1: One-time machine setup

Do this once per machine. **Finish all of Part 1 before starting Part 2.** If you run `git init` before step 1.2, your first project ends up on a branch called `master` instead of `main`.

### 1.1 Install the tools

A fresh Ubuntu 24.04 doesn't come with git, the GitHub CLI, or Python's venv module.

```bash
sudo apt update
sudo apt install -y git gh python3-venv curl
```

It will ask for your password. Nothing appears on screen as you type it. That's normal.

**Checkpoint:**
```bash
git --version && gh --version && dpkg -s python3-venv > /dev/null && echo "ALL THREE OK"
```
You should see version lines for git and gh, then `ALL THREE OK`.

### 1.2 Tell git who you are

```bash
git config --global user.name "<your GitHub username>"
git config --global user.email "<your email>"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

**About the email:** it's published in every commit you push, where anyone can read it. If you'd rather it wasn't, GitHub gives you a private one. On github.com, go to **Settings → Emails**, tick **Keep my email addresses private**, and use the address it shows (it ends in `@users.noreply.github.com`).

**Checkpoint:**
```bash
git config --global --list | grep -E "user|init|pull"
```
Four lines: your name, your email, `init.defaultbranch=main`, `pull.rebase=false`.

### 1.3 Make an SSH key and give it to GitHub

This is what lets your machine push to GitHub without typing a password every time. There are two halves: make the key on your machine, then paste the public half into GitHub's website.

**On your machine:**
```bash
ssh-keygen -t ed25519 -C "<your email>"
```
It asks three questions. **Press Enter for all three:**
- `Enter file in which to save the key` → Enter (accepts `~/.ssh/id_ed25519`)
- `Enter passphrase` → Enter (no passphrase)
- `Enter same passphrase again` → Enter

(A passphrase is more secure, but you'd have to type it on every push. For a home machine, no passphrase is a common choice. It's your call.)

Now print the **public** half. This one is safe to share:
```bash
cat ~/.ssh/id_ed25519.pub
```
Select the whole line it prints, from `ssh-ed25519` to your email, and copy it with **Ctrl+Shift+C** (plain Ctrl+C doesn't copy in the terminal).

Never share `~/.ssh/id_ed25519` itself, the file without `.pub`. That's the private half.

**On github.com:**
1. Click your profile picture (top right), then **Settings**.
2. In the left menu, click **SSH and GPG keys**.
3. Click **New SSH key**.
4. **Title:** your machine's name (e.g. `nix6`). **Key type:** Authentication Key. **Key:** paste.
5. Click **Add SSH key**. GitHub may ask for your password.

**Checkpoint:**
```bash
ssh -T git@github.com
```
The first time, it asks `Are you sure you want to continue connecting (yes/no/[fingerprint])?`. Type **`yes`** in full and press Enter. You should then see:
```
Hi <your username>! You've successfully authenticated, but GitHub does not provide shell access.
```
That "does not provide shell access" part is expected. It means it worked.

### 1.4 Make sure `~/.local/bin` is on your PATH

This is where project commands (like `cast` from [localcast](https://github.com/IHoggan/localcast)) and the nix6 scripts live.

```bash
mkdir -p ~/.local/bin
echo $PATH | tr ':' '\n' | grep -qx "$HOME/.local/bin" && echo "PATH OK" || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

**Close the terminal and open a new one**, then:

**Checkpoint:**
```bash
echo $PATH | tr ':' '\n' | grep local/bin
```
Prints `/home/<you>/.local/bin`.

---

**Part 1 done.** You never need to repeat it on this machine.

If you want the one-command scripts, do [Appendix A](#appendix-a-the-github-cli-gh) now, then see the [README](README.md). Otherwise carry on with Part 2, which is the same thing done by hand, and worth doing once so you know what the scripts do.

---

## Part 2: Every new project

Replace `<project>` with your project's name: lowercase, no spaces (e.g. `weather-logger`).

### 2.0 Preflight: is Part 1 really done?

```bash
git config --global user.name > /dev/null && [ "$(git config --global init.defaultBranch)" = "main" ] && ssh -T git@github.com 2>&1 | grep -q "successfully authenticated" && echo "Part 1 OK" || echo "Part 1 NOT done: go back"
```
If it says `NOT done`, go back to Part 1. Muddling through is how branches end up called `master`.

### 2.1 Make the project folder

```bash
mkdir -p ~/code/<project>
cd ~/code/<project>
```

**Checkpoint:** `pwd` prints `/home/<you>/code/<project>`.

### 2.2 Make the venv inside it

Every project gets its own `venv` folder, inside the project. Its packages never clash with another project's or with the system's.

```bash
python3 -m venv venv
source venv/bin/activate
```

Your prompt now starts with `(venv)`. Install whatever the project needs, for example:
```bash
pip install requests rich
pip freeze > requirements.txt
```

**Checkpoint:** `which python` prints `/home/<you>/code/<project>/venv/bin/python`.

### 2.3 Tell git to ignore the venv

The venv is hundreds of files that are specific to your machine. It must never be pushed.

```bash
printf 'venv/\n__pycache__/\n*.pyc\n' > .gitignore
```

**Checkpoint:** `cat .gitignore` shows those three lines.

### 2.4 Add your files, then make the first commit

Write your code in the folder, or copy it in (e.g. `cp ~/Downloads/main.py .`). Then:

```bash
git init
git add .
git commit -m "Initial commit"
```

**Checkpoint:**
```bash
git branch --show-current
git ls-files | grep -c venv/
```
The first prints `main`. The second prints `0`: nothing from the venv was committed.

### 2.5 Create the repo on GitHub and push

**On github.com:** click **+** (top right), then **New repository**.
- **Repository name:** `<project>`, exactly the same as your folder
- Public or Private: your choice
- **Leave every "Initialize this repository with" box unticked** (no README, no .gitignore, no licence). You already have files, and GitHub's would clash with them.
- Click **Create repository**.

**Back on your machine:**
```bash
git remote add origin git@github.com:<your username>/<project>.git
git push -u origin main
```

The order matters: `git remote add` tells git *where* GitHub is. Pushing before that fails with `'origin' does not appear to be a git repository`.

**Checkpoint:** the push ends with `main -> main`, and refreshing the repo page on GitHub shows your files.

### 2.6 Coming back to it later

```bash
cd ~/code/<project>
source venv/bin/activate
```
Always activate the venv before running the project's Python. If you see `ModuleNotFoundError`, you forgot.

---

## Part 3: Day-to-day

### Save and push a change

```bash
git add <files you changed>
git commit -m "Say what changed, e.g. Fix the date format"
git push
```

### Try something bigger without risking `main`

Make a branch, work on it, and only bring it into `main` once it works:
```bash
git switch -c <idea-name>
# ... edit, commit, edit, commit ...
git switch main
git merge --ff-only <idea-name>
git push
git branch -d <idea-name>
```
If `merge --ff-only` refuses, `main` changed while you were away. Stop and look before forcing anything.

### Delete a project

On GitHub: the repo's **Settings** tab, scroll to the bottom, **Delete this repository**. Then locally:
```bash
rm -rf ~/code/<project>
```
`rm -rf` doesn't ask and there's no undo. Check the path twice.

---

## Part 4: Troubleshooting

**`git: command not found` / `gh: command not found`**
Step 1.1 wasn't done. `sudo apt install -y git gh python3-venv`

**`The virtual environment was not created successfully because ensurepip is not available`**
`python3-venv` is missing. `sudo apt install -y python3-venv`, then delete the half-made folder (`rm -rf venv`) and try again.

**`Please tell me who you are` when committing**
Step 1.2 wasn't done.

**First commit landed on `master`**
Step 1.2's `init.defaultBranch` wasn't set before `git init`. Fix this repo with `git branch -m master main`, then do 1.2 so it doesn't happen again.

**`Permission denied (publickey)` when pushing**
The SSH key isn't on GitHub, or you pasted the wrong half. Redo step 1.3 and check that `ssh -T git@github.com` says "successfully authenticated".

**`'origin' does not appear to be a git repository`**
You pushed before `git remote add origin ...` (step 2.5).

**`error: failed to push some refs` / `Updates were rejected`**
The GitHub repo has files yours doesn't, usually because a README was ticked at creation. Easiest fix for a brand-new project: delete the GitHub repo and recreate it with nothing ticked.

**`error: externally-managed-environment` from pip**
You're installing into the system Python. `source venv/bin/activate` first.

**A command you installed says `command not found`**
Open a new terminal (step 1.4 only applies to new ones), or run `source ~/.bashrc`. Also check `which <name>` **before** choosing a command name. Ubuntu already uses some obvious ones (for example `chat`, an old modem tool).

**`gh` is stuck asking for a token**
See Appendix A: you picked "Paste an authentication token" instead of "Login with a web browser". Press Ctrl+C and start again.

---

## Appendix A: the GitHub CLI (`gh`)

Optional for Part 2, but **required for the `nix6-new` / `nix6-open` scripts**, because they create and find repos for you.

```bash
gh auth login
```

It asks a series of questions. Use the arrow keys and Enter. The exact wording varies slightly between gh versions, so match on meaning:

| It asks | Answer |
|---|---|
| Where do you use GitHub? (or: What account do you want to log into?) | **GitHub.com** |
| Preferred protocol for Git operations? | **SSH** |
| Upload your SSH public key to your GitHub account? | **Skip**. You already added it in step 1.3. |
| How would you like to authenticate GitHub CLI? | **Login with a web browser**. Do **not** pick "Paste an authentication token": it demands a token you'd have to create first, and that's where people get stuck. |

It then shows a one-time code like `ABCD-1234`. Press Enter, and a browser opens. Type the code, click **Continue**, then **Authorize github**.

**Checkpoint:**
```bash
gh auth status
```
Shows `Logged in to github.com account <your username>`.

If you ever want `gh repo delete` to work, it needs one extra permission: `gh auth refresh -s delete_repo`.

---

## Version history

- **v3** (2026-09-25): SSH key made by hand and added on GitHub's website, which is easier to follow than `gh`'s prompts; `gh` moved to Appendix A with every answer written down. The venv lives inside the project folder. Added `python3-venv` to the installs, a `.gitignore` step with a check that nothing from the venv was committed, and the "push before remote add" trap. Placeholders instead of my own details. Moved from the localcast repo to nix6-tools.
- **v2** (2026-09-22): hardened after the first real run: `git` and `gh` weren't installed on fresh Ubuntu, and a preflight check was added.
- **v1** (2026-09-22): first draft.
