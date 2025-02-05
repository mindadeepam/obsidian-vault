# some common tasks and their commands

## git clone

`git clone https://github.com/<username>/<repo>.git`

## git clone using username and password(token)

`git clone https://username:password@github.com/username/repository.git`

## folder level git credentials in local

In your workspace folder, you will need to use your own credentials to interact with the git repos.
To personalize your local git repo with the right credential update the local config file

```
# The file is located at .git/config

[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = https://username:password@github.com/FarMart-Tech/ds-serverless-functions
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "dev"]
        remote = origin
        merge = refs/heads/dev

[user]
        name = Deepak Mishra
        email = deepakmishra@farmart.co
```

## push to github

```bash
gh repo create                          # follow prompts to create repo

git init 
git add .                               # to remove added files:  git rm -r --cached .
git commit -m "message"
git branch -M main
git remote add origin https://github.com/mindadeepam/deep-learning 
git remote -v 
git push origin main

git rm -r --cached "file or dir name"   # remove folder/file
```


## contribute to a repo

### 1st time

1. clone the repo `git clone url/of/repo`
2. cd inside the repo
3. create your dev branch `git branch local_branch`
4. checkout to that dev branch `git checkout local_branch`
5. add the remote repository `git remote add origin url/of/repo`
6. create remote branch from your local branch `git push origin local_branch:remote_branch`
7. keep pushing to that remote_branch. raise PRs to dev.

### then on

**how to pull latest changes from dev to personal_dev?**

It's a good practice to as soon as feasible after person *A* pushes the changes to `dev` for person *B* to get these changes into their branch `b`. This is so that person *B* works on latest code and their eventual merge to `dev` is easy.

### Option 1, pull

1. Commit all changes to branch `feature_branch` (git status shows `clean`)
2. `git checkout dev`
3. `git pull` - this fetches (downloads) the changes onto computer `b` **and** merges these changes into the currently checked out local branch on computer `b` (in this case branch `dev`). This operation should normally be a 'fast-forward' (so no merge conflicts)
4. `git checkout feature_branch`
5. `git merge dev` - this merges changes from `b`'s local `dev` to the `feature_branch`.
6. `git mergetool` - resolve conflicts
7. `git commit` - commit your merge

With this option `b`'s both local `dev` and `feature_branch` have latest changes.

### Option 2, fetch

1. Commit all changes to branch `feature_branch` (git status shows `clean`)
2. `git fetch origin dev` - this downloads latest changes to `dev`, but *doesn't* merge them to local `dev`
3. `git merge origin/dev` - this merges changes from the downloaded version of `dev` to the `feature_branch`.

In this scenario `b`'s local `feature_branch` will have the most recent changes from `dev` *as they are on the remote repo* and their local `dev` will not have these changes. This is OK, since `b` isn't working on `dev`, (s)he's working on `feature_branch`.

## go back to previous commit

- `git reset --hard <commit_number>`
    
    this will delete all changes u made since this commit, make sure git status is clean, else if u dont require those changes do it.

## remove a file (not commited)
```
# remove from both staging area and local filesystem
git rm unwanted-file.txt

# remove from staging area only and not local filesystem
git rm --cached unwanted-file.txt
```

## remove locally commited file
To remove the last two commits locally without reverting the changes in the files, you can use the `git reset` command. This will reset your branch to the state before the last two commits while keeping your working directory unchanged.

Here are the steps:

1. Use `git reset` to move the HEAD pointer back by two commits.
2. Use the `--soft` option to keep the changes in the staging area.

`git reset --soft HEAD~2`

This command effectively undoes the last two commits but leaves your changes in the working directory and staging area, allowing you to recommit them if needed.


## other tidbits

- Creates a new branch, with no history or contents, called gh-pages, and switches to the gh-pages branch
	`$ git checkout --orphan gh-pages`

- add files
	 `git add .` - adds all changed files
	 `git add somepath/` - adds all changed files after somepath
	 `git add -u .` - adds all tracked files with modifications. will leave untracked files.
	 
--- 

## create a new branch 

**usual**: `git checkout new_branch`
**without commit history**: `git checkout --orphan new_branch`


## Git Workflow and pull strategy Guidelines 
**Branching Workflow** 
1. **Base Branch:**  
• dev is the most current branch where all new feature branches originate.  
2. **Feature Branch Workflow:**  
• Developers create branches from dev, work on their feature, and push their branch to the remote (GitHub).  
3. **Pull Requests:**  
• Once a feature is complete, developers create a pull request to merge into dev.  
4. **Production Deployment:**  
• Files/folders are cherry-picked from dev to production for deployment.

