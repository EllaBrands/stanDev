### Overview

The Stan developer process is based on the gitflow model described by Vincent Driessen in the blog post
"[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/ )."  The rest of this document details the steps required from developers.  If you don't read anything else, please remember: 

* Never push directly into the master or development branches, nor the hotfix-* or release-* branches.

* For working on new features, branch from `develop` into a branch called `feature/<some-descriptive-name>`, and when the work is done (thoroughly tested and documented), create a pull request back into `develop`.



### Permanent Branches

The Stan repository has two permanent branches:

#### `master`

The master branch is always at our most recent, production-ready release.  Only production-ready releases should be pushed to this branch.  Each point on this branch will be tagged with the most recent version number.

#### `develop`  

The development branch is the current working branch for development integration.  We require this branch to pass all unit tests so that all development branches may branch from it.  To enforce the functional state of the development branch, all pushes to it will be mediated by pull request. The unit tests required to pass in order to merge into the development branch are `test-unit`, `test-distribution` and `test-models`.

#### Honor System

We are working on an honor system. The active developers with push permission on the Stan repository will still technically have the ability to push to the master, development, release, and hotfix branches. 

#### Preventing Mistakes

Developers should take the following steps to configure git to prevent unintentional pushes to the master or development branches.  

First, install git version 1.8 or later.

Second, configure git's push and merge behavior to match the gitflow process requirements.  To configure push and merge behavior just for Stan, change directories to `stan` and execute the following two commands.

    > git config push.default simple
    > git config merge.ff false

To make these commands global to all git projects (not just Stan), use the following.

    > git config --global push.default simple
    > git config --global merge.ff false

The simple push default implies that only the _current_ branch is git pushed to the specified remote
repository (typically `origin`), as opposed to the default behavior of git pushing _all_ local branches to the specified remote repository. 

The fast-forward merge configuration tweak prevents any fast-forward merges, since
they are inconsistent with the gitflow development model.

Third, download the pre-push script from the following link

