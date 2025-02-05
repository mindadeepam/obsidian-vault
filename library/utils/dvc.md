Instead of using bucket storage and hard links for storing large files (GitHub has a limit for max filesize), we use a tool called [dvc](https://dvc.org/doc). Its interface is similar to git and hence will feel natural to use. Below is a 0-1 guide on how to use it.

## What is DVC?

DVC is an open-source version control system for machine learning projects. It's designed to handle large files, data sets, and machine learning models, working alongside Git.

At the time of writing we only use it for data and model artifacts versioning, think of it as _git for large files_.

## Installation

```
uv pip install dvc
```

# Basic Commands

## #1. Initialize DVC

Start by initializing dvc in your project using [dvc init](https://dvc.org/doc/command-reference/init). for initializing dvc in a subdirectory (like in a mono repo), cd into the sub dir and use [dvc init —subdir](https://dvc.org/doc/command-reference/init#initializing-dvc-in-subdirectories).

```
cd path/to/project_repo 
dvc init.  
```

This creates a `.dvc` directory to store configuration and cache.

## #2. Adding Data

The [dvc add](https://dvc.org/doc/command-reference/add) command is used to add a folder to dvc. (just like git add)

```
# use this so dvc commit is performed automatically when added.
dvc config core.autostage true 


# lets say data folder contains `train` and `test` subdirectories

# add data folder to dvc
dvc add data 
```

This creates a `data.dvc` file that tracks your `data` folder, similar to Git tracking source code. The data has been added to the **local cache**, which is a storage location managed by DVC to keep a copy of the tracked data. The .dvc file acts as a pointer to this cached version and includes metadata like the file’s hash, size, and structure. **Note that data is not backed up remotely yet.**

This file looks something like this:

```
outs:
- md5: 47c9c09578e264749a54f265b6a6ab51.dir
  size: 272915818
  nfiles: 8
  hash: md5
  path: models
```

The md5 checksum is calculated using the data and hence represents the state of the directory. DVC uses this to identify changes. (just like git commit hashes).

**Similarly**,

```
dvc add models  # same as `dvc add models.dvc`
```

This creates a `models.dvc` file that tracks your `models` directory, similar to Git tracking source code.

**And, you can also add sub-directories to dvc using something**

```
dvc add data/train
```

This creates a `train.dvc` and `.gitignore` file _inside the data folder._

## #3. Back up data to Remote Storage

```
# Add a remote storage
dvc remote add -d myremote s3://mybucket/path

# Or for local storage
dvc remote add -d myremote /path/to/storage
```

**Now to back up the local cache to remote storage:**

```
# Push data to remote
dvc push

# push specific directory (recommended for faster downloads/uploads)
dvc push data/train 
dvc push models

# ...
```

## #4. Tracking Changes

We have backed up your data and models after an experiment to dvc. But you might have noticed we haven’t tagged our data yet; i.e. our data is backed up but we don’t have tracking yet. To do that we use git.

**Data is backed up using dvc but tracked using git.**

```
git add data.dvc
git commit -m "added the data folder"    # suppose commit-hash generated is cmt00
git push
```

Now the data is backed up as well as indexed by a commit hash. we can now retrieve this data anytime we want using its git commit hash.

## #5. Pulling Data

Now let’s say we make a change,

```
# we want to remove test data
rm data/test

# now add new data folder state to dvc and push it
dvc add data
dvc push data

# tag it with a git commit
git add data.dvc. # this is usually done automatically 

git commit -m "removed test data"       # commit hash generated: cmt01
git push

rm .dvc/cache/  # removing the local cache as well just for illustration...
```

Now, to get back test and train data as in commit `cmt00`:

```
# chekcout the relevant .dvc file from git
git checkout cmt00 -- data.dvc

# now pull this version of data from the remote
dvc pull data
```

And _voila,_ you have your earlier version back!

You can not selectively pull subdirectories of directories added to dvc, for eg

- clear cache using `rm -r .dvc/cache`
    
- `dvc pull data/test` : this pulls the entire data folder contents as referenced by the current .dvc file in the local cache folder. You will only see the `test` folder inside `data/` , but the local cache contains everything. This will slow down the pulls of data of size >3 GBs (more than 5 mins). So if the size of the data is very large it is recommended to find ways to optimize this.
    

You can also get the data across all the directories backed up by dvc files using current .dvc files using `dvc pull`.

---

DVC is also used to track ML experiments. However, that is not covered here. Refer to the documentation for more on that.



# dvc merge issue
[google search](https://www.google.com/search?q=how+to+do+dvc+merge&sca_esv=0cd5d31f344aa732&ei=F0-TZ4vXGK_z4-EPqpuJ-A0&ved=0ahUKEwjLvtuM-I2LAxWv-TgGHapNAt8Q4dUDCBA&uact=5&oq=how+to+do+dvc+merge&gs_lp=Egxnd3Mtd2l6LXNlcnAiE2hvdyB0byBkbyBkdmMgbWVyZ2UyBxAAGLADGB4yBxAAGLADGB4yDhAAGIAEGLADGIYDGIoFMg4QABiABBiwAxiGAxiKBTIOEAAYgAQYsAMYhgMYigUyDhAAGIAEGLADGIYDGIoFMgsQABiABBiwAxiiBDIIEAAYsAMY7wUyCBAAGLADGO8FMgsQABiABBiwAxiiBEjbDlDIBVjHDXABeACQAQCYAckBoAGNDKoBBTAuOC4xuAEDyAEA-AEBmAIBoAICmAMAiAYBkAYKkgcBMaAH2yo&sclient=gws-wiz-serp)
## Additional Resources

- [Official DVC Documentation](https://dvc.org/doc)
    
- [DVC GitHub Repository](https://github.com/iterative/dvc)
    
- [DVC Tutorial](https://dvc.org/doc/tutorial)
    

This guide covers the basics of DVC. For more detailed information, refer to the official documentation




