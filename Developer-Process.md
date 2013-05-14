### Contributing Code

**Rule 1.** Branch `master` is in a releasable state.

**Rule 2.** All contributions come through pull requests. 

1. Anyone can [create a pull request](https://help.github.com/articles/creating-a-pull-request). We need you to explicitly assign copyright to the Stan development team.

2. A pull request should be unit-tested, `make test-unit`, at the very least. If your code touches anything that goes into distributions, a pull request should test the distributions, `make test-distributions`. If your code touches the modeling language, you should run the model tests, `make test-models`.

3. Anyone can do an informal code review for a pull request and leave comments on the pull request's page on GitHub.

4. One of the admins can start the tests on [Jenkins](http://d1m1s1b1.stat.columbia.edu:8080) (this may take hours).

5. One of the admins can declare code review done and push the patch to `master`.
