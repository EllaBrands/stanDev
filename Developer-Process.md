### 1. Overview

The Stan developer process is based on the gitflow model described by Vincent Driessen in the blog post
"[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/ )."  The rest of this document details the steps required from developers.  

If you don't read anything else, please remember the following when working on features: 

* Never push directly into master or development branches.  

* For working on new features, branch from `develop` into a branch called `feature/<some-descriptive-name>`, and when the work is done (thoroughly tested and documented), create a pull request back into `develop`.  

### 2. Branches

The gitflow process distinguishes between the permanent branches managed by the code administrators and the temporary branches where development is actually carried out.

#### 2.1 Permanent Branches

**Master branch:** The `master` branch is always at our most recent, production-ready release.  Only production-ready releases should be pushed to this branch.  Each point on this branch will be tagged with the most recent version number.

**Development branch:** The `develop` branch is the current working branch for development integration.  We require this branch to pass all unit tests (detailed below) so that all development branches may branch from it.  To enforce the functional state of the development branch, all pushes to it will be mediated by pull request.  Before a pull request is merged with the development branch, the unit tests, distribution tests and model tests must pass.


#### 2.2 Non-Permanent Branches

**Feature branches:** All development work on features that have not been released go in feature branches. Feature branches branch from `develop` and should be named `feature/<some-descriptive-name>`.  After development is complete on a feature, submit a pull request back to `develop`.

**Hotfix branches:**  Patches to the current release go in hotfix branches. Hotfix branches branch from `master` and are named `hotfix/<some-issue-number>`.  There should be an issue created for the bug in Stan's GitHub issue tracker prior to branching to provide an issue number.    Once the patch is complete, submit a pull request back to `master` and it will be merged into both `master` and `develop` branches.

**Release branches:**  Work to complete a new release go in release branches. Release branches branch from `develop` and are named `release/<some-release-number>`.  Once a release branch is complete, it will be merged into both `master` and `develop`.

### 3.  Preventing Inadvertent Pushes

We are working on an honor system. The active developers with push permission on the Stan repository will still technically have the ability to push to the master and development branches, but are asked to take the following steps to prevent inadvertent pushes to the master and development branches.

**First**, install git version 1.8 or later.

**Second**, configure git's push and merge behavior to match the gitflow process requirements.  To configure push and merge behavior just for Stan, change directories to `stan` and execute the following two commands.

    > git config push.default simple
    > git config merge.ff false

To make these configurations apply globally to all git projects, use:

    > git config --global push.default simple
    > git config --global merge.ff false

The simple push default implies that only the _current_ branch is git pushed to the specified remote
repository (typically `origin`), as opposed to the default behavior of git pushing _all_ local branches to the specified remote repository. 

The fast-forward merge configuration prevents fast-forward merges, which are inconsistent with the gitflow development model.

