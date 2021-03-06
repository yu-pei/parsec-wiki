This is a local copy of the Open MPI best practices page https://github.com/open-mpi/ompi/wiki/GitBestPractices.

For a nice, relatively brief overview of Git's basic usage, please see [the Pro Git book by Scott Chacon, available for free online](http://git-scm.com/book).  If you want to understand the nitty-gritty, check out [Git From the Bottom Up](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf).  There's also a fantastic blog post at Github entitled [How to undo almost anything in Git](https://github.com/blog/2019-how-to-undo-almost-anything-with-git) -- this is required reading.

Otherwise, here are a list of typical "dos" and "don'ts" for Git, some with a specific eye towards Open MPI:

Do
--
* make smaller, logically self-contained commits
* [write nice commit messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html), with a concise subject line and a useful description of *what* change is being made and *why*
* (In general) Rebase your local work before pushing up to `ompi/master` instead of creating merge commits that say "`Merge branch 'master' of 'ssh://...'`".  Unfortunately, the default behavior of `git pull` is to merge instead of rebase, [see here for how to change that default](http://viget.com/extend/only-you-can-prevent-git-merge-commits).
* If you think you screwed up a repository (yours or a shared one), **STOP**.
  * See this [Awesome flow chart on how to get out of Git messes](http://justinhileman.info/article/git-pretty/)
  * See the "[How to undo almost anything in Git](https://github.com/blog/2019-how-to-undo-almost-anything-with-git)" blog post at Github
  * Ask for help on devel@open-mpi.org.  Many Git operations are reversible if you don't take too many steps in the wrong direction.
* run `git gc` on your repository periodically, it will speed up many operations and shrink the space consumed on disk by your `.git` directory

Don't
-----
* change published history
* **change published history!**
* ***CHANGE PUBLISHED HISTORY!!***
* push non-master branches to the main `ompi` repo
* **push non-master branches to the main `ompi` repo!**
* push with `--force` or `--mirror` to the main `ompi` repo
* delete or rename published tags -- this is a real pain to unwind
* make large rambling commits that intermix multiple unrelated changes
* make unnecessary whitespace changes in the same commit where you are changing logic
* commit any large binaries to the repository, this will rapidly increase the storage space required for everybody's clones
* use `git filter-branch` unless you *really* know what you are doing, and especially don't use it on published history
* use `git pull` instead of `git pull --rebase` for normal local work that will be pushed up to `ompi/master` (see "Do" above)

# ABSOLUTELY Critical to read (and understand [this](http://justinhileman.info/article/git-pretty/)) #

# Another git [ruby](https://github.com/blog/2019-how-to-undo-almost-anything-with-git) #

For any other issue related to creating and managing Pull Requests (PR) I strongly encourage you to visit the Open MPI git [webpage](https://github.com/open-mpi/ompi/wiki)

