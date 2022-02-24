# Parsec Migration from Bitbucket to Github

# Fork PaRSEC on Github

Use the 'Fork' option to create a local fork for the repo on your own Github.

## Changing your local copy

Get the list of current remotes you have configured on your current local clone

```
$ git remote -v
$ git remote -v
george  git@bitbucket.org:bosilca/parsec.git (fetch)
george  git@bitbucket.org:bosilca/parsec.git (push)
joseph  git@bitbucket.org:schuchart/parsec.git (fetch)
joseph  git@bitbucket.org:schuchart/parsec.git (push)
mine    git@bitbucket.org:abouteiller/parsec.git (fetch)
mine    git@bitbucket.org:abouteiller/parsec.git (push)
nuria   git@bitbucket.org:nuriallv/parsec.git (fetch)
nuria   git@bitbucket.org:nuriallv/parsec.git (push)
origin  git@bitbucket.org:icldistcomp/parsec.git (fetch)
origin  git@bitbucket.org:icldistcomp/parsec.git (push)
thomas  git@bitbucket.org:herault/parsec.git (fetch)
thomas  git@bitbucket.org:herault/parsec.git (push)
```

Foreach of these remotes, rename them (you can also elect to erase personal remotes, as GitHub as a much better way of direct pulling PRs from the origin rather than having to pull from individual repos).

```
$ git remote rename origin bitbucket
$ git remote rename mine bbt-mine
[...]
$ git remote -v
[...]
bbt-mine   git@bitbucket.org:abouteiller/parsec.git (fetch)
bbt-mine   git@bitbucket.org:abouteiller/parsec.git (push)
bitbucket  git@bitbucket.org:icldistcomp/parsec.git (fetch)
bitbucket  git@bitbucket.org:icldistcomp/parsec.git (push)
```

Add the new remotes 

```
git remote add origin git@github.com:icldisco/parsec.git
git remote add mine   git@github.com:abouteiller/parsec.git
```

## Copy your branches

You will need to copy your branches you still want to work on to your new Github Fork, so that you can use them as sources for PRs later

```
git checkout bbt-mine/topic/xyz
git checkout push -vu mine
```

## Create a PR that replicates your existing PR on Bitbucket, but this time on Github.

Go to the Github interface. Your newly pushed branch will probably show as "new" and propose to make a PR. Do it. If not go into 'branches' in your local repo and create the PR from there instead. 

If your PR  is a copy of a Bitbucket PR, 

1. Add the text `[BBT#123]` to the PR title, e.g., `[BBT#123] Fix some nasty bug`
2. Add a link to the Bitbucket PR in the first line of the description of the PR `https://bitbucket.org/icldistcomp/parsec/pull-requests/582`
3. Copy or make up some description of what the PR does so that it remains self contained. Add only most relevant discussion items from the original, if any needed. 
4. Make sure you tagged the PR with the appropriate tags, e.g., `bug`, `feature`, `high-priority`, etc. 
5. Is this PR part of a milestone target? (not all PR have to be).
6. Assign yourself (or the relevant person)
7. Pick 2 (or more) reviewers that would have good insight on the code area or feature. 


