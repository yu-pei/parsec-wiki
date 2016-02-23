# To create your own pull request

First you need to fork the repository in your own account. You can do that simply by clicking the fork button on the bitbucket interface.

https://bitbucket.org/bosilca/dplasma/fork

Then, clone the repository on your laptop:
```
#!shell
git clone git@bitbucket.org:username/yourforkname.git
```

Once this is done, you can setup the parsec repository as the upstream of your clone to simplify the update of your fork repository.
```
#!shell
git remote add upstream git@bitbucket.org:icldistcomp/parsec.git
```

Now, you have your repository configures, and you want to create a new pull request. The first step is to create a branch from the HEAD of the repository
```
#!shell
git branch your_branch_name
git checkout your_branch_name
```

Apply your modifications in your branch. Then, you need to push this branch on your online repository
```
#!shell
git push -f origin your_branch_name
```

or without -f, if the branch already exists online, and you just want to update it.

Once your branch is online, on the bitbucket interface, go to the branches webpage, select the branch you want to push as a pull request, and push the button !!!

***Be careful to check the 'close after merge' check box, and to push to the icldistcomp/parsec repository*** By default the checkbox is not checked, and the default repository is your fork.


# To review locally a private pull request

Get the patch from the pull request
```
#!shell
curl https://bitbucket.org/api/2.0/repositories/icldistcomp/parsec/pullrequests/#PR/patch > pr#PR.patch
```

Than apply the patch on your local copy

```
#!shell

git apply pr#PR.patch
```

