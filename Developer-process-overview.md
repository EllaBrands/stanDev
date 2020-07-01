# Goals

The three main goals of the developer process are to support *correctness*, *maintainability* and *ease of development*.  To support correctness, all code must be unit tested for its main and edge case behavior against its API-level documentation.  To support maintainability, all code should be designed as simply and modularly as possible using idiomatic C++ style for understandability.  To support ease of co-operative development, the `develop` branch of every repository should always be in a releasable state.  To support both ease of development and maintainability, code should be written for future developers to read, with the aim of a future developer being able to easily figure out how to call a function and what its return value is.

# Process Checklist

## 1.  Create an issue on GitHub

Issues should be focused and have limited scope. Each issue should conceptually be one thing. It is common for a larger project to to be broken down into more than one issue or one issue. It is also common to have to start work on one issue and then generate multiple issues. Use your best judgement, but in most cases, smaller issues are better than bigger issues.
  
Did I mention we prefer more smaller issues than fewer large issues? 
  
## 2. Create a branch for the issue

In your local clone / fork of the Stan repo, create a branch from `develop`. (Branching from `develop` is the right move most of the time, but there are occasions where you might want to use another branch as the base branch.) Replace the `#` below with the issue number from above.
  - If the issue is a bug, then name the branch: `bugfix/issue-#-short-description`.
  - If the issue is a feature, then name the branch: `feature/issue-#-short-description`.

This should look like: 

```
> git checkout -b feature/issue-#-short-description
```

