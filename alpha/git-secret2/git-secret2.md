## 撤回你的误操作

## 介绍index区域与只提交一个文件中的部分修改

如果我们在工作目录中修改了多处，但只想commit一部分，怎么办呢？这就需要一个暂存区，来保存想要commit的部分。

![state change](img/state-change.jpg)

一图顶千言，index就表示这个暂存区域，添加进其中的修改就是将要commit的修改。

而通过 `git add [--patch|-p] [path]` 这个命令模式，可以将一个文件中的部分代码添加进index。
执行符合这个模式的命令，比如 `git add -p model.bone.js` ，git会把 `model.bone.js` 文件中的修改分割成一块一块的，然后交互式的问你需要将哪一块添加进index，如下图：

![git add --patch](img/add-patch.jpg)

你可以选择是否将这一块代码添加进index，或其他操作，甚至分割成更小的代码块，手动编辑当前代码块，从而实现把任意一行的修改添加进index（不过有时候git傲娇了，这时候说明你编辑的姿势不对）。

## git commit --allow-empty

在历史commit记录中插入commit


## 使用引用来指定某个commit



## fetch与pull

这两个命令，都会将远程的commit拉取下来，并用 `<remoteRepo>/<remoteBranch>` 和 `<remoteRepo>/HEAD` 引用所拉取的

- fetch：只是将远程的commit拉取下来，并

## log

## merge --no-ff

## commit --amend

## stash

如果你的工作目录和index还有修改没commit，却想checkout另一个分支，那么工作目录和index就会被覆盖，但你又不想先commit再checkout到另一个分支，比如你的代码还没写好，或者你其实想在另一个分支commit这些修改，怎么办呢？

`git stash --patch`

## 文件路径与参数名或子命令名相同怎么办
