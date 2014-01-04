# git rebase V.S. git merge

## git分支介绍及其原理

git的一大特色是其分支理念，使用git的过程中总少不了使用分支（复杂度很低的项目版本除外）。

git的分支，本质上就是将某个commit标记为分支的HEAD, 这个commit的所有upstream commit就是这个分支的历史记录。
在一个分支上提交后，就是更新这个分支所指向的HEAD commit。
