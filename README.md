# git-practice

trying git commands on a disposable repo.

BTW, if you're here trying to understand git better, I highly  recommend this lecture:
- [MIT - The Missing Semester: Lecture 6: Version Control (git)](https://www.youtube.com/watch?v=2sjqTHE0zok)


## Remove all history

I've used this method a couple of time to create a clean, pruned repo: [Clearing Git History in Local and Remote Branches](https://www.systutorials.com/how-to-clear-git-history-in-local-and-remote-branches/)

[edited: changed "master" to "main"]

```zsh
git checkout --orphan tmp-main # create a temporary branch
git add -A  # Add all files and commit them
git commit -m 'Add files'
git branch -D main # Deletes the master branch
git branch -m main # Rename the current branch to master
git push -f origin main # Force push master branch to Git server
```

Then finish the local cleanup:

```zsh
git branch --set-upstream-to=origin/main main # Local master tracks origin/master
git gc --aggressive --prune=all # remove the old files
```


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