For more detailed information on our management of branches, see [Developer Process](https://github.com/stan-dev/stan/wiki/Dev:-Git-Process).
  
## 3. Fix the issue

Fix the issue on the branch that was created. This branch will be used to create a pull request.
  
Please limit the fixes on the branch to the issue. If you find yourself fixing other things on the branch: stop, create a new issue, and follow the process to fix that separate issue. Multiple conceptual fixes together on one branch will slow down the time it takes for it to get into the code base and may not get in as-is. 
  
If you're fixing a bug, first create a (unit) test that demonstrates the bug. This should fail on the branch without applying fix to code. Commit that test (`git commit` so it gets added to the history). Then fix the issue. The test failure should go away. **This test is a necessary condition for getting the patch in.**

If you're creating a new feature, then every new piece of code should be unit tested (with exceptions). Any sections that touch existing code should be tested. See the section on testing to get a feel for what we want tested.

**Tip:** The commit message for the final commit for the issue should have `fixes #N` where N is the issue number;  that will make sure the issue is closed when the pull request is merged (you can also use `closes` or `resolves` in place of `fixes`).
  
## 4. Create a pull request

Before a pull request can make it into the Stan `develop` branch, it needs to be up to standards. Not all of these apply for every pull request, but for most, it does:

  1. Testing
      - New / changed code must be tested. There should be new tests.
      - All header tests must pass: `> make test-headers` (this ensures each file has enough includes)
      - All unit tests must pass: `> runTests.py src/test/unit`
      - All integration tests must pass: `> runTests.py src/test/integration`
      - We have some performance tests in `src/test/performance`. These need to compile, but they may not pass locally: there's some configuration dependent, fragile test in there now.

  2. Documentation
      - All new / changed code must be documented. There should be doxygen documentation for code. If this affects something in the manual, that should be changed appropriately.
      - `> make doxygen` must report 0 warnings / errors.
      - `> make manual` must compile the manual successfully.

  3. Code Quality
      - As much of the code should be written idiomatic to either C++ or math. This is subjective, but it's important to remember we're working as a team and the person that writes it may not be the person that needs to fix it later.
      - We use cpplint to check for consistent C++ style. We have relaxed some of the rules; the description of the rules can be found in [Coding Style and Idioms](https://github.com/stan-dev/stan/wiki/Coding-Style-and-Idioms)
      - Run `> make cpplint`. Your branch should introduce 0 new cpplint errors. (We are currently at 69 as of 2.8.0 and are trying to get it to 0.)

Summary. In short, these things must pass in order for the pull request to go in. We would prefer if you checked before you submitted your pull request.
  
```
> make test-headers
> runTests.py src/test/unit
> runTests.py src/test/integration
> make doxygen
> make manual
> make cpplint
```
  
Once all of the prerequisites are taken care of, create a GitHub pull request. In most cases, the base branch with be `develop`. If it's a bugfix, the base branch might be the latest hotfix branch, but we haven't been using hotfix branches that often.
  
Create the pull request using the GitHub web interface. You will have to push your branch to `origin` (GitHub) first in order for the web interface to know about your branch. Fill out the Pull Request Template. Then hit submit.
  
Our continuous integration will then automatically run the tests and report back in the Conversation tab of the issue. 
  
## 5. Code review of pull request

Before anyone reviews a pull request, it must pass the tests on the continuous integration machines.  Note: the continuous integration tests sometimes fail due to configuration and is not a problem on the pull request. If you think this is the case, let someone know.
  
Someone on the Stan development team will look at the code for:

  - utility, or at least having the code do what you claim it does
  - tests. Making sure enough of the code is tested. Making sure code can be instantiated outside of running all of Stan. This also helps with design. If you can't verify that something does what you want it to do, something's not right.
  - idiomatic use of C++. Once again, this is subjective, but this should allow other people to help out with the code in the future.
  - documentation. Tests don't detect whether appropriate documentation was added. People can do that.

If the code doesn't pass tests, it probably won't get reviewed. If there's something tricky and you need help, the stan-dev mailing list is the right place to go. If you don't currently have permissions to post there, email `mc.stanislaw@gmail.com` asking for permissions on stan-dev.

## 6. Fix the branch, if required

That should be pretty straightforward.

## 7. Merge branch into development branch

Before merging, make sure:

  - tests pass
  - someone on the core team has reviewed the pull request thoroughly
  - someone on the core team has approved of the pull request

---------------------------------------

# Detailed Git Process

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


#### 4.1 Contributing to another organization's forked branch

Suppose someone makes a pull request to `stan-dev/stan` from a fork of the `stan` repository in their own GitHub organization.  If their organization is named `theirorg`, the fork will reside in `theirorg/stan` on a branch with a name like `feature/foo`.    That branch name will be listed on he pull request as the source of the requested merge.

First, you need to clone their repository:

    > git clone https://github.com/theirorg/stan

Then checkout their branch:

    > cd stan
    > git checkout feature/foo

Now you can work on their branch.  When you're ready to push your changes back to their branch, you can set up an alias `theirstan` the remote origin of the repository:

    > git remote add theirstan https://github.com/theirorg/stan

Then, all you need to do is push:

    > git push theirstan feature/foo

Yes, the permissions will just automatically let you do that.  [If that's not true if you don't have merge permissions on stan-dev, please include an updated process if you know what it is.]

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

# How to compile with debugging turned on

The Makefile of Math library provides a minimalist debug build, replacing `-O3` with `-g -O0` in `CXXFLAGS`.

To switch on debug build, add the following line in `make/local`
```bash
DEBUG = 1     # debug build
```
In addition to the usage of debuggers such as GDB, debug build turns on DEBUG macros, so in C++ code one can do
```c++
#ifdef DEBUG
// ...
#endif
```
Last but not least, `cout` and `typeid` is your friend.


# Adding a dependency to other software

We want to answer the following questions:

* What is the dependency used for?
* Why do we need or want this dependency?
* Is the license compatible with BSD? Does it allow us to distribute its source and binaries?
* How mature is the dependency?
  * Are there alternatives?
  * How often will we need to update the dependency? (How often has it changed over the last year?)
* How does this affect Math maintainers? 
  * Will we need to change any of the source before we include it into the Math library?
  * Is this dependency header-only? Static library? Shared library?
  * Do we need to change the build to get this to work?
* How does this affect:
  * Math developers?
  * Math users?
  * Stan?
  * Stan interface users?
  * Stan interface maintainers?
* Can we still support all the same [compilers we support](https://github.com/stan-dev/stan/wiki/Supported-C---Compilers-and-Language-Features) now?
* How much time does this add to compiling the first model with:
    * RStan?
    * CmdStan?
    * What about subsequent compile times?
* Where do we want to use this? Does any other source code need to change?
* How difficult would it be to write our own version of just the functionality we require?


# Code Review Guidelines

All pull requests must have these things:

1. clear licensing information. Check that the person that owns the copyright is listed.
2. the code base remains stable after the merge. Every pull request should pass all the continuous integration tests. There are a very few exceptions when they don't, but that's rare.
3. the changes proposed are maintainable by the community. We're an open-source project. Once the changes go into the code base, we're all responsible for the code. Please read the code with these things in mind:
    - overall readability. If it's hard to understand the code and it can be made simpler, please mention that.
    - code design. It should either be designed with well-known C++ patterns or there should be a good reason it isn't.
    - documentation. The documentation that goes with the changes should go in this pull request.
4. the changes are tested. Please verify that there's at least one new test that forces the code to execute. Please also verify there's at least one test that shows how to handle errors in the code. Having just these two tests (without full coverage) is usually enough to trap errors in the future. It is also enough to use these to refactor the code when the time comes.
5. the changes adhere to the Math library's [C++ standards](https://github.com/stan-dev/stan/wiki/Code-Quality). This is a large code base. We want the quality to be consistent so it's easy to navigate and understand other parts of the code base.

Most of the above is subjective, so please use your best judgement.

Pull requests should be narrow in scope. Please don't let inadvertent changes get into the code base.

## Who Can Review a Pull Request

It would help if anyone that's knowledgable in the code base reviews code. This is also for those that aren't active developers, but can read C++ fluently!


## Who Can Merge a Pull Request
Members of the Stan development team that contribute regularly to the Math library are allowed to merge the pull requests.

If all tests pass and the pull request has been reviewed, if no one merges, @syclik will do so (not on any schedule, but at least once a week).


## How to Review a Pull Request
Thanks! Here are some guidelines on how to review. At a high level, you're making sure the code submitted is indeed what the submitter says they have submitted. If an issue exists and is good, then it should be fixed. There are lots of ways to solve a problem -- we're not always looking for the perfect solution. We're looking for something that can get into the code base without putting a maintenance burden on other developers.

Feel free to review a narrow part of the pull request. By that, I mean one of these things: build process, numerical computation, template metaprogramming. If you're reviewing a small portion of the pull request, please mention it in the comment and only mark it as a **comment** (not as an "Approve"). In most cases, this won't matter, but in the ones that span across different concerns, one of the core developers will coordinate the review and approve.

1. Before starting, review the linked Math issue. That issue should describe what should happen. The pull request will be how that happens.
2. Wait for the continuous integration to complete and pass. If the tests don't pass, the rest is optional.
3. Review the pull request text:
    1. If any of the checklist is missing, please reject the pull request by leaving a comment for the submitter. If there's no response after some time, please close the pull request.
    2. If the checklist is complete, read the description of the pull request. If the description of the fix doesn't seem right, comment on the pull request.
    3. If the description of the tests don't seem like they are enough to keep the technical debt down, comment on the pull request asking for more tests. If you can, ask for specific tests.
4. Review the contents of the pull request. Dig into the code:
    1. Verify that what was described is indeed what's happening in the code.
    2. Spot check the tests to verify that they exist.
    3. Verify documentation exists.
    4. Check that the code is really readable. When you sign off, you're suggesting that someone else will be able to figure out the code to fix or modify it.
    3. Add line-specific comments when you can.
    4. Please check that there aren't other fixes or other code that sneaks into the pull request. Submitters shouldn't be sneaking in patches under the guise of a pull request.
5. "Review Changes" -- hit the green button and submit the review. We're playing within GitHub's rules, so we need to coordinate together. GitHub will allow the pull request to be merged when all continuous integration tests pass, there are no reviews that request changes, and at least one reviewer has approved. Pull requests are tricky because they can actually spawn across different concerns. Here's how we'll coordinate:
    1. **Request changes**. If there are changes that need to be made, finalize your review with "Request changes."
    2. **Comment**.  If you're only reviewing a portion of the pull request, e.g. numeric computation, finalize your review with "Comment" if everything is good.
    3. **Approve**. Finalize the pull request with "Approve" when the pull request is ready to be merged. If you're looking at the whole pull request, then feel free to approve when it's ready. If you're only looking at a part of the pull request, please do not approve. One of the core Math developers will coordinate the different reviewers and approve based on the feedback.

# Backwards compatibility requirements

## How do we keep consistent behavior in the Math library?

We try to keep the behavior of the functions in the Math library consistent for edge cases. Unfortunately, we implemented a lot of functions before we started realizing we should be consistent. That said, we prefer backwards compatibility to consistency -- we shouldn't change behavior until we get to a major version change. If we throw, we want well-formed messages because they do get passed back to the user from within Stan.

We follow these rules in order:
1. Existing functions should maintain their behavior.
2. Use the standard library's behavior. That means that we should be using the standard library functions for math if it exists.
3. Use Boost's math functions. If the error messages aren't nice, find some way to provide nice error messages.

# Supported Math Library API

### What is the supported Math API?
1. The Math Library header accessed via `stan/math.hpp`.
2. All math functions and classes provided by that header in the `stan::math` namespace are supported according to the API and usage listed in their doxygen doc.
3. How we call (reverse-mode) automatic differentiation (via the functionals `gradient` and `jacobian` found in  `rev/mat/functor/` followed by `recover_memory()`). 
4. [These compilers and OSes.](https://github.com/stan-dev/stan/wiki/Supported-C---Compilers-and-Language-Features).
5. Single-threaded, or parallelism via `map_rect`.

### What is not part of the supported API:
* Autodiff system internals - memory layout, the exact mechanism by which automatic differentiation is performed, etc.
* Anything in `stan::math::internal`.
* Anything missing a doc string.

### What does "supported" mean?
Just that we attempt to test that this API is maintained through time (or semantic version numbers updated accordingly). We also accept bugfixes and pull requests against this API for sure, though of course we also accept many pull requests on things that aren't officially supported yet.

### References

0. https://discourse.mc-stan.org/t/what-is-the-supported-external-api-of-the-math-library/7438


# Updating Libraries

Stan depends on a few libraries. Here are instructions for updating the libraries. This should happen pretty infrequently.

## Overview of Updating Libraries

All the source libraries we depend on are BSD (3-clause) compatible. Stan math currently depends on:

- Boost
- Eigen
- Sundials
- Google Test (for tests)
- OpenCL
- CppLint (for code style)
- TBB (for multi-threading)

To update one of these libraries, the steps are:

1. Create a [new issue](https://github.com/stan-dev/math/issues/new) stating which library needs to be updated.
2. Download the library source.
3. Unpack the source into `lib/<library_name>_<version>`
4. Modify the makefile reference to the library path. This will be found in `make/libraries` at the top of the file.
4. Git remove the old source and git add the new source. Make a git commit that says `"Updating <library> to version <version>"`.
5. Follow any specific instuctions for updating the particular library below. If the instructions call for pruning without modification, feel free to do that before making the commit.
6. Git commit the new changes.
7. Test to make sure everything still works. Please `make-clean-all` first. The makefiles aren't designed to find files when they have moved locations.
8. Git push and create a pull request.

Once it goes through the continuous integration, we can merge the pull request into `develop`. Please also update the library version number as listed in the `README.md` of `stan-math`.

## Boost

`std::logic_error("Section not yet written.")`

## Eigen

`std::logic_error("Section not yet written.")`

## Google Test

Google Test will not work directly with MPI. In order to make it work, please update `src/gtest_main.cc` so the main function looks like this:

```{c++}
GTEST_API_ int main(int argc, char **argv) {
#ifdef STAN_MPI
  // for MPI testing we test with all workers in listen mode. No	
  // output is generated from the workers.	
  stan::math::mpi_cluster cluster;	
  cluster.listen();	
  if (cluster.rank_ != 0)	
    return 0;
#endif
  printf("Running main() from %s\n", __FILE__);
  testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}

```

This is the only modification that is necessary.

## OpenCL

`std::logic_error("Section not yet written.")`

## CppLint

`std::logic_error("Section not yet written.")`

## Sundials (02/2019)

To upgrade Sundials:
1. download the target version to `lib/sundials-<version>.tar.gz` from [https://computation.llnl.gov/projects/sundials/sundials-software]
2. navigate to `lib/`
3. run from a terminal: `./upgrade-sundials.sh sundials-<version>.tar.gz`
4. after this is completed, verify that this works and then `git push`

Additional notes:
The official sundials source code contains printf, sprintf and fprintf
calls which we have to make optional when compiling the library. This
is needed due to restrictions imposed by CRAN which disallows these
function calls to even appear in any binary part of a R package. To
achieve this we modify the vanilla sundials source files such adequate
replacements are done.
