### 1. Overview

The Stan developer process is based on the gitflow model described by Vincent Driessen in the blog post
"[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/ )."  The rest of this document details the steps required from developers.  

If you don't read anything else, please remember the following when working on features: 

* Never push directly into `develop` or `master` branches.  

* For working on new features, create a new issue, branch from `develop` into a branch called `feature/issue-<number>-<some-descriptive-name>`, and when the work is done (thoroughly tested and documented), create a pull request back into `develop`.  

### 2. Branches

The gitflow process distinguishes between the permanent branches managed by the code administrators and the temporary branches where development is actually carried out.

#### 2.1 Permanent Branches

**Development branch:** The `develop` branch is the current working branch for development integration.  We require this branch to pass all unit tests (detailed below) so that all development branches may branch from it.  To enforce the functional state of the development branch, all pushes to it will be mediated by pull request.  Before a pull request is merged with the development branch, the unit tests, distribution tests and model tests must pass.

**Master branch:** The `master` branch is always at our most recent, production-ready release.  Only production-ready releases should be pushed to this branch.  Each point on this branch will be tagged with the most recent version number.


#### 2.2 Non-Permanent Branches

**Feature branches:** All development work on *new* features that have not been released go in feature branches. Feature branches branch from `develop` and should be named `feature/issue-<number>-desc`.  After development is complete on a feature, submit a pull request back to `develop`.

<!-- DL: we haven't really used hotfix branches
**Hotfix branches:**  Patches to the current release go in hotfix branches. Hotfix branches branch from `master` and are named `hotfix/v<major>.<minor>.<patch>`.  There should be an issue created for the bug in Stan's GitHub issue tracker prior to branching to provide an issue number.    Once the patch is complete, submit a pull request back to `master` and it will be merged into both `master` and `develop` branches.
-->

**Bugfix branches:** All development work to *fix bugs* should branch from the latest hotfix branch (see above) and should be named `bugfix/issue-<number>-desc`

**Release branches:**  Work to complete a new release go in release branches. Release branches branch from `develop` and are named `release/<some-release-number>`.  Once a release branch is complete, it will be merged into both `master` and `develop`.

### 3.  Preventing Inadvertent Pushes

We are working on an honor system. The active developers with push permission on the Stan repository technically have the ability to push to the master and development branches, but are asked to take the following steps to prevent inadvertent pushes to the master and development branches.

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

*First*, clone the GitHub stan-dev/stan repository.

    > git clone https://github.com/stan-dev/stan.git
    > cd stan

*Second*, create a branch on which to work.  How this is done differs if you're developing a feature or a bug fix. 

The typical naming of a branch will call out the issue number from the issue tracker and include a short description, for example, ```feature/issue-12-sd-vectorize``` for a new feature specified in issue 12 that's about vectorizing the standard deviation function (this one's hypothetical).

For a *new feature*, checkout develop and get up to date.

    > git checkout develop
    > git pull
    > git checkout -b feature/foo
    > git push -u origin feature/foo

For a *bug fix*, checkout the latest hotfix branch and get up to date;  the first pull is there in case the hotfix version hasn't been pulled to the local Git.

    > git pull
    > git checkout hotfix/v2.0.2
    > git pull
    > git checkout -b bugfix/foo
    > git push -u origin bugfix/foo

The last push command is to make the branch public;  the `-u` option sets the upstream branch to the origin, so that git pull requests pull from the origin's version of the branch `feature/foo`.

This will be necessary in order to create a pull request to have the work merged back into the branch from which it was checked out.  (The first such push may require verifying the github credentials.)

*Third*, code and document using the usual git commands.  Collaboration is easy since all the developers have push access to the feature or hotfix branch you just created.

*Fourth*, when finished, create a pull request. A pull request indicates that the work is done and should be merged back to the appropriate branch.  Instructions on pull requests are in the next section.


### 5. Pull Requests