**Merge vs Rebase Strategies** 
1. **Keeping Feature Branches Updated with** dev**:**  
• **Use Rebase**:  
• Rebase the feature branch onto dev to keep it current (git pull --rebase).  
• This avoids unnecessary merge commits and ensures a linear history.  
• **Important Note:** Only rebase private (unshared) feature branches to avoid rewriting public history.  
2. **Merging Feature Branches into** dev**:**  
• **Use Merge**:  
• Merge feature branches into dev via pull requests to preserve the commit history.  
• Use squash commits if there are too many small or redundant commits.  
3. **Pushing to** production**:**  
• **Use Cherry-Picking**:  
• Cherry-pick specific commits or files from dev to production for controlled and selective deployment.

**GitHub Configuration** 
1. **Repository Settings:**  
• Enable these options under the **Merge Button** section:  
• **Rebase and Merge** for clean, linear history.  
• **Squash and Merge** for single-commit merges (use sparingly).  
• **Merge Commits** for context preservation during major integrations.  
• Disable unnecessary merge methods if you want to enforce a specific workflow.  
2. **Branch Protection Rules:**  
• Require feature branches to be up-to-date with dev before merging.  
• Enforce CI checks to ensure code quality and prevent broken merges.

**Git Blame Considerations** 
1. **Rebase:**  
• Maintains clean history, but conflict resolutions during rebasing attribute those lines to the rebaser.  
2. **Merge Commits:**  
• Preserves all commit history and accurately reflects authorship for git blame.  
3. **Squash Commits:**  
• Combines all changes into one commit, attributing all changes to the squasher, which may obscure authorship in git blame.

**Best Practices** 
1. **During Development:**  
• Use git pull --rebase to stay updated with dev.  
• Regularly rebase private branches to avoid conflicts during pull requests.  
2. **Merging to** dev**:**  
• Use merge commits to preserve history.  
• Avoid squashing large, multi-contributor feature branches if git blame accuracy is needed.  
3. **Production Deployment:**  
• Use cherry-picking for selective deployment of tested features.  
4. **Collaboration and Automation:**  
• Conduct code reviews during pull requests to ensure quality.  
• Use GitHub Actions or CI to enforce rebasing and ensure branches are current before merging.By following these guidelines, you can balance a clean Git history, accurate git blame, and efficient collaboration.

In summary  

- When pulling changes from dev use rebase
- When making a PR to dev user merge

> one thing ive noted is if i push changes to my remote and then pull from dev then rebasing causes remote and local to diverge. this then requires manually merging possible conflicts in each commit, so ig we should always pull dev --rebase before we push any changes. (edited) 


## how to squash commits

Squashing commits involves combining multiple commits into a single one. This is typically done to clean up a commit history before merging branches. Here’s how to squash commits using Git:

1. Interactive Rebase

Use an interactive rebase to squash commits. For example, if you want to squash the last three commits:

`git rebase -i HEAD~3`

2. Mark Commits for Squashing

This opens a text editor showing the commits:

```
pick abc123 Commit message 1
pick def456 Commit message 2
pick ghi789 Commit message 3
```

•	Replace pick with squash (or s) for the commits you want to squash into the previous one:

```
pick abc123 Commit message 1
squash def456 Commit message 2
squash ghi789 Commit message 3
```

3. Edit Commit Messages
After saving, Git will prompt you to edit the combined commit message. You can edit it to summarize all changes or leave it as is:
```
# This is a combination of commits.
# The first commit's message is:
Commit message 1

# The following commit messages will also be included:
Commit message 2
Commit message 3
```

Make your changes, then save and close the editor.

4. Finish the Rebase (only If there are conflicts)
Git applies the changes, combining the commits. If there are conflicts, Git will pause the rebase and allow you to resolve them. Once resolved, continue the rebase:

`git rebase --continue`

5. Push Changes
If you’ve already pushed the commits to a remote repository, force push the branch to update it:

`git push --force`

Notes:
- Squashing is best done on local branches or before merging into shared branches, as rewriting commit history affects collaborators.
- Use git log or git reflog to verify the changes after squashing.

## debugging

