## Testing framework

### Current state of non-probability function math unit tests
In order to cut down on the inconsistencies in stan/math non-probability unit tests and the menial labor that goes into writing the unit tests, we need to design a unified testing framework.

Right now we leave unit test writing to the developer. The developer must manually instantiate all combinations of admissible types for each function written. Next, it is up to the developer to write input tests, Nan tests, valid values, etc.

### Current state of probability function math unit tests

While we do now have a framework for probability tests, the current code isn't testing second and third derivatives as was intended.

