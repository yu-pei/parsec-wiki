# Memory Error Detectors #

This is absolutely not a replacement for the regular usage of a tool such as [Valgrind](http://valgrind.org/), but it does provide somehow similar benefits, with a significant lesser cost. Welcome in your tools repertoire, the widely acclaimed "Address Sanitizer", a Google project hosted at [Code.Google](https://code.google.com/p/address-sanitizer/)

The integration in PaRSEC is quite straightforward, simply adding *"-fsanitize=address -O1 -fno-omit-frame-pointer"* to your different CFLAGS (C and CXX at this point), and using a compatible compiler ([clang](http://clang.llvm.org/) > 3.0) should be enough. Once, everything compiled with the flags run a quick ctest to validate the code.

Any output matching the following pattern "==[0-9]+==ERROR: AddressSanitizer:" is a bad sign and should be acted upon immediately.

# Git Best Practices #

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

# To review locally a private pull request

Get the patch from the pull request
```
#!shell
curl https://bitbucket.org/api/2.0/repositories/icldistcomp/parsec/pullrequests/{#PR}/patch > pr{#PR}.patch
```

Than apply the patch on your local copy

```
#!shell

git apply pr{#PR}.patch
```