- Missing or invalid credentials
	```
	Missing or invalid credentials. Error: connect ECONNREFUSED /run/user/337388563/vscode-git-f0e90613a7.sock at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1595:16) { errno: -111, code: 'ECONNREFUSED', syscall: 'connect', address: '/run/user/337388563/vscode-git-f0e90613a7.sock' } Missing or invalid credentials. Error: connect ECONNREFUSED /run/user/337388563/vscode-git-f0e90613a7.sock at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1595:16) { errno: -111, code: 'ECONNREFUSED', syscall: 'connect', address: '/run/user/337388563/vscode-git-f0e90613a7.sock' } remote: Invalid username or password. fatal: Authentication failed for '[https://github.com/FarMart-Tech/data_pipelines.git/](https://github.com/FarMart-Tech/data_pipelines.git/)'
	```

	close and restart terminal


## git reset

### diff bw git reset hard soft etc when to use what?
I'll explain the key differences between git reset types and when to use each one:

#### 1. `git reset --soft`
- Moves HEAD to specified commit
- Does NOT touch staging area (index)
- Does NOT touch working directory
- **When to use**: When you want to undo commits but keep all your changes staged
- **Common use case**: When you want to squash multiple commits into one
```bash
git reset --soft HEAD~1  # Undo last commit, keep changes staged
```

#### 2. `git reset --mixed` (default)
- Moves HEAD to specified commit
- Updates staging area to match repository
- Does NOT touch working directory
- **When to use**: When you want to undo commits and unstage changes, but keep them in working directory
- **Common use case**: When you want to reorganize your changes into different commits
```bash
git reset HEAD~1  # Undo last commit, unstage changes
```

#### 3. `git reset --hard`
- Moves HEAD to specified commit
- Updates staging area to match repository
- Updates working directory to match repository  
- **When to use**: When you want to completely discard commits and changes
- **Common use case**: When you want to abandon local changes and go back to a previous state
- ⚠️ **WARNING**: This is destructive! Changes are permanently lost
```bash
git reset --hard HEAD~1  # Undo last commit and discard all changes
```
- **if u want to change the actual files u have use this, remote still has latest assuming u pushed/stashed it**


#### Quick Reference Table:
| Type | HEAD | Staging Area | Working Directory | Safety Level |
|------|------|--------------|-------------------|--------------|
| --soft | ✅ Changes | ❌ Unchanged | ❌ Unchanged | Safest |
| --mixed | ✅ Changes | ✅ Changes | ❌ Unchanged | Safe |
| --hard | ✅ Changes | ✅ Changes | ✅ Changes | Dangerous |

#### Additional Tips:
1. If you're unsure, start with `--soft` or `--mixed` as they're safer
2. Before using `--hard`, consider creating a backup branch
3. You can always use `git reflog` to recover from accidental hard resets
4. For public/shared branches, prefer `git revert` over `git reset`


## git directories: working vs staging vs local vs remote



Let me break down the different areas in Git and explain how they relate to each other:

### Git Areas/Levels

1. **Working Directory** (Working Tree)
- Where you actually edit your files
- Untracked and modified files live here
- What you see in your file system
```bash
# See working directory changes
git status
```

2. **Staging Area** (Index)
- Preparation area for your next commit
- Files you've added but haven't committed yet
```bash
# Add files to staging
git add file.txt
```

3. **Local Repository**
- Contains all your commits
- Lives in the `.git` folder
- Full history of your project
```bash
# Commit staged changes to local repo
git commit -m "message"
```

4. **Remote Repository**
- The "cloud version" (like GitHub/GitLab)
- Separate copy of your repository
- Shared with other developers
```bash
# Push local commits to remote
git push origin main
```

### Flow Diagram
```
Working Directory → Staging Area → Local Repository → Remote Repository
    (git add)         (git commit)        (git push)
```

### Important Points About Remote

1. **Git reset DOES NOT change remote**
- `git reset` only affects your local repository
- Remote stays unchanged until you force push
```bash
# This only changes local
git reset --hard HEAD~1

# This would be needed to change remote (dangerous!)
git push --force origin main
```

2. **Syncing with Remote**
- `git fetch` downloads changes but doesn't merge them
- `git pull` = `git fetch` + `git merge`
```bash
# Just download changes
git fetch origin

# Download and merge changes
git pull origin main
```

3. **Common Remote Operations**
```bash
# See remote status
git remote -v

# Add remote
git remote add origin URL

# Get changes from remote
git pull origin main

# Send changes to remote
git push origin main
```

