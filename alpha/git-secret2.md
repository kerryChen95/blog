# git secret 2

## git stash
`git stash pop` 相当于 `git stash apply stash@{0}` 加 `git stash drop stash@{0}` ，如果apply失败，比如有冲突，那么不会执行drop，也就不会将stash从栈中清除。所以pop时如果有冲突，解决冲突后记得手动drop一下。
