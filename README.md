# Git setup for opensource
Steps to setup git for contributing to opensource using best practices

> Note: This guide assumes you have already installed git and configured your credentials using [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager/blob/main/README.md)

## Configure your fork

This guide refers to the original repository as the upstream repository and to the forked repository as the origin repository. I assume you already have a fork of the upstream and you `git clone` it already.

## Script to synchronize master branch on your fork with the upstream master branch

```
#!/usr/bin/env bash
set -o errexit
set -o pipefail
set -o nounset

ORG_NAME=$1
REPO_NAME=$2
BRANCH=$3

function setRemoteUpstream() {
    ORG_NAME=$1
    REPO_NAME=$2
    git remote add upstream https://github.com/$ORG_NAME/$REPO_NAME.git
}

if setRemoteUpstream $ORG_NAME $REPO_NAME
then
    
    printf --  "\033[37m  New upstream pointing to https://github.com/${ORG_NAME}/${REPO_NAME}.git is set \033[0m\n"
else
    printf --  "\033[37m  Making sure proper upstream is set to https://github.com/${ORG_NAME}/${REPO_NAME}.git \033[0m\n"
    git remote remove upstream
    setRemoteUpstream $ORG_NAME $REPO_NAME
fi

git fetch upstream $BRANCH
git branch -u upstream/$BRANCH $BRANCH

printf -- "\033[32m Now $BRANCH branch on you local fork is in sync with upstream $BRANCH branch \033[0m\n"
git branch -vv | grep $BRANCH
```

Save this script as `conf_fork.sh` inside the root of your local repository, this script will set the remote upstream as the orgs repo for your local fork

## Usage

Open a terminal at the root of the repository.

The script accepts three parameters like so `bash conf_fork.sh <ORG_NAME> <REPO_NAME> <MASTER_OR_MAIN>`

### Example

If you want to contribute to [cli](https://github.com/asyncapi/cli) to [AsyncAPI](https://github.com/asyncapi) and you have forked the repo, then run the following:

```
bash conf_fork.sh asyncapi cli master
```

That's it, you now have your local master branch in sync with the upstream orgs master branch.

## Start contributing

### Create a local branch on your fork

```
git switch -c new-branch
```

Verify that the branch was created

```
git branch -a
```

Once you create a new branch you must run the `--set-upstream` switch the first time you perform a push.

```
git push --set-upstream origin new-branch
```

Or you can use the shorthand `-u` switch.

```
git push -u origin new-branch
```

All subsequent `git push` commands automatically move local branch changes up to the remote branch.

Once the `git push` of the new branch has completed check your remote to verify that the new-branch was successfully pushed.

Subsequent commits can be pushed directly by `git push origin`

Finally create a pull request from the branch of your forked repository to the `master` branch of the upstream repository and wait for the maintainers' review.
