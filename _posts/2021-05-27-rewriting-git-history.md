---
layout: post
title: "rewriting git history"
date: 2021-05-27 18:27:00 -0400
categories: git
---

# Rewriting Git History

This is actually a repost of an older post I had written.

> Prolouge

I recognized in my most recent git projects. I was using the wrong email / user information for my repositories. While annoying I figured this could be a learning event. Keep in mind rewriting git history in _most_ situations is a bad idea. However This is an opportunity to dive further into the depths of git.

## git basics

Lets start off by looking at how git handles version control.

Lets take a look at the `.git` folder found at the root of any git project.
```bash
.
├── branches
├── COMMIT_EDITMSG
├── config
├── description
├── HEAD
├── hooks
│   └── ...
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
├── objects
│   ├── ...
│   ├── info
│   └── pack
└── refs
    ├── heads
    ├── remotes
    └── tags
                                                           
```

Heres a brief overview for more detailed information see the [documentation](https://git-scm.com/docs/gitrepository-layout):
* branches/
    * mostly deprecrated
* COMMIT_EDITMSG
    * Essentially just a temporary file for storing the git commit message
* config
    *  contains all the settings and configuration values for a given git repository
    * shadows the global configuration file, such that any value that is not set here will default from the global configuration file
    * for more information see the [git config documentation](https://git-scm.com/docs/git-config)
* description
    * used mostly by githooks
    * not used by github, gitlab, ect
* HEAD
    * namespace that references the current active branch
* hooks/
    * git hook scripts
* index
    * the current index file for the repository
    * essentially all the data needed to make a git tree object
    * an awesome writeup can be found [here](https://mincong.io/2018/04/28/git-index/)
* logs
    * stores changes made to refs
* objects
    * object store for a given repositoryhttps://mincong.io/2018/04/28/git-index/
* refs
    * references for the repository such as tip-of-the-tree commit objects of a branch, tags, remotes, ect



## Rewriting the history

The simplest way I've found to rewrite a [git filter-branch](https://git-scm.com/docs/git-filter-branch). Basically this command can walk a list of branches and rewrite the history associated with them. This can be incredibily dangerous of course but since the repo I want to rewrite is owned solely by me it should be okay.


So first things first, lets rewrite the config file to use a local username and a local email. This will prevent (at least in this branch) from me using the wrong email and username. If I wanted to update these globally I could use `--global`
```bash
> $ git config user.name "jmp-rax"
> $ git config user.email pop.rax.jmp.rax@gmail.com
> $ cat .git/
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = git@github.com:jmp-rax/jmp-rax.github.io.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[branch "gh-pages"]
	remote = origin
	merge = refs/heads/gh-pages
[user]
	name = jmp-rax
	email = pop.rax.jmp.rax@gmail.com                                         
```


Next we need to go and rewrite the history.


I found a script that can be used for this purpose on [git-tower.com](https://www.git-tower.com/learn/git/faq/change-author-name-email/). So lets give this a whirl.

```bash
$ git filter-branch --env-filter '
WRONG_EMAIL="wrong@example.com"
NEW_NAME="New Name Value"
NEW_EMAIL="correct@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$NEW_NAME"
    export GIT_AUTHOR_EMAIL="$NEW_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags

```

Heres my (amended) git logs before

```
commit 5f69458a1980590cd3a2057a0ccc956d754e3ea2 (HEAD -> gh-pages, origin/gh-pages)
Author: My Name <myname@wrongemail.com>
Date:   Sun May 23 17:12:44 2021 -0400

    initial updates

commit 233e09bc30ca85a06ca0fb79de46fabf91a0c7ea
Author: My Name <myname@wrongemail.com>
Date:   Sun May 23 16:55:59 2021 -0400

    updating the _config.yml

commit e03d28942b61a9754938a118864d0c45fe5707f1
Author: My Name <myname@wrongemail.com>
Date:   Sun May 23 16:23:15 2021 -0400

    initial pages with jekyll

```

Here is the after. Notice the commit sha's changed, the history has completely been rewritten.

```bash
commit f49dbd61f19090e2c0f4abf9edc6085e21cd88cc (HEAD -> gh-pages)
Author: jmp-rax <pop.rax.jmp.rax@gmail.com>
Date:   Sun May 23 17:12:44 2021 -0400

    initial updates

commit b79d68217f42015f0a5902193c229e7112783be6
Author: jmp-rax <pop.rax.jmp.rax@gmail.com>
Date:   Sun May 23 16:55:59 2021 -0400

    updating the _config.yml

commit 57d1da70c0a300eab87f6f84cf2c41fd79fe45a0
Author: jmp-rax <pop.rax.jmp.rax@gmail.com>
Date:   Sun May 23 16:23:15 2021 -0400

    initial pages with jekyll
```

Now if I perform a `git push` we will see something like the following:
```bash
> $ git push
To github.com:jmp-rax/jmp-rax.github.io.git
 ! [rejected]        gh-pages -> gh-pages (non-fast-forward)
error: failed to push some refs to 'git@github.com:jmp-rax/jmp-rax.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

This is because since I rewrote all of the history our git history locally. Now the remote and our local branch are incompatible with one another. I will have to use the `--force` flag to push the repo regardless of what the remote thinks. If someone else had my repo checked out and I were to do this bad things can happen. Remember to apply the changes on all the branches using `--all`. In the example below I forgot to do that and had to push the master branch seperately.

```bash
> $ git push --force                                                                     
Enumerating objects: 23, done.
Counting objects: 100% (23/23), done.
Delta compression using up to 4 threads
Compressing objects: 100% (22/22), done.
Writing objects: 100% (23/23), 6.92 KiB | 708.00 KiB/s, done.
Total 23 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), done.
To github.com:jmp-rax/jmp-rax.github.io.git
 + 5f69458...f49dbd6 gh-pages -> gh-pages (forced update)
                                                            
```