### Important Concepts

1. **Matching to Remote means:**
- Making your local branch match exactly what's on remote
- Can be done with:
```bash
# Safe way (recommended)
git fetch origin
git reset --hard origin/main

# Or pull with reset
git pull --reset origin main
```

2. **Remote Safety**
- Remote is generally considered the "source of truth"
- Changing remote history (force push) can cause problems for team
- Always be careful with `--force` commands

3. **Best Practices**
- Always pull before pushing
- Use branches for new features
- Don't force push to shared branches
- Keep local and remote in sync regularly

4. **Recovery Options**
```bash
# See history of all actions (including resets)
git reflog

# Recover deleted commits
git reset --hard HEAD@{1}
```

Remember:
- Local changes (add, commit, reset) don't affect remote
- Remote changes require explicit push/pull
- Force pushing can overwrite remote history (dangerous!)
- When in doubt, create a backup branch before major operations



## inside .git: the local repository

Okay, let's break down the contents of the `.git` directory based on the file listing you provided. This directory is the heart of your Git repository, containing all the necessary information to track your project's history and manage changes.

Here's an explanation of each item:

### Files

1.  **`COMMIT_EDITMSG`**
    *   **Type:** File
    *   **Purpose:** Stores the commit message of the most recent commit. This is used when you're amending a commit or when Git opens an editor for you to write a commit message.
    *   **Content:** Text of the commit message.

2.  **`FETCH_HEAD`**
    *   **Type:** File
    *   **Purpose:** Stores the commit information of the last `git fetch` operation. It keeps track of the remote branches that were fetched.
    *   **Content:** References to remote branches and their commit IDs.

3.  **`HEAD`**
    *   **Type:** File
    *   **Purpose:** A symbolic reference that points to the currently checked-out branch or commit. It's a pointer to the current working state.
    *   **Content:** Usually contains a reference to a branch (e.g., `ref: refs/heads/main`) or a commit ID in detached HEAD state.

4.  **`ORIG_HEAD`**
    *   **Type:** File
    *   **Purpose:** Stores the previous value of `HEAD` before a potentially destructive operation (like `git reset`). It's used for recovery.
    *   **Content:** A commit ID.

5.  **`REBASE_HEAD`**
    *   **Type:** File
    *   **Purpose:** Used during a rebase operation. It stores the commit ID of the branch you're rebasing onto.
    *   **Content:** A commit ID.

6.  **`config`**
    *   **Type:** File
    *   **Purpose:** Contains configuration settings for your Git repository. This includes remote URLs, user information, and other settings.
    *   **Content:** Text-based configuration data.

7.  **`description`**
    *   **Type:** File
    *   **Purpose:** Contains a description of the repository. This is often used by Git hosting services.
    *   **Content:** A short text description.

8.  **`index`**
    *   **Type:** File
    *   **Purpose:** This is the staging area (also known as the index). It stores the changes that are staged for the next commit.
    *   **Content:** Binary data representing the staged changes.

### Directories

1.  **`hooks/`**
    *   **Type:** Directory
    *   **Purpose:** Contains scripts that Git executes before or after certain events (e.g., commit, push). These are used for automation and customization.
    *   **Content:** Example scripts and custom scripts.

2.  **`info/`**
    *   **Type:** Directory
    *   **Purpose:** Contains information files for the repository.
    *   **Content:** Usually contains a file called `exclude` which lists files that should be ignored by Git.

3.  **`logs/`**
    *   **Type:** Directory
    *   **Purpose:** Stores logs of changes to references (branches, tags). This is used by `git reflog`.
    *   **Content:** Log files for each reference.

4.  **`objects/`**
    *   **Type:** Directory
    *   **Purpose:** Stores all the Git objects (commits, trees, blobs) in a compressed format. This is where the actual data of your repository is stored.
    *   **Content:** Subdirectories and files representing Git objects.

5.  **`refs/`**
    *   **Type:** Directory
    *   **Purpose:** Stores references to commits, including branches and tags.
    *   **Content:**
        *   `heads/`: Contains files representing local branches.
        *   `tags/`: Contains files representing tags.
        *   `remotes/`: Contains files representing remote branches.

### Summary

The `.git` directory is a complex structure that Git uses to manage your project's history and changes. It contains files and directories that store configuration, commit messages, references, and the actual data of your repository. Understanding the contents of this directory can help you better understand how Git works under the hood.


