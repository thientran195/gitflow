# Git workflow  

## Introduction


### Branches

#### master

Master branch is use to merge new features to be deliver in production environment
but must not be checkout directly, instead create tags version.

#### develop

Develop branch is use for the staging / preproduction environments.

#### feature

> feature/trello-[issue-number]

Features branches are use to create new feature or bug fixes in development environment.

*They are create from the develop branch and merged back to develop*

#### release

> release/v[tag-version]

Releases branches are use to deliver new features in production environment, they can be create
to update only the CHANGELOG.md or to switch off from the develop branch to avoid to include
in the release scope unwanted features.

*They are create from the develop branch and merged back to master and develop*

##### hotfix

> hotfix/trello-[issue-number]

Hotfixes branches are use to bug fixes in production environment.

*They are create from the master branch and merged back to master and develop*

### Tags

Tags are use to deliver new features develop branch (but technically the develop branch is not merge
directly to the master branch, but from the release branch).

---  
  
* [Develop a feature](#develop-a-feature)  
* [Deliver the production](#deliver-the-production)  
* [Bug fix in production](#bug-fix-in-production)


### Develop a feature

#### 1. Update your local

Checkout the **develop** branch and pull the latest commits.

```
git checkout develop
git pull
```

>:information_source: **Tips**
>---
>If you have some untracked (unversionned files) / unstaged (unadded files) i recommand you to **stash** them before to execute  above commands.
>
>```
>git stash --include-untracked
>```
>
>and after the commands execution you can retrieve those files to drop them off from the stash :
>
>```
>git stash drop stash@{0}
>```
>
>*stash@{0}* mean the most recent index from the stash, you can explore the stash with `git stash list` or show stash index content with `git stash show -p stash@{0}`.*

#### 2. Create your feature branch

Create your new feature branch from **develop** branch.

```
git checkout -b feature/trello-[issue-number]
```

>:warning: **Warning** !
>---
>
>Before to execute above command, do the following command to check if nobody has already create / merge a branch with this name :
>
>```
>git log | grep 'feature/trello-[issue-number]
>```
>

#### 3. Ready to commit ?

Add your files

```
git add <files-paths>
``` 

Commit your files

```
git commit -m "<your message>"
```

> Your message must be short and meaning what you've done in commit

#### 4. Do my feature is up-to-date with develop ?

Before to create the merge request, you need to fetch all the commits you've miss from develop since you create your feature branch.

Let's do the [1. Update your local](#1-update-your-local) again and go back to our feature branch. 

Now it's time to rebase your feature branch with the **develop** branch.

```
git rebase develop
```  

or you can use
```
git fetch develop && git rebase develop
```

If you have files in conflicts, solve them and do a `git status` to know if you need to `git rebase --continue` (to continue to rewind your branch) or it's finish.

Push your branch to the remote repository

```
git push
```
or
```
git push --set-upstream origin feature/trello-<issue-number>
```
#### 5. It's time to create the pull request !

Go to your [Merge requests dashboard](https://github.com/<your account>/<your project>/pulls) and create a new merge request with as source your feature branch and as target branch the develop branch, assign the merge request to the project maintainer.  

### Deliver the production [For maintainers]

Production environment must use the *master* branch, to deliver your developments from the **develop**
branch, create a release branch.

> The release branch can be create earlier to avoid to have unwanted features and define the features scope.

#### 1. Go the the develop branch

```
git checkout develop
```

#### 2. Create your release branch

```
git checkout -b release/v1.0.0 origin/release/v1.0.0
```

#### 3. Update the CHANGELOG.md

Find the commit hash from the latest release and get log the since this commit (or with the latest tag).

```
git log --pretty="%h - %s (%an)" <commit-hash|tag>..HEAD^
```

Copy/paste the output in the top of the CHANGELOG.md, remove the merge commits lines and add bullet point
before each commit lines.

Commit the CHANGELOG.md updated.

See the [CHANGELOG template](../templates/changelog.md).

#### 4. Ready to deliver !

Merge to the develop branch

```
git checkout develop
git merge --no-ff release/trello-[issue-number]
```

Merge to the master branch

```
git checkout master
git merge --no-ff release/trello-[issue-number]
```

Create a new tag to keep Git log history clear

```
git tag -a v[tag-version]
git push --tags
```

Example : `git tag -a v1.0.0`

Go to your production environment, fetch and deploy your new tag

```
git fetch --tags
git checkout tags/v[tag-version]
```

### Bug fix in production

When there is a problem in production environment, we can't follow the [Develop a feature](#develop-a-feature) workflow because
if the develop branch is used to create the branch to fix the bug, it will include all the commits since the latest production deployment.

#### 1. Update your local

Checkout the **master** branch and pull the latest commits.

```
git checkout master
git pull
```

#### 2. Create your hotfix branch
 
```
git checkout -b hotfix/trello-[issue-number]
```
 
#### 3. Fix the issue

Fix the issue and commit it (see [Ready to commit ?](#3-ready-to-commit-))

#### 4 It's time to ... !

The hotfix need to be apply on **develop** and **master** branches.

##### 4.1 Create the merge request ! [For developers]

Create a merge request per target branches.

- One for the develop branch
- One for the master branch

>**Warning** !
>---
>
> Do not check the opt-in to remove the branch after the merge request will be accept !
>
> Only remove the source branch manually after both merge requests will have bee accepted.

##### 4.2 Merge ! [For maintainers]

Merge to the develop branch

```
git checkout develop
git merge --no-ff hotfix/trello-[issue-number]
```

Merge to the master branch

```
git checkout master
git merge --no-ff hotfix/trello-[issue-number]
```

Create a new tag to keep Git log history clear

```
git tag -a v[tag-version]
git push --tags
```

Example : `git tag -a v1.0.0`

And finally remove the hotfix branch

```
git branch -D hotfix/trello-[issue-number]
```


