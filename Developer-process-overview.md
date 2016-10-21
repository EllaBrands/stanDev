## Why we have a process

We don't like process, but it's necessary for coordinating a development team.  We have multiple goals for our developer process:

- Ensure the proposed changes are maintainable
    - designed properly
    - uses consistent coding style
    - uses idiomatic C++ conventions (so other people can understand the code)
    - documented
- Keep the `develop` branch in a working state
    - passing all unit tests
    - documented at API level
- Make sure the code is understandable
    - another should be able to understand the intent of the code
    - another developer should have an easy time instantiating the code (unit tests)
    - the code should be modular enough to be changed without destroying all of Stan

## 1.  Create an issue on GitHub

Issues should be focused and have limited scope. Each issue should conceptually be one thing. It is common for a project to spawn multiple issues. It is also common to have to start work on one issue and then generate multiple issues. Use your best judgement, but in most cases, smaller issues are better than bigger issues.
  
Did I mention we prefer more smaller issues than fewer large issues? 

See: [Where do I create a new issue?](Where-do-I-create-a-new-issue)
  
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
      - We use cpplint to check for consistent C++ style. We have relaxed some of the rules; the description of the rules can be found in [Coding Style and Idioms](wiki/Coding-Style-and-Idioms)
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