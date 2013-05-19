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
to your local version of the `stan` repository, `stan/.git/hooks/pre-push`

Note that the file must _not_ have any extension and must be _executable_.  For anything but Windows (or Windows through Cygwin), the file can be made executable with the command

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
    > git push

For a hotfix branch (less common), run the following, where `v3.2.2` is replaced with the apppropriate version number (hotfixes increment the last number, which is available as the latest tag on the master branch).

    > git checkout master
    > git checkout -b hotfix/v3.2.2
    > git push


The last push command is to make the branch public. This will be necessary in order to create a pull request to have the work merged back into the branch from which it was checked out.

Fourth, code and document using the usual git commands.  Collaboration is easy since all the developers have push access to the feature or hotfix branch you just created.

Fifth, when finished, create a pull request. A pull request indicates that the work is done and should be merged back to the appropriate branch.



### 5. Pull Requests

#### 5.1 Preconditions for Pull Requests

When a pull request is made, it is expected that the current code passes:

    > make manual
    > make doxygen
    > make test-unit
    > make test-distributions
    > make test-models

Passing these tests does not guarantee that the pull request will be accepted and merged.  It will first go through code review and then be tested on the integration server. 

#### 5.2  Steps for Pull Requests

First, if the pull request is for a feature branch, update branch `develop` and merge it into the current branch.  If the request is for a hotfix branch, no merge is necessary.

Second, deal with any conflicts arising from the merge. 

Third, make sure the unit tests pass.  (You might want to skip the model tests and let those get handled by the integration server Jenkins after the pull request is made.)

Fourth, push the result of the merge so that the changes are available to GitHub.

    > git pull origin develop
    > git merge --no-ff develop
    > git push

Fifth, create the pull request through GitHub.  Instructions for doing this are in the next section.

Sixth, relax. At this point, you are waiting for

* other developers to carry out an informal code review, and

* one of the administrators will kick off Stan's continuous integration server (Jenkins) to verify that all tests pass on Windows.   The full model functional tests currently take around 10 hours.

At this point, if the code passes code review and unit tests, then

* the pull request will be accepted, 

* the pulled code will be merged by one of the administrators into the correct location(s) as defined for the gitflow process above, and

* the branch will be deleted.  

If the code is not accepted by code review or it fails unit tests, you need to fix the issues (by yourself or by enlisting help).  At this point, you can 

    > git checkout feature/foo
    > git pull origin feature/foo

Or you can set upstream tracking so you can do

    > git pull feature/foo

Fix whatever needs to be fixed.  Others can help at this point because the branch can be pushed to through GitHub.  Pushing to the working branch will update the pull request with the message used for the commits.  

Go back to step 5, "relax."


### Creating Pull Requests

#### Pre-Conditions for Pull Requests

When the pull request is made, it is expected that the current code passes the following tests:

    > make manual
    > make doxygen
    > make test-unit
    > make test-distributions
    > make test-models

If it does not pass on your local machine, it won't be accepted. Passing these tests does not guarantee 
the pull request will be accepted.

#### Creating Pull Requests for Features

* Go to https://github.com/stan-dev/stan  (refresh the page if necessary)

* Select the feature branch you are working on in the drop-down list. (Below the "Clone in ..." row) A 

* Click the "Pull Request" button that pops up underneath feature branch drop-down list. 

* Fill in form and press "Send pull request".

#### (Alternative) Creating Pull Requests for Features

* Go to https://github.com/stan-dev/stan (refresh page if necessary). 

* Click `Pull Request` button. It's towards the top right, on the same row as the repository name, `stan-dev/stan`.  

* Select the correct base repo, `stan-dev/stan`, and base branch, `develop`. 

* Select the correct head repository, `stan-dev/stan`, and head branch, `feature/foo`. 

* Fill in form and press "Send pull request".

#### Creating a Pull Request for a Hotfix

* Go to <https://github.com/stan-dev/stan>.  Refresh the page if necessary

* Select the hotfix branch you are working on in the drop-down list (the list is below the "Clone in ..." row).  

* Click on the "Pull Request" button that pops up underneat the hotfix branch drop-down list.

* Fourth, fill in form.

* Fifth, press "Send pull request".

#### (Alternative) Creating a Pull Request for a Hotfix

* Go to <https://github.com/stan-dev/stan> and refresh page if necessary.

* Click the "Pull Request" button. It's towards the top right, on the same row as the repository name, `stan-dev/stan`

* Select the correct base repository, `stan-dev/stan`, and base branch, `develop`

* Select the correct head repository, `stan-dev/stan`, and head branch, `feature/foo`

* Fill in form and press "Send pull request".




### How to Contribute with a Fork of the Repository

