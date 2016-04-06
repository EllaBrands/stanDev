# Background

Our Jenkins configuration isn't the best. I, Daniel, am trying to take inventory of our current situation so we can make it better. We've evolved our Jenkins config over the years. I'm the only one that's keeping tabs on it, for now. 

We are currently having problems with false positives (Jenkins failing in some way and this causing a slow-down in our development). If we could cut that down, that would be great.

My maintenance of Jenkins has been fairly minimal. Day-to-day, I tend not to look at Jenkins config. But when things break, I spend some time with it.

# GitHub repos

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


# Current jobs

We test each pull request for the three repositories. We also test every time a merge happens into the `develop` branch. Why do we do both? Honestly -- weird shit happens sometimes and even a merge that seems safe ends up breaking the `develop` branch. (It hasn't happened in a while, but it has happened.) I'd just rather know when that happens.

**Note:** we should really be testing more on Windows. Travis-ci tests Linux and Mac.


## Math

### Pull Request

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
   - [math dependenices](http://d1m1s1b1.stat.columbia.edu:8080/job/Math Pull Request - Warnings - Dependencies)
      - Build: `make test-math-dependencies`
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

## Stan

### Pull Request

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

## CmdStan

### Pull Request

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

### New commit on develop

[CmdStan](http://d1m1s1b1.stat.columbia.edu:8080/job/CmdStan)


## Misc jobs

These are jobs that don't pertain to any of the pull requests or builds when `develop` changes.

- [Stan daily performance](http://d1m1s1b1.stat.columbia.edu:8080/job/Daily%20Stan%20Performance/)

    This project runs the performance test on the Mac Pro when nothing else is running. It's run once a day and waits until the machine is free. I don't know how to programatically test performance. This is here so we can tell if something bad has happened with speed. It doesn't test nearly enough, but it's a start.
- [ScmSyncConfiguration test job](http://d1m1s1b1.stat.columbia.edu:8080/job/ScmSyncConfiguration%20test%20job/)

    The backup of the Jenkins config is on a github repo. This is just a test job to make sure the repo is reachable.

# Computers

## Current nodes

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


## Additional machines:

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

# Features

Here are some of the features we rely on. And some of the features we wish to have.

## Current features

- GitHub authentication
- Backup to git


## Wish list

- Better notification when build passes or fails. We used to have that through comments, but it's not working any more.
- Reduce failing builds due to GitHub being unreachable.
- Encrpytion across nodes?
- Better GitHub notification. Currently, all builds say: "Build finished. No test results found."
- Easier retest if something like a GitHub timeout caused the error.
- Make it easier to control the upstream project branch? For example, if we know a change to Stan will break CmdStan, I have to manually go into the project and add put in the branch name.
- Update the upstream branch script needs a lot of work. I want to update the existing issue and pull request if it's still open. Currently, it just adds a new issue and a new pull request.
- Test that the submodule is pointed to some place on develop and make sure to update to latest develop before merging.
