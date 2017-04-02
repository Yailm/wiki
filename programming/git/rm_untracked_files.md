
rm untracked files
------------------------------------

- show what will be deleted by using -n option: (--dry-run)

      git clean -f -n

- clean files

      git clean -f

    - remove directories, `git clean -fd`
    - remove ignored files `git clean -fX`
    - remove ignored and non-ignored files, `git clean -fx`
