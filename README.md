# bfg-test-repo
A safe place to test a dangerously powerful weapon, the [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/).

See BFG documentation for usage. I plan to create some files in `main` branch and other branches, and then use them bfg to delete them from existence.

- [Initial state](#initial-state)
- [Naive clean-up](#naive-clean-up)
- [Using BFG to rewrite history](#using-bfg-to-rewrite-history)
  - [Before Pushing Dangerous Changes](#before-pushing-dangerous-changes)

## Initial state

Before trying to remove any files, this is the state of the repo:

On branch, `main`:

```
├── README.md
└── files
    └── bfg_deleted.md
```

Contents of `files/bfg_deleted.md`:

> # This File Does not Exist
>
> After this file is merged to a PR branch, pushed to the remote, and then merged to the `main`  branch, it will be deleted from existence using `bfg`.
>
> It's my expectation that there will be no evidenve of the contents of this file. For example, pretend that it accidentally contained some senstive information such as a secure API key.
>
>
> ```
> QAx9x1aMzcLdvt8aAcKgrmZKfVYEDfk2r1NDzYMVA1rm5FN60Qxh8e94wtDx7NdngFy8XX8PwhQmT69uxVG41MTByeVfRg73ivKD
> ```
>
> Oh no. What now?



On branch, `unmerged`:

```
├── README.md
└── files
    ├── bfg_deleted.md
    └── unmerged_secrets.txt
```

Changes to  `files/bfg_deleted.md`:

```diff
--- a/files/bfg_deleted.md
+++ b/files/bfg_deleted.md
@@ -9,3 +9,5 @@ QAx9x1aMzcLdvt8aAcKgrmZKfVYEDfk2r1NDzYMVA1rm5FN60Qxh8e94wtDx7NdngFy8XX8PwhQmT69u

 Oh no. What now?
+
+Just to make it more interesting. This file is being modified in another branch that is not merged, but pushed to the remote.
```

Contents of `files/unmerged_secrets.txt`:

```
This file potentially contains some secret information that has
been commited to an unmerged branch.

Let's assume it was realized it shouldn't have been commited
before it was merged to `main`. But it still needs to be deleted.

Oh look, here's some secret info:

LgouCi4KICAgICAgICAgVE9PIE1BTlkgU0VDUkVUUwouCi4KLgouCi4KM3QjLUQt
QE1AJW1nLjdiMmpWN21yLHh0Kkw+QWpxNXN2THQ4WHhdcyx3QjEpUEIsQXpCZzZR
eHk9b2VQQm5dPTUjNykyMGFdJWVOaV5iXVlMYUA5QWpLXVdRdk04clkzSy0yIQo=

Let's hope this secret doesn't fall into the wrong hands.
```

## Naive clean-up

First, let's try using git to remove the offending file on `main` like this:

```shell
$ git checkout main
Your branch is up to date with 'origin/main'.

$ git rm files/bfg_deleted.md
rm 'files/bfg_deleted.md'

$ git commit -m "rm leaked secret"
[main 2e820f8] rm leaked secret
 1 file changed, 11 deletions(-)
 delete mode 100644 files/bfg_deleted.md

$ git push
git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 10 threads
Compressing objects: 100% (1/1), done.
Writing objects: 100% (2/2), 467 bytes | 467.00 KiB/s, done.
Total 2 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:mmercurio/bfg-test-repo.git
   b3710d5..2e820f8  main -> main
```

This is considerd naive because the potentially leaked secret is still available in the git commit history.

For example:

```shell
$ git log -n1
commit 2e820f8465946a2e4c2f6b0d8fdce80cd12fdd47 (HEAD -> main, origin/main, origin/HEAD)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 11:22:02 2024 -0400

    rm leaked secret
```

Then we can easily see what was deleted:

```shell
$ git show 2e820f8465946a2e4c2f6b0d8fdce80cd12fdd47
commit 2e820f8465946a2e4c2f6b0d8fdce80cd12fdd47 (HEAD -> main, origin/main, origin/HEAD)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 11:22:02 2024 -0400

    rm leaked secret

diff --git a/files/bfg_deleted.md b/files/bfg_deleted.md
deleted file mode 100644
index b2b8ed9..0000000
--- a/files/bfg_deleted.md
+++ /dev/null
@@ -1,11 +0,0 @@
-# This File Does not Exist
-
-After this file is merged to a PR branch, pushed to the remote, and then merged to the `main` branch, it will be deleted from existence using `bfg`.
-
-It's my expectation that there will be no evidenve of the contents of this file. For example, pretend that it accidentally contained some senstive information such as a secure API key.
-
-```
-QAx9x1aMzcLdvt8aAcKgrmZKfVYEDfk2r1NDzYMVA1rm5FN60Qxh8e94wtDx7NdngFy8XX8PwhQmT69uxVG41MTByeVfRg73ivKD
-``
-
-Oh no. What now?
```

Oops.


## Using BFG to rewrite history

Based on documentation [here](https://rtyley.github.io/bfg-repo-cleaner/), using the following bfg commands:

```shell
git clone --mirror git@github.com:mmercurio/bfg-test-repo.git
bfg --delete-files bfg_deleted.md bfg-test-repo.git
bfg --delete-files unmerged_secrets.txt bfg-test-repo.git
```

Followed by:

```shell
cd bfg-test-repo.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push
```

Note, that the `git push` will result in errors pushing some of the refs because of how it rewrites history and should be considered ***extremely dangerous***! You should first backup the entire repo before attempting to use bfg or similar tools to write history like this.

Example of errors when pushing (even if using `push --force`):

```shell
$ git push
Enumerating objects: 13, done.
Writing objects: 100% (13/13), 3.12 KiB | 3.12 MiB/s, done.
Total 13 (delta 0), reused 0 (delta 0), pack-reused 13 (from 1)
remote: Resolving deltas: 100% (6/6), done.
To github.com:mmercurio/bfg-test-repo.git
 + 2e820f8...4ad9cf6 main -> main (forced update)
 + 8745fbf...c3cbac1 newbranch -> newbranch (forced update)
 + ee5188f...22d2b27 readme -> readme (forced update)
 + 853c259...c13d9ea staging -> staging (forced update)
 ! [remote rejected] refs/pull/1/head -> refs/pull/1/head (deny updating a hidden ref)
error: failed to push some refs to 'github.com:mmercurio/bfg-test-repo.git'

$ git push --force
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:mmercurio/bfg-test-repo.git
 ! [remote rejected] refs/pull/1/head -> refs/pull/1/head (deny updating a hidden ref)
error: failed to push some refs to 'github.com:mmercurio/bfg-test-repo.git'
```

At this point the offending files are gone. They no longer appear in any of the commit history.

> [!NOTE]
> BFG does not modify history of the latest commit by default. If this is required, either commit a change to remove the offending content and then use BFG to remove the content from the commit history (my preference), or use the `--no-blob-protection` option with BFG. However, the BFG help indicates the `--no-blob-protection` is *NOT RECOMMNEDED to rewrite the lastest commit*. So, maybe don't do that.

### Before Pushing Dangerous Changes

Before attempting to push changes that have the potential to destroy your repo I strongly recommend taking two additional actions:

1. Backup the contents of the repo to a safe and secure location (e.g., a secure encrypted disk).
1. Make a local clone of the repo and examine the contents carefully. This can be done using the following:

```shell
$ mkdir local_cone
$ cd local_clone
$ git clone --local ../bfg-test-repo.git
Cloning into 'bfg-test-repo'...
done.
```

You now have a fresh clone of the repo state prior to pushing the changes. Examine the contents carefully before proceeding to push to the remote.

*"With great power comes great responsibility."*
