## Contributing Code

**Rule 1.** Branch `master` is in a releasable state.

**Rule 2.** All contributions come through pull requests. 

1. Anyone can [create a pull request](https://help.github.com/articles/creating-a-pull-request).

2. A pull request should be unit-tested, `make test-unit`, at the very least. If your code touches anything that goes into distributions, a pull request should test the distributions, `make test-distributions`.

3. Anyone can code review a pull request. [Jenkins](http://d1m1s1b1.stat.columbia.edu:8080) will verify that all tests pass (this may take hours).

4. Once code review is complete, one of the admins will push to `master`.










- We need to explain how things are labeled and milestoned in the issue tracker.
- Explain how to review a pull request.
- How to push the patch.