If you do not have push permission on `stan-dev`, then you have to fork the repository.
If you have push permission on `stan-dev`, you should clone rather than fork.

If you want others to help you, you will need to give them permissions to push to your forked repository.  Or make them send you pull requests.

First, fork the GitHub stan-dev/stan repository. Done through either the web or the GitHub application.

Second, create a branch. 

For a feature branch (most common):

    > git checkout develop
    > git checkout -b feature/foo
    > git push

For a hotfix branch (less common), suppose we're current at version v3.2.1.

    > git checkout master
    > git checkout -b hotfix/v3.2.2
    > git push

The last push command makes the branch public. This is necessary in order to create a pull request later.

Work on the feature using regular git commands. 

Create a pull request (see instructions above).




For a feature branch:

Merge the current state of "develop" back into the current feature branch:

Update "develop", merge "develop" into current branch (deal with any conflicts using git), push so the changes are available to GitHub

    > git pull origin develop
    > git merge --no-ff develop
    > git push

Create a pull request.

Go to <https://github.com/stan-dev/stan> and refresh page if necessary.

Select the feature branch you are working on in the drop-down list (below the "Clone in ..." row).

A "Pull Request" button should pop up underneath the feature branch drop-down list. Click it.

Fill in form and press "Send pull request".

Alternate directions:

Go to <https://github.com/stan-dev/stan>, refreshing the page if necessary.

Click "Pull Request" button. It's towards the top right, on the same row as the repository name "stan-dev/stan"
        
Select the correct base repository, `stan-dev/stan`, and base branch, `develop`

Select the correct head repository, `personal-repo/stan`, and head branch, `feature/foo`.

Fill in form and press "Send pull request".

For a hotfix branch:

Create a pull request.

Go to <https://github.com/stan-dev/stan>, refreshing page if necessary.

Select the hotfix branch you are working on in the drop-down list (below the "Clone in ..." row)

A "Pull Request" button should pop up underneath the hotfix branch drop-down list. Click it.

Fill in form and press "Send pull request".

Alternate directions:

Go to <https://github.com/stan-dev/stan>, refreshing the page if necessary.
        
Click "Pull Request" button. It's towards the top right, on the same row as the repository name `stan-dev/stan`

Select the correct base repository, `stan-dev/stan`, and base branch, `master`.

Select the correct head repository, `personal-repo/stan`, and head branch, `feature/foo`.

Fill in form and press "Send pull request".

Relax.

At this point, one or more of the developers will check the pull request and do an informal code review. 

At the same time, our Jenkins box will verify that all tests pass on Windows. 

If both are good, then the pull request will be accepted, the code will get into the correct branch, and the branch will be deleted [ed. how do we do that on `personal-repo`?].
    
If the pull request is not accepted immediately, fix the issues.

Check out the branch

    > git checkout feature/foo

    > git pull origin feature/foo

Or you can set upstream tracking so you can do 

    > git pull feature/foo

Do work.

Others can help if you've given them access.  If this workflow is a problem, follow directions for cloning.

The work will update the pull request.

Go back to step 5, relax.



### How to Update Your Current Stan Directory

If you have no changes, just follow the above directions on cloning. If that doesn't work, do:
> git pull

That will pull down all the current branches. If you were on working on branch "foo", the new branch will be at "feature/foo".

Commit all changes to foo:
> git checkout foo
> git commit -m "mesage" -a

Check out the new feature branch named "feature/foo"
> git checkout feature/foo

Merge the changes in your current branch:
> git merge foo

Delete old branch:
> git branch -d foo
(shouldn't get warnings -- you've already merged into feature/foo)

Start working like normal. You're at step "3. Do Work" from the sections above.


(Informal) Code Review

We need to define what we're looking for, but to start:
- unit tests
- style
- correctness (by eye)

Here, it helps to be on the shared repo because this will allow other developers to help create tests if you're bogged down to finish out the process.

That should be most of the things we need for developers. If anything is unclear, please let me know. The Stan repo is back to being operational.






### Contributing Code

**Rule 1.** Branch `master` is in a releasable state.

**Rule 2.** All contributions come through pull requests. 

1. Anyone can [create a pull request](https://help.github.com/articles/creating-a-pull-request). We need you to explicitly assign copyright to the Stan development team.

2. A pull request should be unit-tested, `make test-unit`, at the very least. If your code touches anything that goes into distributions, a pull request should test the distributions, `make test-distributions`. If your code touches the modeling language, you should run the model tests, `make test-models`.

3. Anyone can do an informal code review for a pull request and leave comments on the pull request's page on GitHub.

4. One of the admins can start the tests on [Jenkins](http://d1m1s1b1.stat.columbia.edu:8080) (this may take hours).

5. One of the admins can declare code review done and push the patch to `develop`.