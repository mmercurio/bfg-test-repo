# bfg-test-repo
A safe place to test a dangerously powerful weapon, the [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/).

See BFG documentation for usage. I plan to create some files in `main` branch and other branches, and then use them bfg to delete them from existance.

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

Contents of `files/ unmerged_secrets.txt`:

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
Author: Michael Mercurio <***REMOVED***>
Date:   Fri Aug 30 11:22:02 2024 -0400

    rm leaked secret
```

Then we can easily see what was deleted:

```shell
$ git show 2e820f8465946a2e4c2f6b0d8fdce80cd12fdd47
commit 2e820f8465946a2e4c2f6b0d8fdce80cd12fdd47 (HEAD -> main, origin/main, origin/HEAD)
Author: Michael Mercurio <***REMOVED***>
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
