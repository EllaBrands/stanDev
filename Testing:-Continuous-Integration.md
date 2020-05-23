### 1. Stan's Continuous Integration Server
http://d1m1s1b1.stat.columbia.edu:8080

The server is a desktop computer running Windows 7. It has an Intel Core i7 920 @ 2.67 GHz processor with 8 GB of RAM. It is located in Daniel, Bob, and Sophie's office at Columbia University.

We use [Jenkins](http://jenkins-ci.org/) as our continuous integration server.

Daniel is currently maintaining the server.

### 2. Goals of Continuous Integration
We use our continuous integration server to assist our [development process](Developer-Process). Here are some of our goals for our continuous integration:
* Maintaining the state of the `develop` branch. We test the `develop` branch on every merged pull request. If a merge ends up breaking the build, we are able to detect it rather quickly.
* Verifying pull requests. Before pull requests are merged into the `develop` branch, Jenkins is used to verify that the tests pass.
* Cross platform testing. Most of the developers are on Mac or Linux machines. Running the server in Windows catches some of the cross platform issues. Ideally, we would set up Jenkins slave processes on each of our target platforms, but we currently do not have that set up.

For additional information on continuous integration, see the [Wikipedia entry](https://en.wikipedia.org/wiki/Continuous_integration).

### 3. Jenkins Projects

Here are the current Jenkins projects:


**[Stan Develop Branch.](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/)** These projects test the current state of the `develop` branch.

* [Doxygen](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Doxygen/):

    This project verifies that the C++ API documentation can be built and tracks Doxygen warnings.

    |               |               |
    |---------------|---------------|
    | Build target  | `make doxygen`
    | Frequency     | daily, when changes are made to the `develop` branch
    | Artifacts     | [current API doc](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Doxygen/ws/doc/api/html/index.html)

* [Manual](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Manual/): 

    This project verifies that the manual can be built.

    |               |               |
    |---------------|---------------|
    | Build target  | `make manual`
    | Frequency     | daily, when changes are made to the `develop` branch
    | Artifacts     | [draft version of the manual](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Manual/ws/doc/)

* [Unit Tests](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Unit%20Tests/): 

    This project executes and tracks the unit tests and tracks compiler warnings. On success, calls the Header Tests.

    |               |               |
    |---------------|---------------|
    | Build target  | `make test-unit O=0 -j4`
    | Frequency     | every push to `develop` and daily
    | Artifacts     | none

* [Header Tests](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Header%20Tests/):

    This project executes the header tests and tracks compiler warnings. On success, calls the Distribution Tests.

    |               |               |
    |---------------|---------------|
    | Build target  | `make test-headers -j4 O=0`
    | Frequency     | called when Unit Tests reports success
    | Artifacts     | none

* [Distribution Tests](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan Develop Branch/job/Stan Distribution Tests):

    This project executes and tracks the distribution tests and tracks compiler warnings.

    |               |               |
    |---------------|---------------|
    | Build target  | `make test-distributions -j4 O=0`
    | Frequency     | called when Header Tests reports success
    | Artifacts     | none

* [Model Tests](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Model%20Tests/):

    This project executes and tracks the model tests and tracks compiler warnings. These tests are currently used as an integration test that will indicate whether Stan is completely broken.
 
    |               |               |
    |---------------|---------------|
    | Build target  | `make test-models`
    | Frequency     | daily
    | Artifacts     | time to execute each model is recorded. The files can be found here: [timing files](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Develop%20Branch/job/Stan%20Model%20Tests/ws/)


**[Stan Pull Requests.](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Pull%20Requests/)** These projects test incoming pull requests before they are merged into `develop`.

* [Stan GitHub Pull Requests Project](http://d1m1s1b1.stat.columbia.edu:8080/view/Stan%20Pull%20Requests/job/Stan%20Github%20Pull%20Requests/):

    This project tests pull requests. See the [Testing Pull Requests through Jenkins]() section for more information. The workspace starts with a current copy of `develop`, merges the pull request, then runs the suite of tests. Compiler warnings are tracked.

    |               |               |
    |---------------|---------------|
    | Build target  | `make manual doxygen && make test-unit test-headers test-distributions test-models -j2 O=3`
    | Frequency     | [see Testing Pull Requests through Jenkins]()
    | Artifacts     | none


**[rstan.](http://d1m1s1b1.stat.columbia.edu:8080/view/rstan/)** These projects test the current state of the `rstan` repository.

* [rstan build](http://d1m1s1b1.stat.columbia.edu:8080/view/rstan/job/rstan%20build/):

    This project tests pull requests. See the [Testing Pull Requests through Jenkins]() section for more information. The workspace starts with a current copy of `develop`, merges the pull request, then runs the suite of tests. Compiler warnings are tracked.

    |               |               |
    |---------------|---------------|
    | Build target  | `git submodule update` <br> `cd stan` <br> `git fetch` <br> `git checkout origin/develop` <br> `cd ..` <br> `git submodule status` <br> `R -q -e "library(Rcpp); sessionInfo()"` <br> `cd rstan` <br> `make clean` <br> `make check && (cd tests & R -q -f runRunitTests.R --args ../rstan.Rcheck)`
    | Frequency     | daily
    | Artifacts     | none


**[Miscellaneous projects.](http://d1m1s1b1.stat.columbia.edu:8080/view/Miscellaneous/)** These are utility projects.

* [Stan Push Website](http://d1m1s1b1.stat.columbia.edu:8080/view/Miscellaneous/job/Stan%20Push%20Website/):

    This project pushes the website from branch `master` to http://mc-stan.org

    |               |               |
    |---------------|---------------|
    | Build target  | N/A
    | Frequency     | on demand
    | Artifacts     | none

### 4. Testing Pull Requests through Jenkins

* We use our continuous integration server to test pull requests prior to merging into our `develop` branch. The [plugin documentation](https://wiki.jenkins-ci.org/display/JENKINS/Github+pull+request+builder+plugin) is sparse. We describe how we use the plugin.
* The [Stan GitHub Pull Requests project](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Github%20Pull%20Requests/) is controlled through comments on the pull request.
* We created a GitHub user called `stan-buildbot`. This is how Jenkins communicates through comments.
* We maintain a list of admins and whitelisted GitHub users on Jenkins.


#### Overview:

##### Steps for whitelisted contributors:
1. A whitelisted contributor creates a pull request.
2. Jenkins automatically places the pull request on the testing queue.
3. After the pull request is tested, `stan-buildbot` posts the status of the build as a comment.
4. An admin can retest the pull request by posting a comment.

##### Steps for other contributors.
1. A contributor creates a pull request.
2. Jenkins recognizes the pull request. `stan-buildbot` posts a comment on the pull request requesting verification from an admin.
3. An admin places the pull request on the testing queue.
4. After the pull request is tested, `stan-buildbot` posts the status of the build as a comment.
5. An admin can retest the pull request by posting a comment.



#### Steps in Detail

##### Creating a pull request.

All contributors need to create a GitHub pull request from their feature branch to the `stan-dev:stan:develop` branch. This is done through GitHub.


##### Non-whitelisted contributors: Jenkins recognizes the pull request.

When Jenkins recognizes the pull request, the `stan-buildbot` GitHub user will respond to the pull request with:
> Can one of the admins verify this patch?

Jenkins is now waiting for an admin to approve the pull request for testing. An admin can do this by commenting on the GitHub pull request with:
> Jenkins, ok to test.

Jenkins will then place the pull request in the queue for testing.


##### `stan-buildbot` posts the status of the build as a comment.

Jenkins will merge the pull request into a clean `develop` branch. The pull request needs to be able to be merged automatically.

After running the tests, `stan-buildbot` will comment on the pull request with either:
> Test PASSed. <br>
> Refer to this link for build results: http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Github%20Pull%20Requests/###/

or 

> Test FAILed. <br>
> Refer to this link for build results: http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Github%20Pull%20Requests/###/

If the tests pass and the pull request passes a code review, one of the developers should merge the pull request. After the pull request is merged, this will kick off another round of tests.


##### An admin can retest the pull request by posting a comment.

An admin may want to retest the pull request. A couple common reasons would include: 
1. Updated pull request.
2. Updated `develop` branch.

To retest the pull request, an admin can comment:
> Jenkins, retest this please. 

The pull request will be placed back on the testing queue. Jenkins will respond with a comment indicating pass or fail as above.


#### Note: pull requests originating from a whitelisted user

Every subsequent push to the originating feature branch will put the pull request on the testing queue. This happens for every push, tagged with the commit. If the feature branch is still active, this could result in the same pull request being tested multiple times holding up other pull requests from being tested.


#### Jenkins details:

##### Lists of GitHub Users

* Admins:
  * bob-carpenter
  * syclik
* Whitelist:
  * all admins are automatically whitelisted
* Default messages (these can be reconfigured):
  * Request for testing phrase: `Can one of the admins verify this patch?`
  * Success: `Test PASSed`.
  * Failure: `Test FAILed`.

If you would like to be added to either the whitelist or admin list, please get in touch with Daniel.

##### Admin Commands

* Accept to test phrase:
    
    Jenkins will comment: `Can one of the admins verify this patch?`

    An admin can accept the pull request with:
    > Jenkins, ok to test.

    The acceptance regex: `.*ok\W+to\W+test.*`

* Test phrase:

    To retest the pull request, an admin can comment:
    > Jenkins, retest this please.

    The test regex: `.*test\W+this\W+please.*`
   
    Although an admin can just type `test this please`, it could be misinterpreted as an imperative statement directed to the creator of the pull request.

* Add to whitelist:

    To add the contributor to the whitelist, an admin can either manage the whitelisted users through Jenkins or respond with:
    > Jenkins, add to whitelist.

    The add to whitelist regex: `.*add\W+to\W+whitelist.*`


## Restarting a Travis job
To restart a Travis job:

1. Log into https://travis-ci.org with your GitHub credentials. It'll pull up all the repos you have access to on the left.
2. Find the job that's broken. Here's an example: I clicked on the "stan-dev/stan" link (in green) on the left. Then I clicked on "Branches". Here, you can see that "feature/issue-2012-make" has an error. 
<Screen Shot 2016-08-17 at 10.59.27 AM.png>
3. Click on the red "#1750 errored" link. That's the build that's broken. It'll bring up a view of all the build jobs that it kicked off. I've scrolled down a little here to show a couple of the broken build jobs:
<image.png>
4. Click on one of the red links. Here, I clicked on "1750.18". It'll pull up detail of the build. I added the green arrows. 
<image.png>
5. Restart! Click on the the icon I've marked with green arrows. The hover-over tool-tip says "Restart job"

---------------------------------

## Jenkins

For a tutorial on interacting with the current Jenkins jobs, please see this discourse post:
http://discourse.mc-stan.org/t/new-jenkins-jobs-tutorial/2383

The rest of this wiki is historical.

### Background

Our Jenkins configuration isn't the best. I, Daniel, am trying to take inventory of our current situation so we can make it better. We've evolved our Jenkins config over the years. I'm the only one that's keeping tabs on it, for now. 

We are currently having problems with false positives (Jenkins failing in some way and this causing a slow-down in our development). If we could cut that down, that would be great.

My maintenance of Jenkins has been fairly minimal. Day-to-day, I tend not to look at Jenkins config. But when things break, I spend some time with it.

### GitHub repos

These are the repositories tested using Jenkins:

- [Math](github.com/stan-dev/math)
- [Stan](github.com/stan-dev/stan)
- [CmdStan](github.com/stan-dev/cmdstan)

There is also a dependency between the repos managed by git submodules:
```
math  <-  stan  <- cmdstan
```

Math is standalone.

Math is a Stan submodule. It's located at `./lib/stan_math`.

Stan is a CmdStan submodule. It's located at `./stan`.


### Current jobs

We test each pull request for the three repositories. We also test every time a merge happens into the `develop` branch. Why do we do both? Honestly -- weird shit happens sometimes and even a merge that seems safe ends up breaking the `develop` branch. (It hasn't happened in a while, but it has happened.) I'd just rather know when that happens.

**Note:** we should really be testing more on Windows. Travis-ci tests Linux and Mac.


### Math

#### Pull Request

Link to Jenkins job: [Math Pull Request](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request/)

This is a MultiJob project (that's what Jenkins calls it).

1. Phase: Documentation
   - [Doxygen](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Doc%20-%20Doxygen/)
      - Requires: doxygen
      - Build: clean, then `make doxygen`
      - Post-build: scan for compiler warnings (Doxygen).
2. Phase: Warnings
   - [cpplint](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Warnings%20-%20CppLint/)
      - Requires: python2.7. Cpplint doesn't run on Python 3
      - Build: `make cpplint || echo '--- cpplint has errors ---'` (mask the error message so the post-build action happens)
      - Post-build: scan for compiler warnings (CppLint)
   - math dependenices
      - Build: `make test-headers`
      - Post-build: scan for compiler warnings (math-dependencies)
      - The parser for `math-dependencies` is a custom one written in Jenkins. Here's what it looks like:
         - Name: `math-dependencies`
         - Link name: `mc-stan.org`
         - Trend report name: `math-dependencies`
         - Regular expression: `^(.*)\((.*)\):\s(.*\.)\s*\[(.*)\]\s*$`
         - Mapping script: 
                
                ```
                import hudson.plugins.warnings.parser.Warning

                String fileName = matcher.group(1);
                String lineNumber = matcher.group(2);
                String category = matcher.group(4);
                String message = matcher.group(3);

                return new Warning(fileName, Integer.parseInt(lineNumber), "Dynamic Parser", category, message);
                ```
         - Example log message: `src/stan/math/prim/scal/meta/OperandsAndPartials.hpp(21): File includes a stan/math/rev header file. [prim]`
3. Phase: Quick tests
   - [Unit tests](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Tests%20-%20Unit/)
      - Build: `./runTests.py -j${PARALLEL} test/unit`
      - Post-build: 
          - scan for compiler warnings (GNU C Compiler 4 (gcc))
          - publish junit test report (test report xml: `test/**/*.xml`)
      - There are ~6000 tests
   - [Header tests](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Tests%20-%20Header/)
       - Build: `make -j${PARALLEL} test-headers`
       - Post-build: scan for compiler warnings (GNU C Compiler 4 (gcc))
4. Phase: Long test
    - [Distribution tests](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Tests%20-%20Distribution/)
       - Builds only on gelman-group-mac. Compiled tests take up a lot of space... something like ~200 GB.
       - Also takes a lot of time. About 5.5 hours.
       - Build:
            - Conditional step. Execute shell: 
                
                ```
                ## searches for changes to source in the Math Library stan/math
                git diff --name-only $sha1 $(git rev-parse origin/develop) | grep -c stan/math
                ```
                
            - run:
                 
                 ```
                 echo 'CC=clang++' > make/local
                 echo 'O=0' >> make/local
                 ./runTests.py -j${PARALLEL} test/prob
                 ```
                 
       - Not only are the executables and the output large, this generates a lot of test results. Something like 600k.
5. Phase: Upstream projects.
   - [Stan](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Upstream%20-%20Stan/)
     - This takes the branch specified in `stan_branch`, which defaults to `develop`, and the pull request's sha, in `math_sha`, and merges the math pull request with develop. Then it runs the Stan tests.
     - Build
        - prep:
            
            ```
            make math-update/develop
            cd lib/stan_math
            git merge ${math_sha} --ff
            ```
            
        - run: `./runTests.py src/test`
     - Post-build: publish junit report.
   - [CmdStan](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20Pull%20Request%20-%20Upstream%20-%20CmdStan/)
     - This takes the branch specified in `cmdstan_branch` (default `develop`), `stan_branch` (default `develop`), and the pull request's sha, in `math_sha`, and merges the math pull request with develop. Then it runs the CmdStan tests.
     - Build
        - prep:

                ```
                make stan-update/${stan_branch}
                cd stan
                make math-update/develop
                cd lib/stan_math
                git merge ${math_sha} --ff
                cd ..
                ```
                
        - `./runCmdStanTests.py src/test`
     - Post-build: publish junit report.

### New commit on develop

Link to [Math](http://d1m1s1b1.stat.columbia.edu:8080/job/Math/).

1. Update Stan
    - [Update Stan](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Upstream%20-%20Update%20Stan/)
   
      This is my really poor attempt at updating Stan's submodule now that Math has been updated.
      It grabs a script I wrote from https://github.com/syclik/stan-scripts/blob/master/jenkins/create-stan-pull-request.sh and runs it. 
2. Documentation
    - [Doxygen](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Doc%20-%20Doxygen/)
3. Warnings
    - [Cpplint](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Warnings%20-%20CppLint/)
    - [math-dependencies](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Warnings%20-%20Dependencies/)
4. Tests
    - [Header](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Tests%20-%20Header/)
    - [Unit](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Tests%20-%20Unit/)
    - [Distribution](http://d1m1s1b1.stat.columbia.edu:8080/job/Math%20-%20Tests%20-%20Distribution/)

Note: no upstream tests.

### Stan

#### Pull Request

This is also a MultiJob project. Link to [Stan Pull Request](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request/).

1. Documentation
    - [Doxygen](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Doc%20-%20Doxygen/)
        - build: `make doxygen`
    - [Reference Manual](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Doc%20-%20Reference%20Manual/)
        - build: `make manual`
2. Stan Tests
    - [Header](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Tests%20-%20Header/)
        - build: `make test-headers`
    - [Unit](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Tests%20-%20Unit/)
        - build: `./runTests.py -j${PARALLEL} src/test/unit`
        - post-build: 
            - scan for compiler warnings (GNU C Compiler 4 (gcc))
            - Publish junit test report: ~1100
    - [Integration](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Tests%20-%20Integration/)
        - build: `./runTests.py -j${PARALLEL} src/test/integration`
        - post-build:
            - scan for compiler warnings (GNU C Compiler 4 (gcc))
            - Publish junit test report: ~60
    - [Windows Unit](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Windows%20-%20Tests%20-%20Unit/)
        - build: `.\runTests.py -j%PARALLEL% src/test/unit`
        - post-build:
            - scan for compiler warnings (GNU C Compiler 4 (gcc))
            - Publish test report: ~1100
3. Warnings
    - [Cpplint](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20CppLint/)
        - build: `make cpplint || echo '--- cpplint has errors ---'`
        - post-build: scan for compiler warnings (CppLint)
4. Performance
    - [Performance](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Performance/)
        - This is really more like a regression test. This checks a few things:
            - the Stan program still compiles
            - when repeated, results are the same given a seed
            - results are the same given a seed compared to some reference value (only useful on a single machine with the same settings)
            - shows timing. want to make sure we're not completely messing up.
      - build:
      
            ```
            ./runTests.py src/test/performance
            cd test/performance
            RScript ../../src/test/performance/plot_performance.R 
            ```
            
       - post-build:
           - archive artifacts: test/performance/performance.csv, test/performance/performance.png
           - publish junit report

5. Upstream projects (envision a few more in the future, but maybe not.)
    - [CmdStan](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Pull%20Request%20-%20Upstream%20-%20CmdStan/)
        - build:
            - update math library:

                ```
                git clean -d -x -f
                git submodule foreach --recursive git reset --hard
                make stan-update/develop
                cd stan
                git pull --ff
                git merge ${stan_sha} --ff
                make math-revert
                ```
            - run: `./runCmdStanTests.py src/test`
        - post-build:
            - publish junit report: ~50


### New commit on develop

Link to [Stan](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan/).

1. Update CmdStan
    - [Update CmdStan](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Upstream%20-%20Update%20CmdStan/)

2. Documentation
    - [Doxygen](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Doc%20-%20Doxygen/)
    - [Reference manual](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Doc%20-%20Reference%20Manual/)
3. Tests
    - [Unit](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Tests%20-%20Unit/)
    - [Header](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Tests%20-%20Header/)
    - [Integration](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Tests%20-%20Integration/)
4. Warnings
    - [Cpplint](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20CppLint/)
5. Performance
    - [Performance](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20-%20Performance/)
6. [Windows](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Windows/)
    - [Header](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Windows%20-%20Tests%20-%20Header/)
    - [Integration](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Windows%20-%20Tests%20-%20Integration/)
    - [Unit](http://d1m1s1b1.stat.columbia.edu:8080/job/Stan%20Windows%20-%20Tests%20-%20Unit/) 

Note: no upstream tests.

### CmdStan

#### Pull Request

A simple project: [CmdStan](http://d1m1s1b1.stat.columbia.edu:8080/job/CmdStan%20Pull%20Request/)

- build:
    - update math:

        ```
        git clean -d -x -f
        make clean-all
        git submodule update --init --recursive
        make stan-revert
        make stan-update
        make
        ```
        
    - run tests: `./runCmdStanTests.py src/test/interface`
    - build manual: `make manual`
- post-build:
    - scan for compiler warnings (GNU Make + GNU C Compiler (gcc))
    - publish junit test report: ~50
    - publish performance test report. We never look at this.

#### New commit on develop

[CmdStan](http://d1m1s1b1.stat.columbia.edu:8080/job/CmdStan)


### Misc jobs

These are jobs that don't pertain to any of the pull requests or builds when `develop` changes.

- [Stan daily performance](http://d1m1s1b1.stat.columbia.edu:8080/job/Daily%20Stan%20Performance/)

    This project runs the performance test on the Mac Pro when nothing else is running. It's run once a day and waits until the machine is free. I don't know how to programatically test performance. This is here so we can tell if something bad has happened with speed. It doesn't test nearly enough, but it's a start.
- [ScmSyncConfiguration test job](http://d1m1s1b1.stat.columbia.edu:8080/job/ScmSyncConfiguration%20test%20job/)

    The backup of the Jenkins config is on a github repo. This is just a test job to make sure the repo is reachable.

### Computers

#### Current nodes

Master (d1m1s1b1.stat.columbia.edu):

  - iMac. OSX 10.9.5
  - 2.93 GHz Intel Core i7 (4 core)
  - 16 GB
  - 250 GB

gelman-group-mac (gelman-group-mac.stat.columbia.edu):

  - Mac Pro. OSX 10.11.1
  - 3 GHz 8-Core Intel Xeon E5
  - 64 GB
  - 500 GB

Windows Box (gelman.stat.columbia.edu):

  - Dell. Windows 7 Enterprise
  - Intel Xeon CPU E5507 @ 2.27 GHz 2.26 GHz (2 processors) (shows as 8 cores)
  - 12 GB
  - 464 GB

gelman-group-win (gelman-group-win.stat.columbia.edu):

  - HP Z640 Workstation. Windows 7 Professional
  - Intel Xeon CPU E5-2630 @ 2.40 GHz (shows as 16 cores)
  - 32 GB
  - 918 GB


#### Additional machines:

unamed-server:

  - HP Z640 Workstation.
  - Intel Xeon CPU E5-2630 @ 2.40 GHz
  - 32 GB
  - 918 GB

Andrew's broken laptop:

  - Macbook Pro Retina
  - 2.3 GHz Intel Core i7
  - 16 GB
  - 153 GB

### Features

Here are some of the features we rely on. And some of the features we wish to have.

#### Current features

- GitHub authentication
- Backup to git


#### Wish list

- Better notification when build passes or fails. We used to have that through comments, but it's not working any more.
- Reduce failing builds due to GitHub being unreachable.
- Encrpytion across nodes?
- Better GitHub notification. Currently, all builds say: "Build finished. No test results found."
- Easier retest if something like a GitHub timeout caused the error.
- Make it easier to control the upstream project branch? For example, if we know a change to Stan will break CmdStan, I have to manually go into the project and add put in the branch name.
- Update the upstream branch script needs a lot of work. I want to update the existing issue and pull request if it's still open. Currently, it just adds a new issue and a new pull request.
- Test that the submodule is pointed to some place on develop and make sure to update to latest develop before merging.
