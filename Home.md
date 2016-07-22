Welcome to the Stan wiki. This page is primarily intended for developers, both active and new to Stan.

## Useful Links

* [Dev: Git Process](wiki/Dev:-Git-Process)
* [Coding Style and Idioms](wiki/Coding-Style-and-Idioms)
* [Issue Mover for GitHub](https://github-issue-mover.appspot.com/)

# Contents

* [Developer Process](#developer-process)
* [Model Concept](#model-concept)
* [Feature Design Docs](#stan-feature-specs-and-design-docs)
* [Long Term To-Do List](#long-term-to-do-list)

[//]: # (DL: Rearrange. I think it makes sense to split this section into)
[//]: # (DL:   two parts: common information to be accessed by current)
[//]: # (DL:   developers, a second part for process that's useful for )
[//]: # (DL:   new developers)

# Developer Process

We don't like process, but it's necessary for coordinating a development team. Process, itself, is fluid and we adjust it when it is either a bottleneck or it fails to capture some practical development process.

We have multiple goals for our process:

- introduce changes to Stan
- ensure the proposed changes are maintainable:
    - designed properly
    - uses consistent coding style
    - uses idiomatic C++ conventions (so other people can understand the code)
    - documented
- keep the `develop` branch in a working state
- "good enough" for the next person to be able to:
    - understand the intent of the code
    - have an easy time instantiating the code (unit tests)
    - feel like it can be changed without destroying all of Stan

The next section describes the process, then we'll describe testing.


## The Process

1. **Create an issue on GitHub.**

  Issues should be focused and have limited scope. Each issue should conceptually be one thing. It is common for a project to spawn multiple issues. It is also common to have to start work on one issue and then generate multiple issues. Use your best judgement, but in most cases, smaller issues are better than bigger issues.
  
  Did I mention we prefer more smaller issues than fewer large issues? 

  See: [Where do I create a new issue?](Where-do-I-create-a-new-issue)
  
2. **Create a branch for the issue.**

  In your local clone / fork of the Stan repo, create a branch from `develop`. (Branching from `develop` is the right move most of the time, but there are occasions where you might want to use another branch as the base branch.) Replace the `#` below with the issue number from above.
  - If the issue is a bug, then name the branch: `bugfix/issue-#-short-description`.
  - If the issue is a feature, then name the branch: `feature/issue-#-short-description`.

  This should look like: 
  `> git checkout -b feature/issue-#-short-description`

  For more detailed information on our management of branches, see [Developer Process](wiki/Developer-Process).
  
3. **Fix the issue.**

  Fix the issue on the branch that was created. This branch will be used to create a pull request.
  
  Note: limit the fixes on the branch to the issue. If you find yourself fixing other things on the branch: stop, create a new issue, and follow the process to fix that separate issue. Multiple conceptual fixes together on one branch will slow down the time it takes for it to get into the code base and may not get in as-is. 
  
  If you're fixing a bug, first create a (unit) test that demonstrates the bug. This should fail on the branch without applying fix to code. Commit that test (`git commit` so it gets added to the history). Then fix the issue. The test failure should go away. **This test is a necessary condition for getting the patch in.**

  If you're creating a new feature, then every new piece of code should be unit tested (with exceptions). Any sections that touch existing code should be tested. See the section on testing to get a feel for what we want tested.
  
4. **Prerequisites for creating a pull request.**

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
      - We use cpplint to check for consistent C++ style. We have relaxed some of the rules; the description of the rules can be found in [Coding Style and Idioms](wiki/Coding-Style-and-Idioms)
      - Run `> make cpplint`. Your branch should introduce 0 new cpplint errors. (We are currently at 69 as of 2.8.0 and are trying to get it to 0.)

  Summary. In short, these things must pass in order for the pull request to go in. We would prefer if you checked before you submitted your pull request.
  
  - `> make test-headers`
  - `> runTests.py src/test/unit`
  - `> runTests.py src/test/integration`
  - `> make doxygen`
  - `> make manual`
  - `> make cpplint`
  

4. **Create a pull request.**
  
  Once all of the prerequisites are done, create a GitHub pull request. In most cases, the base branch with be `develop`. If it's a bugfix, the base branch might be the latest hotfix branch, but we haven't been using hotfix branches that often.
  
  Create the pull request using the GitHub web interface. You will have to push your branch to origin (GitHub) first in order for the web interface to know about your branch. Fill out the Pull Request Template. Then hit submit.
  
  Our continuous integration will then take the pull request and run the tests.
  
5. **Code review of pull request.**

  Before anyone reviews a pull request, it must pass the tests on the continuous integration machines. Note: the continuous integration tests sometimes fail due to configuration and is not a problem on the pull request. If you think this is the case, let someone know.
  
  Someone on the Stan team will look at the code for:

  - usefulness, or at least having the code do what you claim it does
  - tests. Making sure enough of the code is tested. Making sure code can be instantiated outside of running all of Stan. This also helps with design. If you can't verify that something does what you want it to do, something's not right.
  - idiomatic use of C++. Once again, this is subjective, but this should allow other people to help out with the code in the future.
  - documentation. Tests don't detect whether appropriate documentation was added. People can do that.

  Note: if it doesn't pass tests, it won't get reviewed. If there's something tricky and you need help, the stan-dev list is the right place to go. If you don't currently have permissions to post there, email mc.stanislaw@gmail.com asking for permissions on stan-dev.

6. **Fix the branch, if required.**

  That should be pretty straightforward.

7. **It gets merged**

  Right now, it's @syclik that merges things into `develop`. (There's not official gate-keeper and it doesn't always have to be @syclik.) The things he looks for:
  - tests pass
  - someone on the core team has reviewed the pull request thoroughly
  - someone on the core team has approved of the pull request



### Testing

* [Unit Testing with Python](wiki/Testing-Stan-using-Gnu-Make-and-Python)
* [Continuous Integration Testing](wiki/Continuous-Integration)
* [How to Write Unit Tests](wiki/How-to-Write-Unit-Tests-with-GoogleTest)
* [Code Quality Requirements](wiki/Code-Quality)
* [Coding Style and Idioms](wiki/Coding-Style-and-Idioms)
* [Include-what-you-use](wiki/include-what-you-use)


### General Notes for Developer Process

The Git and code review process used for developers and some useful tools:

* [How to Contribute a New Function to Stan](wiki/Contributing-New-Functions-to-Stan)
* [Developer Tools and Tricks](wiki/Developer-Tricks)
* [OS and Platform Detection Macros for C++](wiki/Compiler-and-OS-Detection-Macros-for-Cpp)

---

# Model Concept

Explanation of the code generated for a model:

* [Stan Model Concept](wiki/Model-Concept) 

---

# Stan Feature Specs and Design Docs

Specs and design docs and for Stan features, as originally proposed.  See Stan developer group mailing list for more discussion around specific features.

* [ODE Design](wiki/ODE-Integrator-Support)

* [MLE and MML Design](wiki/MLE-and-MML-Design)

* [Command Refactor Spec](wiki/Stan-API-Refactor)

* [Testing framework](wiki/Testing-framework)

* [Ragged array spec](wiki/Ragged-array-spec)


---

# Long Term To-Do List

This is complementary to the issue tracker of smaller tidbits:

* [Longer-Term To-Do List](wiki/Longer-Term-To-Do-List)

### C++11 Migration Plan

* [C++11 Migration Plan](wiki/Cpp11-Upgrade)

### Out-of-Date To-Do List

GitHub's issue tracker is more up to date and contains a `project` tag if you're looking for projects. An older list of to-do items, with some more ambitious projects, is here:

* [Stan To-Do List](wiki/To-Do-List)
