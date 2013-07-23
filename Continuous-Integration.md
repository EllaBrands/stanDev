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
  | Build target  | ```git submodule update 
cd stan
git fetch
git checkout origin/develop
cd .. 
git submodule status
R -q -e "library(Rcpp); sessionInfo()"
cd rstan
make clean
make check && (cd tests & R -q -f runRunitTests.R --args ../rstan.Rcheck)```
  | Frequency     | daily
  | Artifacts     | none


**[Miscellaneous projects.](http://d1m1s1b1.stat.columbia.edu:8080/view/Miscellaneous/)** These are utility projects.

* [Stan Push Website](http://d1m1s1b1.stat.columbia.edu:8080/view/Miscellaneous/job/Stan%20Push%20Website/):

This project pushes the website to http://mc-stan.org

  |               |               |
  |---------------|---------------|
  | Build target  | N/A
  | Frequency     | on demand
  | Artifacts     | none

### 4. Testing Pull Requests through Jenkins
TODO: Write this section