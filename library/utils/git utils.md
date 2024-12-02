# some common tasks and their commands

### git clone

`git clone https://github.com/<username>/<repo>.git`

### git clone using username and password(token)

`git clone https://username:password@github.com/username/repository.git`

### folder level git credentials in local

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

### push to github

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


### contribute to a repo

#### 1st time

1. clone the repo `git clone url/of/repo`
2. cd inside the repo
3. create your dev branch `git branch local_branch`
4. checkout to that dev branch `git checkout local_branch`
5. add the remote repository `git remote add origin url/of/repo`
6. create remote branch from your local branch `git push origin local_branch:remote_branch`
7. keep pushing to that remote_branch. raise PRs to dev.

#### then on

**how to pull latest changes from dev to personal_dev?**

It's a good practice to as soon as feasible after person *A* pushes the changes to `dev` for person *B* to get these changes into their branch `b`. This is so that person *B* works on latest code and their eventual merge to `dev` is easy.

#### Option 1, pull

1. Commit all changes to branch `feature_branch` (git status shows `clean`)
2. `git checkout dev`
3. `git pull` - this fetches (downloads) the changes onto computer `b` **and** merges these changes into the currently checked out local branch on computer `b` (in this case branch `dev`). This operation should normally be a 'fast-forward' (so no merge conflicts)
4. `git checkout feature_branch`
5. `git merge dev` - this merges changes from `b`'s local `dev` to the `feature_branch`.
6. `git mergetool` - resolve conflicts
7. `git commit` - commit your merge

With this option `b`'s both local `dev` and `feature_branch` have latest changes.

#### Option 2, fetch

1. Commit all changes to branch `feature_branch` (git status shows `clean`)
2. `git fetch origin dev` - this downloads latest changes to `dev`, but *doesn't* merge them to local `dev`
3. `git merge origin/dev` - this merges changes from the downloaded version of `dev` to the `feature_branch`.

In this scenario `b`'s local `feature_branch` will have the most recent changes from `dev` *as they are on the remote repo* and their local `dev` will not have these changes. This is OK, since `b` isn't working on `dev`, (s)he's working on `feature_branch`.

### go back to previous commit

- `git reset --hard <commit_number>`
    
    this will delete all changes u made since this commit, make sure git status is clean, else if u dont require those changes do it.

### remove a file (not commited)
```
# remove from both staging area and local filesystem
git rm unwanted-file.txt

# remove from staging area only and not local filesystem
git rm --cached unwanted-file.txt
```

### remove locally commited file
To remove the last two commits locally without reverting the changes in the files, you can use the `git reset` command. This will reset your branch to the state before the last two commits while keeping your working directory unchanged.

Here are the steps:

1. Use `git reset` to move the HEAD pointer back by two commits.
2. Use the `--soft` option to keep the changes in the staging area.

`git reset --soft HEAD~2`

This command effectively undoes the last two commits but leaves your changes in the working directory and staging area, allowing you to recommit them if needed.


### other tidbits

- Creates a new branch, with no history or contents, called gh-pages, and switches to the gh-pages branch
	`$ git checkout --orphan gh-pages`

- add files
	 `git add .` - adds all changed files
	 `git add somepath/` - adds all changed files after somepath
	 `git add -u .` - adds all tracked files with modifications. will leave untracked files.
	 
--- 

### create a new branch 

**usual**: `git checkout new_branch`
**without commit history**: `git checkout --orphan new_branch`


### Git Workflow and pull strategy Guidelines 
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


### how to squash commits

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

4. Finish the Rebase
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