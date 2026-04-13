# Cornell Notes

## Topic: Git Submodules and Cloning with Submodules

## Date: 17/03/2026

---

### Cue Column (Questions, Keywords, or Prompts)

- How to clone with submodules?
- How to add a new submodule?
- What if I forgot to use `--recurse-submodules`?
- How to update submodules after pulling?
- How to check submodule status?
- What is detached HEAD in submodules?
- Why does `git push` fail with `src refspec '<branch>' must name a ref` in a submodule?
- How to run commands on all submodules?

---

### Notes Section (Main Notes)

#### 1. Cloning the repository (with submodules)

This project uses Git submodules. To avoid missing content, clone with submodules in a single step:

```bash
git clone --recurse-submodules https://github.com/VuThach3001/software-7dof-robotic-arm.git
```

Or (PowerShell on Windows – same command):

```powershell
git clone --recurse-submodules https://github.com/VuThach3001/software-7dof-robotic-arm.git
```

Then enter the directory:

```bash
cd software-7dof-robotic-arm
```

**If you already cloned without `--recurse-submodules`**

Run:

```bash
git submodule update --init --recursive
```

##### 1.1 Adding a new submodule

Use this command from the parent repository root:

```bash
git submodule add <submodule-repo-url> [path]
```

Example (same style as your recent command):

```bash
git submodule add https://github.com/VuThach3001/software-architecture.git
```

After adding, Git automatically updates:

- `.gitmodules`
- the submodule directory entry (gitlink) in the parent repository

Then commit both in the parent repository:

```bash
git add .gitmodules <submodule-path>
git commit -m "Add submodule: <name>"
git push
```

#### 2. Updating after pulling new changes

When you pull new commits that move submodule pointers, include submodules:

```bash
git pull --recurse-submodules
git submodule update --init --recursive  # (usually redundant after the pull, but safe)
```

If you want Git to always recurse automatically for this repo:

```bash
git config submodule.recurse true
```

Globally (applies to all repos):

```bash
git config --global submodule.recurse true
```

##### 3. Refresh submodules to the latest remote commits (only if you intentionally track remote heads)

Normally you should stay on the exact commit recorded by the superproject. If you intentionally want the latest remote changes inside each submodule branch:

```bash
git submodule update --remote --merge
```

##### 4. Shallow clone (faster) with submodules

```bash
git clone --depth 1 --recurse-submodules --shallow-submodules https://github.com/VuThach3001/software-7dof-robotic-arm.git
```

##### 5. Check submodule status

View the current commit each submodule is on:

```bash
git submodule status
```

For more detailed information:

```bash
git submodule foreach git status
```

##### 6. Execute commands on all submodules

Run any Git command on all submodules at once:

```bash
git submodule foreach 'git fetch'
git submodule foreach 'git checkout main'
```

##### 7. Sync submodule URLs

If submodule URLs change in `.gitmodules`, sync them:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

##### 8. Understanding detached HEAD state

Submodules are typically in "detached HEAD" state (pointing to a specific commit, not a branch). This is normal. To work on a branch inside a submodule:

```bash
cd path/to/submodule
git checkout main  # or any branch
# make changes, commit, push
cd ..
git add path/to/submodule
git commit -m "Update submodule to latest"
```

If you intend to commit and push changes from inside a submodule, make sure you are on a real local branch first. A detached HEAD can be used for inspection, but it is a poor state for development.

Example for this workspace when working in the ESP32 submodule:

```bash
cd esp32-robotic-arm
git checkout develop  # create it first if it does not exist locally
# make changes
git add .
git commit -m "Describe submodule change"
git push -u origin develop
cd ..
git add esp32-robotic-arm
git commit -m "Update esp32 submodule pointer"
git push origin develop
```

##### 9. Recursive push behavior and `push.recurseSubmodules`

If Git is configured with recursive push enabled, pushing the parent repository can also try to push each submodule.

Check the setting:

```bash
git config --get push.recurseSubmodules
git config --global --get push.recurseSubmodules
```

Recommended values:

- `on-demand`: push submodules only when needed and only when the required commits are not already available on the remote.
- `false`: push only the current repository.
- `true`: always recurse into submodules during push. This is stricter and can fail if a submodule is detached or missing the local branch being pushed.

Set a safer global default:

```bash
git config --global push.recurseSubmodules on-demand
```

Or disable recursive push globally:

```bash
git config --global push.recurseSubmodules false
```

### Common issues

- Forgot `--recurse-submodules`: submodule folders exist but are empty -> run the init/update command above.
- Authentication errors (private submodules): ensure you have access and are logged in (Git Credential Manager or a PAT). Re-run `git submodule update --init --recursive` after fixing credentials.
- Local edits inside a submodule not showing in main repo commits: commit & push inside the submodule first, then commit the updated submodule pointer in the main repo.
- `fatal: src refspec 'develop' must name a ref` during parent push: the submodule does not have a local `develop` branch, often because it is in detached HEAD state. Fix it inside the submodule:

```bash
cd path/to/submodule
git checkout -b develop  # or git checkout develop if it already exists remotely and locally
git branch --set-upstream-to=origin/develop develop
git push -u origin develop
```

Then return to the parent repository and push again.
- Parent repo push fails because of submodule recursion: check whether `push.recurseSubmodules` is set to `true`. If that behavior is not required, prefer `on-demand` or `false`.

---

### Summary Section (Summary of Notes)

Git submodules allow embedding external repositories within a parent repository. Always clone with `--recurse-submodules` or run `git submodule update --init --recursive` afterward. Use `git pull --recurse-submodules` or set `submodule.recurse true` for automatic updates. Submodules stay in detached HEAD state by default, pointing to specific commits, but you should switch to a real branch before making and pushing changes inside a submodule. Use `git submodule status` to check current state and `git submodule foreach` to run batch commands. If recursive push is enabled, make sure each changed submodule has a valid local branch and pushed commits before pushing the parent repository.

When adding a submodule, use `git submodule add ...` from the parent repository. Git creates or updates `.gitmodules` automatically, so you normally do not create that file manually.