#### 5.1 Preconditions for Pull Requests

##### New and Altered Code is Tested

All new and altered code should be tested.

For new code, the developer should provide at least one unit test instantiating the code in a test harness. The developer should probably do more testing, but we are concerned saving time in the future if a bug appears in the new code.

For existing code, the developer should provide at least one test showing how the code changes. If the change is a refactoring, the test showing that behavior doesn't change might already exist.


##### Final State Passes All Tests

When a pull request is made, it is expected that the current code allows for all top-level targets in the makefile will build without error.  These targets include:

    > make manual
    > make doxygen
    > make test-headers
    > ./runTests.py src/test/unit
    > ./runTests.py src/test/integration
    > ./runTests.py src/test/prob
    
   
We don't always expect every developer to run all tests on their machine, but we do expect the developer to run a minimal number of tests before creating a pull request. If you've worked on `src/stan/foo/bar.hpp`, there should be a test in `src/test/unit/foo/bar_test.cpp` that passes. We also expect the developer to run `make src/test/unit/foo` prior to submitting.  The script [runTests.py](https://github.com/stan-dev/stan/wiki/Testing-Stan-using-Gnu-Make-and-Python) can be used to run the tests under `src/test`.

Passing all tests does not guarantee that the pull request will be accepted and merged.  It will first go through code review and tested on the [integration server](Continuous-Integration). 

##### Commit History is Clean

We strongly urge you to keep your commit history clean in the sense that each commit passes all tests.  This is also on the honor system---we are not going to enforce this by testing each commit in a pull request. 

If you prefer a personal workflow that involves committing changes that do not pass tests, it is possible to clean up your history.  Git provides a number of tools for "rewriting history," as discussed in Chacon's book, <i>Pro Git</i>, in the chapter "[Rewriting History](http://git-scm.com/book/en/Git-Tools-Rewriting-History)." In particular, a process known as rebasing (followed by a fresh commit) allows you to compose a sequence of previous commits into a single commit that is visible from the outside.


#### Information to Include in Pull Request

Pull requests should all include answers to all of the following questions or indicate why they're not relevant:

* Summary:  What does this pull request do in general terms?

Example:  Improve adaptation for NUTS.

* Intended Effect: What is the expected effect of the pull request?  Specifically, indicate how the behavior will be different than before the pull request.

Example:  Adaptation is broken down from one window into three, one for step-size adjustment and basic convergence, one for covariance matrix esimtation, and a final step-size adjustment.

* How to Verify: How can revieweres verify the request has the intended effect?  This should include descriptions of any new tests that are added or old tests that are updated.  It should also indicate how the code was tested.  

Example:  a new unit test was added for underlying behavior and two sample models which should converge now and didn't converge before. 

* Side Effects:  Does the pull request contain any side effects?  This should list non-obvious things that have changed in the request.

Example: fixed an existing bug in previous NUTS code that was needed for the new behavior;
non-obvious things that had to change to accomodate the request)

* Documentation:  If the pull request is user facing, how is it documented?  Are there examples of how to use the new behavior that users need to know about?  

Example:  added documentation for three new parameters in command, explained behavior in sampler description.  

* Reviewer Suggestions:  Who should look at the pull request for code review?  Barring a need for a specific reviewer, we'll try a round-robin assignment and attempt to balance the quantity of contributed code with the quantity of other code reviewed.

And finally, here's a <a href="https://github.com/stan-dev/stan/pull/430">complete example of a pull request</a> that follows these guidelines.

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

## 6.  How to create a hotfix branch for the next release

This should be done immediately after we release and then all bug fixes should go into the hotfix branch.  The develop branch is just for *new* features.

Suppose we have just released v2.0.1 so that v2.0.1 is the last tag on master.  The next patch branch will be hotfix/v2.0.2

```
> git checkout master
> git pull
> git checkout -b hotfix/v2.0.2
> git push -u origin hotfix/v2.0.2
```