# git-practice

trying git commands on a disposable repo.

BTW, if you're here trying to understand git better, I highly  recommend this lecture:
- [MIT - The Missing Semester: Lecture 6: Version Control (git)](https://www.youtube.com/watch?v=2sjqTHE0zok)


## Remove all history

this seemed to work nicely:

`git update-ref -d HEAD`

> This command deletes the branch we are in. If we are in main, this command will delete the main branch, and all the commits. - [https://candost.blog/how-to-reset-first-commit-in-git/](https://candost.blog/how-to-reset-first-commit-in-git/)

I ran after 3 commits and ended up with this (after `git push -f`):

```
commit 35b536c2281d43d99963d0a62c933a04d53ad117 (HEAD -> main, origin/main, origin/HEAD)
Author: PostgreSqlStan <91010505+PostgreSqlStan@users.noreply.github.com>
Date:   Thu Feb 2 00:19:55 2023 -0600

    tried: git update-ref -d HEAD
```

*Won't waste time trying the various other methods I've seen mentioned.*

## orphan asset branch

[Storing image assets in your repo and referencing in markdown](https://gist.github.com/mcroach/811c8308f4bd78570918844258841942)


```
git checkout --orphan assets
git reset --hard
cp ~/Documents/fav_photo/nap-time.jpeg
git add .
git commit -m 'Added cat picture'
git push -u origin assets
```

`* 78b11b3 (HEAD -> assets, origin/assets) Added cat picture - PostgreSqlStan, 23 seconds ago`

![cat pic?](https://github.com/postgresqlstan/git-practice/blob/assets/nap-time.jpeg)


**Notes:** This works. Furthermore, `reset --hard` seems better than the `rm -rf` method I've seen elsewhere. However, it only deletes the working directory and staging area, which would not be appropriate for actually removing confidential information you might have committed.

The old commits are still accessible:

```
git-practice % git reflog
58e41b9 (HEAD -> main, origin/main, origin/HEAD) HEAD@{0}: commit: removed test dir
ee3ec50 HEAD@{1}: commit: ditto
7dc65bc HEAD@{2}: commit: ditto
3926c3e HEAD@{3}: commit: ditto
56997b9 HEAD@{4}: commit: updated readme
a528620 HEAD@{5}: commit: testing asset branch link
d7f50ff HEAD@{6}: checkout: moving from assets to main
78b11b3 (origin/assets, assets) HEAD@{7}: commit (initial): Added cat picture
d7f50ff HEAD@{8}: commit: preparing second experiment
35b536c HEAD@{9}: commit (initial): tried: git update-ref -d HEAD
f77cdd7 HEAD@{11}: commit: 3rd commit
9db8d5d HEAD@{12}: commit: 2nd commit
822e0a0 HEAD@{13}: clone: from github.com:PostgreSqlStan/git-practice.git
```

## Remove all history - method 2

On second thought, I'd like to know how to erase the entire history, preferably with using `rm -rf`, so I'll try another method.

[Clear Git master branch history in the Git server](https://www.systutorials.com/how-to-clear-git-history-in-local-and-remote-branches/):

```
git checkout --orphan tmp-main # create a temporary branch
git add -A  # Add all files and commit them
git commit -m 'Add files'
git branch -D main # Deletes the master branch
git branch -m main # Rename the current branch to master
git push -f origin main # Force push master branch to Git server
```

Afterwards, my (entire) log looks like this:

```
% git log
commit 5f65e1c53d704841c3763a1e57b9b45be9f3842e (HEAD -> main, origin/main, origin/HEAD)
Author: PostgreSqlStan <91010505+PostgreSqlStan@users.noreply.github.com>
Date:   Thu Feb 2 17:59:34 2023 -0600

    (re)-add all
```

I'm curious if the `reflog` will still show the old commits.

```
% git reflog
5f65e1c (HEAD -> main, origin/main, origin/HEAD) HEAD@{0}: Branch: renamed refs/heads/tmp-master to refs/heads/main
5f65e1c (HEAD -> main, origin/main, origin/HEAD) HEAD@{2}: commit (initial): (re)-add all
792bc64 HEAD@{3}: commit: adding a new method to try
2d18de4 HEAD@{4}: commit: ditto
58e41b9 HEAD@{5}: commit: removed test dir
ee3ec50 HEAD@{6}: commit: ditto
7dc65bc HEAD@{7}: commit: ditto
3926c3e HEAD@{8}: commit: ditto
56997b9 HEAD@{9}: commit: updated readme
a528620 HEAD@{10}: commit: testing asset branch link
d7f50ff HEAD@{11}: checkout: moving from assets to main
78b11b3 (origin/assets, assets) HEAD@{12}: commit (initial): Added cat picture
d7f50ff HEAD@{13}: commit: preparing second experiment
35b536c HEAD@{14}: commit (initial): tried: git update-ref -d HEAD
f77cdd7 HEAD@{16}: commit: 3rd commit
9db8d5d HEAD@{17}: commit: 2nd commit
822e0a0 HEAD@{18}: clone: from github.com:PostgreSqlStan/git-practice.git
```

*It does.*

Oops, somehow my configuration has changed. I've finally broken something!

```
% git push
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main
```

Running the command mentioned in the error message fixes it.

I'm still not entirely sure what, if any, data fragments either of the methods might leave behind, but either seem to clean up the history in the log.

---

[Git Remove All Commits History](https://raturi.in/blog/cleaning-git-repository/):

This extra step convinced me (using the second method of clearing history) the files were actually removed:

```
git gc --aggressive --prune=all # remove the old files
```

---

OK, continuing experiments on another, larger repo.

```
`git update-ref -d HEAD`
git add .
git commit -m 'initial (re)commit'
```

It reduced the log to a single entry:
```
git log
commit 9fb31fce124b5448ff9c77f885767a4d100d826f (HEAD -> main)
Author: PostgreSqlStan <91010505+PostgreSqlStan@users.noreply.github.com>
Date:   Fri Feb 3 14:30:58 2023 -0600

    initial (re)commit
```

Then, the `gc` pruning command reduced the size of the `.git` directory (a lot):
```
pgtab_BK % du -sh .git
3.5M    .git

pgtab_BK % git gc --aggressive --prune=all
Enumerating objects: 766, done.
Counting objects: 100% (766/766), done.
Delta compression using up to 8 threads
Compressing objects: 100% (740/740), done.
Writing objects: 100% (766/766), done.
Total 766 (delta 486), reused 9 (delta 0), pack-reused 0

pgtab_BK % du -sh .git
668K    .git
```

I'm still sure what "secrets" might or might not be hidden in there, but it serves my purpose for now.