* [git pre-push script](https://stan-dev.googlegroups.com/attach/255775ffea1a3d08/pre-push.txt?gda=l1Q5YkYAAAAZxtxdgPezaYoZ-2CibDFNxvRQADtfWXVf7Wp9jTazNhhIKImChiwUZBkdErcfv6Vx40jamwa1UURqDcgHarKEE-Ea7GxYMt0t6nY0uV5FIQ&part=4) 

to your local directory

    > stan/.git/hooks/pre-push

Note that the file must _not_ have any extension and must be _executable_.  For anything but Windows, this can be accomplished with the command

    > chmod +x stan/.git/hooks/pre-push 

The pre-push script will be run whenever you git push but _before_ anything is actually pushed.  This gives the script the opportunity to check whether the remote repository is the origin (i.e., the `stan-dev/stan` repository on GitHub) and whether the branch is among those that you should not push to (i.e., master, development, release, or hotfix). You can still push to a feature branch on the origin or push anything to a remote repository that is not the origin.


### Non-Permanent Branches

This section describes the three types of non-permanent branches. 

#### `feature/`

All development work on features that have not been released go in feature branches.

Feature branches branch from `develop` and should be named `feature/<some-descriptive-name>`.  

After development is complete on a feature, submit a pull request back to `develop`.

#### `hotfix/`

Patches to the current release go in hotfix branches. 

Hotfix branches branch from `master` and are named `hotfix/<some-issue-number>`.  There should be an issue created for the bug in Stan's GitHub issue tracker prior to branching to provide an issue number.   

Once the patch is complete, submit a pull request back to `master` and it will be merged into both `master` and `develop` branches.

#### `release/`

New releases go in release branches.

Release branches branch from `develop` and are named `release/<some-release-number>`.  

Once a release branch is complete, it will be merged into both `master` and `develop`.


### How to Contribute with a Clone of the Repository

Using this method, we can share feature branches with ease for collaboration.  It involves the following steps.

First, clone the GitHub stan-dev/stan repository. From the command line:

    > git clone https://github.com/stan-dev/stan.git
    > cd stan

Second, verify you are in the correct default repository, `develop`, with the command.

    > git branch

Third, create a branch.  To create a feature branch, use the following

    > git checkout develop
    > git checkout -b feature/foo
    > git push

For a hotfix branch (less common), suppose we are currently at version v3.2.1.

    > git checkout master
    > git checkout -b hotfix/v3.2.2
    > git push

The last push command is to make the branch public. This will be necessary in order to create a pull request to have the work merged into the development (or master) branch.

Fourth, code and document using the usual git commands.  Collaboration is easy since all the developers have push access to the feature (or hotfix) branch. 

Fifth, when finished, create a pull request. A pull request indicates that the work is done and should be merged back to the appropriate branch.

When a pull request is made, it is expected that the current code passes:

    > make manual
    > make doxygen
    > make test-unit
    > make test-distributions
    > make test-models

If these tests do not pass on your local machine, the pull request won't be accepted. Passing these tests do not guarantee that the pull request will be merged.  It will first go through code review and then be tested on the integration server. 

### Creating a Pull Request for a Feature Branch

First, merge the current state of the branch `develop` back into the current feature branch.  Update branch `develop` and merge it into the current branch .  

Second, Deal with any conflicts arising from the merge. 

Third, push the result of the merge so that the changes are available to GitHub.

    > git pull origin develop
    > git merge --no-ff develop
    > git push

Fourth, create the pull request through GitHub.  

* Go to https://github.com/stan-dev/stan  (refresh the page if necessary)

* Select the feature branch you are working on in the drop-down list. (Below the "Clone in ..." row) A 

* Click the "Pull Request" button that pops up underneath feature branch drop-down list. 

* Fill in form and press "Send pull request".

Alternative directions to create a pull request:  

* Go to https://github.com/stan-dev/stan (refresh page if necessary). 

* Click `Pull Request` button. It's towards the top right, on the same row as the repository name, `stan-dev/stan`.  

* Select the correct base repo, `stan-dev/stan`, and base branch, `develop`. 

* Select the correct head repository, `stan-dev/stan`, and head branch, `feature/foo`. 

* Fill in form and press "Send pull request".

#### Creating a Pull Request for a Hotfix Branch

    For a hotfix branch:
        Create a pull request.
        Go to https://github.com/stan-dev/stan (refresh page if necessary)
        Select the hotfix branch you are working on in the drop-down list. (Below the "Clone in ..." row)
        A "Pull Request" button should pop up underneath the hotfix branch drop-down list. Click it.
        Fill in form and press "Send pull request".

        Alternate directions:
        Go to https://github.com/stan-dev/stan (refresh page if necessary)
        Click "Pull Request" button. It's towards the top right, on the same row as the repo name 
        "stan-dev/stan"
        Select the correct base repo, "stan-dev/stan", and base branch, "develop"
        Select the correct head repo, "stan-dev/stan", and head branch, "feature/foo"
        Fill in form and press "Send pull request".
    Relax.
    At this point, one or more of the developers will check the pull request and do an informal code 
    review. At the same time, our Jenkins box will verify that all tests pass on Windows. If both are 
    good, then the pull request will be accepted, the code will get into the correct branch, and the 
    branch will be deleted.
    If the pull request is not accepted immediately, fix the issues.
        Check out the branch
        > git checkout feature/foo
        > git pull origin feature/foo
        (Or you can set upstream tracking so you can do "git pull feature/foo")
        Do work.
        Others can help at this point since the branch is available to everyone else.
        The work will update the pull request.
        Go back to "5. Relax"

That's it. That's how it works.


How to Contribute with a Fork of the Repository (we should be using a clone)

(This is how some of us have been working. The downside: if you want others to help, you need to give people permissions to your personal fork of the stan repository.)

    Fork the GitHub stan-dev/stan repository. Done through either the web or the GitHub application.
    Create a branch. For a feature branch (most common):
    > git checkout develop
    > git checkout -b feature/foo
    > git push

    For a hotfix branch (less common):
    If we're currently at v3.2.1
    > git checkout master
    > git checkout -b hotfix/v3.2.2
    > git push

    The last command, git push, is to make the branch public. This is necessary for when a pull request is 
    created.
    Do work.
    Use the regular git commands to work. If you want collaborators, let them have access to the branch.
    Create a pull request.
    A pull request indicates that the work is done and should be merged back to the appropriate place. 
    When the pull request is made, it is expected that the current code passes:
    > make manual
    > make doxygen
    > make test-unit
    > make test-distributions
    > make test-models

    If it does not pass on your local machine, it won't be accepted. Passing these tests do not guarantee 
    acceptance.

    For a feature branch:
        Merge the current state of "develop" back into the current feature branch:
        Update "develop", merge "develop" into current branch (deal with any conflicts using git), push so 
        the changes are available to github.
        > git pull origin develop
        > git merge --no-ff develop
        > git push
        Create a pull request.
        Go to https://github.com/stan-dev/stan (refresh page if necessary)
        Select the feature branch you are working on in the drop-down list. (Below the "Clone in ..." row)
        A "Pull Request" button should pop up underneath the feature branch drop-down list. Click it.
        Fill in form and press "Send pull request".

        Alternate directions:
        Go to https://github.com/stan-dev/stan (refresh page if necessary)
        Click "Pull Request" button. It's towards the top right, on the same row as the repo name 
        "stan-dev/stan"
        Select the correct base repo, "stan-dev/stan", and base branch, "develop"
        Select the correct head repo, "personal/stan", and head branch, "feature/foo"
        Fill in form and press "Send pull request".

    For a hotfix branch:
        Create a pull request.
        Go to https://github.com/stan-dev/stan (refresh page if necessary)
        Select the hotfix branch you are working on in the drop-down list. (Below the "Clone in ..." row)
        A "Pull Request" button should pop up underneath the hotfix branch drop-down list. Click it.
        Fill in form and press "Send pull request".

        Alternate directions:
        Go to https://github.com/stan-dev/stan (refresh page if necessary)
        Click "Pull Request" button. It's towards the top right, on the same row as the repo name 
        "stan-dev/stan"
        Select the correct base repo, "stan-dev/stan", and base branch, "master"
        Select the correct head repo, "personal/stan", and head branch, "feature/foo"
        Fill in form and press "Send pull request".

    Relax.
    At this point, one or more of the developers will check the pull request and do an informal code 
    review. At the same time, our Jenkins box will verify that all tests pass on Windows. If both are 
    good, then the pull request will be accepted, the code will get into the correct branch, and the 
    branch will be deleted.
    If the pull request is not accepted immediately, fix the issues.
        Check out the branch
        > git checkout feature/foo
        > git pull origin feature/foo
        (Or you can set upstream tracking so you can do "git pull feature/foo")
        Do work.
        Others can help if you've given them access. If this workflow is a problem, follow directions for 
        cloning.
        The work will update the pull request.
        Go back to "5. Relax"



How to Update Your Current Stan Directory

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