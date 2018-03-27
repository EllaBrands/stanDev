### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Bob Carpenter.  Should we include a Jacobian option for optimization?  Right now, we can't get max posterior mode (aka MAP) for the unconstrained model, just the constrained one.  Yea/nay.

Bob Carpenter.  Signature for `map_rect` includes `Eigen::Matrix` types or not (just `std::vector`)?   Yea/nay.

Sebastian Weber. Has anyone looked into thread-specific globals ([see here](http://en.cppreference.com/w/cpp/language/storage_duration))? This *could* open the door for threaded AD without any locking if I am not totally off!

Sean Talts. Different goals and use-cases for performance, regression, and end-to-end numerical accuracy testing.

### Open Discussion Topics

_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Bob Carpenter.  Governance, specifically
1. whether we're OK with someone with a final say on each module of our code (mostly aligned with repos),
2. how we make someone an official code-reviewer/merger or if such a thing even makes sense beyond (1),
3. how we make high-level project decisions about things like IP and the rights to run StanCon (e.g., delegate to NumFOCUS?), decide on licensing for code (or leave it with (1) as we do now), and other IP like the logo (which is owned by NumFOCUS, but doesn't necessary need to be controlled by that leadership body), etc.
4. relation to NumFOCUS, which collects non-profit donations for us, helps us run StanCon, and which has its own leadership body for making decisions about 
5.  others I'm sure I'm forgetting now