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



## debugging

- Missing or invalid credentials
	```
	Missing or invalid credentials. Error: connect ECONNREFUSED /run/user/337388563/vscode-git-f0e90613a7.sock at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1595:16) { errno: -111, code: 'ECONNREFUSED', syscall: 'connect', address: '/run/user/337388563/vscode-git-f0e90613a7.sock' } Missing or invalid credentials. Error: connect ECONNREFUSED /run/user/337388563/vscode-git-f0e90613a7.sock at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1595:16) { errno: -111, code: 'ECONNREFUSED', syscall: 'connect', address: '/run/user/337388563/vscode-git-f0e90613a7.sock' } remote: Invalid username or password. fatal: Authentication failed for '[https://github.com/FarMart-Tech/data_pipelines.git/](https://github.com/FarMart-Tech/data_pipelines.git/)'
	```

	close and restart terminal