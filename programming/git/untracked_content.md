
git status shows untracker content
------------------------------------

通常这种情况都出现在包含submodule的项目里，submodule里有文件变化没被提交上

    $ git status
    On branch master
    Untracked files:
      (use "git add <file>..." to include in what will be committed)

        I/am/singular(untracked content)

    nothing added to commit but untracked files present (use "git add" to track)

    $ cd I/am/singular
    $ git ac

回到原来的目录，就变成了(new commits)

    $ cd -
    $ git status
    Untracked files:
      (use "git add <file>..." to include in what will be committed)

        I/am/singular(new commits)

    nothing added to commit but untracked files present (use "git add" to track)

继续commit就行了

