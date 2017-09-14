### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Carpenter.  Should everyting we distribute on the web site have an open license?  Decision.

Betancourt.  Jumping Rivers Proposal.  Opinions.
### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Carpenter.  What should we do about PyStan licensing?  It sounds like Allen wants to keep it copylefted to prevent for-profits from using it without sharing their code.  I'd rather it be BSD-ed.  I'd be in favor of allocating project resources to create a BSD-ed Python interface.  Something lightweight around CmdStan could also avoid all the installation headaches that seem to be endemic to linking through C++.  There's no mechanism to force Allen to more liberally license PyStan because he or his employer owns the copyright.  I've talked to Allen about this, and I don't see an alternative.  Already, many people are building these lightweight wrappers anyway in industry (I know of at least two documented cases).  A related issue if we fork is what to do with PyStan as it stands.

Riddell. Should PyStan adopt a partial copyleft license ([MPL2](https://en.wikipedia.org/wiki/Mozilla_Public_License), also used by Eigen) or rather use a permissive [BSD](https://en.wikipedia.org/wiki/BSD_licenses) license? My preference is for the partial copyleft license. Copyleft licenses are a clever hack. They and their [creator](https://en.wikipedia.org/wiki/Free_Software_Foundation) deserve credit for spreading free and open source software into every corner of computing. Projects using copyleft licenses find it easier to recruit volunteer developers because they offer developers the assurance that the source code files they labor on will, if improved and repackaged in a commercial product, be available for inspection. Moreover, in my experience, the communities that grow up around copyleft software tend to be more attractive sites for public-spirited collaboration. For example, in these communities you tend to find developers and organizations who believe the free market should serve democratic aspirations rather than subvert them or be indifferent to them. If a majority of Stan developers actively prefers one of these licenses over the other, I'd support choosing that one.  Given how heated discussions about this topic can become, I would not ask for preferences to be expressed in public; an anonymous ballot would make sense.

Carpenter.  Web page governance?  We have no way to decide what should go on the web pages as nobody's in charge of them.  I built the first version of the pages.  Michael launched the second version, but I substantially modified the organization over time as new things got added.  Then Michael completely rebuilt the pages recently to organize around "about" and "users" and "developers" at the top level rather than "docs" and "interfaces", etc. at the top level.   Andrew has different ideas of how they should be organized (I think he wants only two tabs---so I wanted seven or eight, Michael used about half that, and Andrew wants to cut it in half again).  Given that we have roughly zero expertise in web design and measurements, it didn't seem like arguments would amount to anything other than "I think it'll be better this way."

Gelman.  Stan books.  I talked with our editor at CRC about the Bayes Econ book (with Jim Savage and others) and the Stan book.  We should figure out how difficult it would be to put together the Stan book:  maybe we already have 90% from existing materials?  It would be good to be able to do this without it feeling like "work."

Gelman.  Blue-sky ideas for Keck grant:  https://research.columbia.edu/content/2018-keck-foundation-research-grants

Gelman:  Datacamp and Coursera (again).

Talts.  Code style.  We need something more precise to make code reviewe easier.

Gelman.  Demos/movies