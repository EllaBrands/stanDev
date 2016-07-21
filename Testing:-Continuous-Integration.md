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