**Third**, download the [git pre-push script](https://stan-dev.googlegroups.com/attach/255775ffea1a3d08/pre-push.txt?gda=l1Q5YkYAAAAZxtxdgPezaYoZ-2CibDFNxvRQADtfWXVf7Wp9jTazNhhIKImChiwUZBkdErcfv6Vx40jamwa1UURqDcgHarKEE-Ea7GxYMt0t6nY0uV5FIQ&part=4) 
to your local version of the `stan` repository, placing it in the file `stan/.git/hooks/pre-push`

Note that the file is named `pre-push` and must _not_ have any extension and must be _executable_.  For anything but Windows (or Windows through Cygwin), the file can be made executable with the command

    > chmod +x stan/.git/hooks/pre-push 

On Windows without Cygwin, right click on the icon for the file and click on "Properties" or "Security and Sharing".  You may need to login as administrator depending on where the pre-push script file is located.  Then set the permissions to allow the user doing the work to read and execute.

The pre-push script will be run whenever you git push but _before_ anything is actually pushed.  This gives the script the opportunity to check whether the remote repository is the origin (i.e., the `stan-dev/stan` repository on GitHub) and whether the branch is among those that you should not push to (i.e., `master` and `develop`). You can still push to a feature branch on the origin or push anything to a remote repository that is not the origin.




### 4.  How to Contribute with a Clone of the Repository

Using this method, we can share feature branches with ease for collaboration.  It involves the following steps.

First, clone the GitHub stan-dev/stan repository.

    > git clone https://github.com/stan-dev/stan.git
    > cd stan

Second, verify you are in the correct default repository, `develop`.

    > git branch

Third, create a branch.  For a new feature, run the following with your feature name in place of `foo`. 

    > git checkout develop
    > git checkout -b feature/foo
    > git push -u origin feature/foo

The last push command is to make the branch public;  the `-u` option sets the upstream branch to the origin, so that git pull requests pull from the origin's version of the branch `feature/foo`.

This will be necessary in order to create a pull request to have the work merged back into the branch from which it was checked out.  (The first such push may require verifying the github credentials.)


For a hotfix branch (less common), run the following, where `v3.2.2` is replaced with the apppropriate version number (hotfixes increment the last number, which is available as the latest tag on the master branch).

    > git checkout master
    > git checkout -b hotfix/v3.2.2
    > git push -u origin feature/foo

Fourth, code and document using the usual git commands.  Collaboration is easy since all the developers have push access to the feature or hotfix branch you just created.

Fifth, when finished, create a pull request. A pull request indicates that the work is done and should be merged back to the appropriate branch.



### 5. Pull Requests

#### 5.1 Preconditions for Pull Requests

##### Final State Passes All Tests

When a pull request is made, it is expected that the current code passes:

    > make manual
    > make doxygen
    > make test-unit
    > make test-headers
    > make test-distributions
    > make test-models

And as soon as the `test-headers` target is in, it should also pass:

    > make test-headers

Passing these tests does not guarantee that the pull request will be accepted and merged.  It will first go through code review and then be tested on the [integration server](wiki/Continuous-Integration). 

##### Commit History is Clean

We strongly urge you to keep your commit history clean in the sense that each commit passes all tests.  This is also on the honor system---we are not going to enforce this by testing each commit in a pull request. 

If you prefer a personal workflow that involves committing changes that do not pass tests, it is possible to clean up your history.  Git provides a number of tools for "rewriting history," as discussed in Chacon's book, <i>Pro Git</i>, in the chapter "[Rewriting History](http://git-scm.com/book/en/Git-Tools-Rewriting-History)." In particular, a process known as rebasing (followed by a fresh commit) allows you to compose a sequence of previous commits into a single commit that is visible from the outside.


#### 5.2  Steps for Pull Requests

First, if the pull request is for a feature branch, update branch `develop` and merge it into the current branch.  If the request is for a hotfix branch, no merge is necessary.

    > git pull origin develop
    > git merge --no-ff develop

Second, deal with any conflicts arising from the merge. 

Third, make sure the unit tests pass.  (You might want to skip the model tests and let those get handled by the integration server Jenkins after the pull request is made.)

Fourth, push the result of the merge so that the changes are available to GitHub.

    > git push

Fifth, create the pull request through GitHub.  This involves

* opening <https://github.com/stan-dev/stan> and refreshing the page,

* selecting the feature branch you are working on from the drop-down list, which can be found below the "Clone in ..." row, 

* clicking the "Pull Request" button that pops up underneath feature branch drop-down list, 

* filling in the form, and

* pressing the "Send pull request" button.

Sixth, relax. At this point, you are waiting for

* other developers to carry out an informal code review and

* one of the administrators to kick off Stan's continuous integration server (Jenkins) to verify that all tests pass on Windows.   

_Warning:_ &nbsp; The full model functional tests currently take around 10 hours.

#### 5.3 Pull Requests that Pass Review

If the code passes code review and unit tests, 

* the pull request will be accepted,

* the pulled code will be merged by one of the administrators into the correct location(s) as defined for the gitflow process above, and

* the branch will be deleted.  


#### 5.4 Pull Requests that Don't Pass Review

If the code is not accepted by code review or it fails unit tests, you need to fix the issues (by yourself or by enlisting help).  At this point, you want to first make sure your copy of the feature is up to date (one of the code reviewers may have contributed a patch):

    > git checkout feature/foo
    > git pull origin feature/foo

If you set upstream tracking, the above two steps can be simplified to

    > git pull feature/foo

Fix whatever needs to be fixed.  Others can help at this point because the branch can be pushed to through GitHub.  

Pushing to the working branch will update the pull request with the message used for the commits.  

At this point, you are back to waiting for another code review and round of integration tests. 




### How to Contribute with a Fork of the Repository

If you do not have push permission on `stan-dev`, then you have to fork the repository.
If you have push permission on `stan-dev`, you should clone rather than fork.

If you want others to help you, you will need to give them permissions to push to your forked repository.  Or make them send you pull requests.

First, fork the GitHub stan-dev/stan repository. Done through either the web or the GitHub application.

Add the stan-dev/stan repo as a remote (typically called `upstream`) to your own fork (more on  [syncing github forks](https://help.github.com/articles/syncing-a-fork) with their upstream repos): 

    > git remote add upstream https://github.com/stan-dev/stan.git

After that, development follows the same process as for a clone of the repository until a pull request is needed.  The differences are that on a fork for a feature, you need to

* select the correct base repository, `stan-dev/stan`, and base branch, `develop`, and

* select the correct head repository, `personal-repo/stan`, and head branch, `feature/foo` (where "foo" is replaced with the feature name).

In the unlikely case a hotfix branch is created from a fork, you need to

* select the correct base repository, `stan-dev/stan`, and base branch, `master`, and

* Select the correct head repository, `personal-repo/stan`, and head branch, `hotfix/v3.2.2` (where the version is the appropriate increment).

After the pull request is submitted, the steps for a fork are the same as for a clone.