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
TODO: Write this section

### 4. Testing Pull Requests through Jenkins
TODO: Write this section