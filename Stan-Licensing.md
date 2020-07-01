# Current Stan licenses and dependencies

- Stan math library (stan-dev/math)
    * Does not depend on any other Stan subprojects
    * Is currently licensed under BSD 3-clause
    * Has external dependencies:
    * Boost, most of it is covered under the Boost Software License https://www.boost.org/users/license.html; we have made modifications to the source
    * Eigen, licensed under MPL2 http://eigen.tuxfamily.org/index.php?title=Main_Page#License; we have made modifications to the source
    * Sundials, licensed under BSD 3-clause https://computing.llnl.gov/projects/sundials/license; we have made modifications to the source
    * OpenCL, licensed under MIT https://github.com/KhronosGroup/OpenCL-Headers/blob/master/LICENSE (for testing only, Google Test, licensed under BSD 3-clause https://github.com/google/googletest/blob/master/LICENSE)
    * CppLint  (testing only): https://github.com/google/styleguide/tree/gh-pages/cpplint
    * Google Test (testing only): BSD 3-page license https://github.com/google/googletest/blob/master/googletest/LICENSE
- Stan library (stan-dev/stan)
    * Depends on stan-dev/math
    * Is currently licensed under BSD 3-clause
    * Has no additional external dependencies

- CmdStan
    * Depends on Stan
    * Is currently licensed under BSD 3-clause
    * Has no additional external dependencies

- RStan
    * Depends on Stan
    * Is currently licensed under GPLv3
    * Has external dependencies: https://github.com/stan-dev/rstan/blob/develop/rstan/rstan/DESCRIPTION
        * R (>= 3.4.0), looks like mostly GPL-2, some GPL-3, some LGPL-2.1 https://www.r-project.org/Licenses/
        * StanHeaders (> 2.18.1), licensed under BSD 3-clause (controlled by RStan) https://cran.r-project.org/web/packages/StanHeaders/
        * ggplot2 (>= 2.0.0), licensed under GPL-2 https://cran.r-project.org/web/packages/ggplot2/
    * Links against: https://github.com/stan-dev/rstan/blob/develop/rstan/rstan/DESCRIPTION
        * Rcpp (>= 0.12.0), GPL-2, GPL-3 https://cran.r-project.org/web/packages/Rcpp/
        * RcppEigen (>= 0.3.3.3.0), GPL-2, GPL-3 https://cran.r-project.org/web/packages/RcppEigen/
        * BH (>= 1.69.0), BSL-1.0 https://cran.r-project.org/web/packages/BH/index.html
        * StanHeaders (> 2.18.1), BSD 3-clause https://cran.r-project.org/web/packages/StanHeaders/


# Ramifications of Apache 2.0 dependency

**Update** On September 15th, with no objections, we became able to include Apache 2.0 licensed dependencies. Of course, any possible dependencies will need to be [evaluated](https://github.com/stan-dev/math/wiki/Checklist-for-adding-a-dependency-to-the-Math-library).

We were evaluating whether to use Intel's TBB with our Math library, which is Apache 2.0 licensed, and ended up getting connected with a NumFOCUS lawyer who specializes in these things and asking a lot of questions about Apache 2.0 mixing with Stan's BSD specifically. Here is a complete dump of the Q&A.

## Questions and Answers from NumFOCUS

(the original questions went far beyond the main scope of including the Intel TBB with the Apache 2.0 license so that we focus on these first, but also address concerns around downstream licenses secondary)

These are the answers from NumFOCUS in Q&A format (names have been taken out of this):

A few responses to the questions below and what I have observed on the associated issue tracker. As I said, it should be fine to use the Apache dependency and as a practical matter it won't change much.  I'll do this in Q&A format.

1. Q: **What is the main difference between the Apache (v2) license and the BSD license?**  
  A: Both the Apache and BSD licenses are "permissive" licenses - you can do almost anything you want with the code, including use it in proprietary applications. The primary substantive differences are that the Apache license includes both a clause that terminates any patent grants if there is litigation against a competitor, and that it explicitly disclaims any trademark license. These terms are unlikely to cause you any issues.

2.  Q: **If we use an Apache-licensed dependency, does that mean our code has to be Apache-licensed?**.  
   A: No. The Apache license is a permissive license, and you can combine it with other code that has other licenses. That means that you can distribute code that is BSD-licensed that bundles with it Apache-licensed code. You just have to make sure that your documentation and notices that you provide a) note the Apache-licensed dependency, and b) also provide a copy of the Apache license.

3. Q: **Why do we need to provide a copy of the Apache License? Our code is under BSD.**  
   A: Yes, *your* code is under BSD. However, you don't have the rights to change the license on the dependency, and you will still need to comply with the Apache License. Thankfully, that is easy to do. Just provide attribution and a copy of the license with the code.

4. Q: **We may not bundle the code - it is only a dependency.**  
   A: Even better! Then just note the dependency, and that it is Apache licensed, in your docs. That's all you need to do (and even that is probably even more than the actual minimum).

5. Q: **What about the GPL?**   
   A: This is the "wrinkle" I mentioned. If somebody wants to take your code (under BSD), and the Apache dependency, *and* GPLv2-licensed code, and bundle them all together, then that person will not be able to distribute that particular mix of code together. Apache v2 and GPL v2 are incompatible, so you can't bundle those two in the same application due to the conflicting requirements. However, that restriction is for GPLv2 only; the GPLv3 is compatible with Apache.

6. Q: **But we don't want to put anything under the GPL, we just want to stay GPL compatible.**  
   A: You won't have to put anything under the GPL, and you will be GPLv3 compatible. Just not GPLv2. *This is the primary side-effect of including the Apache v2 dependency - and it will only apply to a theoretical user of your code that also wants to use GPLv2-licensed code to create an application. This means that your code will have a slightly smaller theoretical scope of use, but it isn't really your problem.*

7. Q: **We have trademarks on the Stan name and logo.  Will that be an issue? We absolutely do not want to disclaim these trademarks or be prevented from enforcing them.**   
   A: There will be no effect on the Stan name and logo. (The Apache license is actually slightly more protective of your marks than the BSD license that you currently use.)

8. Q: **What about ggplot2, the main graphics package in R, which is licensed under the GPLv2? Will this cause an issue with our R packages that include Stan code and ggplot2 code, like RStan and all its derivatives like rstanarm and brms?**  
  A: This depends on the structure of your packages. Specifically, a person cannot distribute a single work that includes both Apache2 and GPL2 code.

There is a lot in that - specifically note the "single work" restriction.

9. Q: **Can you create a set of packages that work together, and one of them is Apache licensed and one is GPLv2 licensed?**  
   A: Yes.

10. Q: **Can users bring code from the two together for their personal use?**  
    A: Yes. You just can't link together the Apache-licensed and GPLv2-licensed libraries into a unitary binary and distribute that unitary binary.

I am not sure how significant this restriction is for your particular use case.

11. Q: **Will there be any problem with RStan? It is licensed under the GPL (>= 3). It links against Stan's C++ code, which is where the Apache module would be.  Will that be a problem?**.  
    A: *No, there is no incompatibility with GPL >=3. You can continue to do this.*


**Additional answers added 7th September 2019**

12. Q: **It would help to know what terms like "link" and "unitary" and "binary" 
mean in this context. Does linking include the technical C++ notions of static and dynamic linking or is it another usage?**  
    A: It means at least the C/C++ notions of static and dynamic linking. It may also mean more – that is the “unitary” part.  
    A quick digression: When I am talking about a “unitary” binary, I am not using a specific legal term of art. What I am trying to convey is the key question is whether various components should be considered “the same work” for copyright purposes. So you ask - what is the bright line rule?  
    There isn’t one.   
    Most people agree that a process boundary is probably sufficient to separate two works in a copyright sense - although some could make an argument that particularly tight coupling between two binaries was sufficient to make them “one work” even if they were technically in different processes.  
    Another thing to look at – and in my opinion, probably the most important – is how the works are referred to. If you had a program with a bunch of subsidiary libraries, but you referred to them all as “the program” and only ran “the program,” then it makes it much more likely that a judge would find them to be one work.  
    If, on the other hand, you referred to “the first program” and separately to “the other program,” and you ran them separately, or downloaded them separately, and they could be run independently, then they would be more likely to be different works with different copyrights (and thus different licenses).
 
13. Q: **How does this apply to the GPL?**.  
    A: The GPL applies to anything that is part of the “same program” as a GPL’d work, or part of the “same program” as something that is based off of a GPL’d work (technically, a “derivative work”). Thus, when I was talking about a “unitary” program, I was trying to holistically convey a sense that you can’t bind Apache v2-licensed and GPLv2-licensed parts into the “same program” and then distribute that combined program. A looser coupling, across various binaries and libraries, brought together by a user – that is most likely OK. But until some of these questions are litigated, it is really about how you reason about the connections, how you describe them, and – crucially – your risk tolerance.   
     Thus, when looking at TBB and Stan, you need to evaluate how closely tied TBB would be to ggplot2 and other GPLv2 elements. If they are loosely and transitively coupled, that is good reason to think that they are separate works and the GPLv2-ness of ggplot2 doesn’t “leak” across the various abstractions to TBB.
 
14. Q: **That is all confusing. What’s the TL;DR version of whether the GPL applies when you combine two works?**.  
   A: The copyright relationship between two linked works is fact-specific, but you should look at:  
     - How closely tied are the two programs together?  Programs that are closely tied together are more likely to be considered derivative works. Introducing substantial new functionality into an existing copyrighted work is more likely to be considered a derivative work.
     - Are the programs advertised as being a single work?  Programs that are described as being a single work are more likely to be considered single works, regardless of whether they are organized into one solitary binary, or have different separate parts.
     - Are the programs downloaded together or made exclusively to work together?  If the GPL-licensed and non-GPL-licensed programs come from different people or in different packages, it seems unlikely that dynamically bringing the two programs together would implicate the GPL.

15. Q: **Is a Docker container with binary Apache2 and GPLv2 components included considered a unitary binary and is it considered linked?   How about an AWS container?**.  
    A: I’ll take “Hard questions with no clear answers” for $100, Alex.  
    The short answer is that it is *probably* doable, but it depends on your risk tolerance and how you discuss the container.  
    The key question is whether the software in a Docker container is best considered to be a collection of independent works that happen to be bundled together - like a zip file – or whether a Docker container is a statically or dynamically linked “binary.”  
    Interpreting a Docker container as a collection suggests that all elements in the Docker image are independent in a copyright sense, and so the different elements do not implicate cross-component GPL licensing concerns. Alternatively, if a Docker container is a linked work, then the co-location of the different code into the single image indicates that the final Dockerized application is a single work, and a  “static linking”-style analysis should apply.  
    As techies, we tend to favor the “collection of independent works” interpretation most of the time, because we see what is inside. But even we fall for “binary”-type language when discussing containers. For example, according to Docker, Inc., a container is:
     ...A standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.  
    Similarly, Amazon Web Services docs say:  
   Containers provide a standard way to package your application's code, configurations, and dependencies into a single object. Containers share an operating system installed on the server and run as resource-isolated processes, ensuring quick, reliable, and consistent deployments, regardless of environment. The AWS Cloud offers infrastructure resources optimized for running containers, as well as a set of orchestration services that make it easy for you to build and run containerized applications in production.    
    On the other hand, creating a docker file that “bundles together the applications and libraries you need to run STAN” sounds like a collection of independent works.   
   As for Amazon containers – I presume you mean AMIs – that is more easily analogized to a disk image, making it more collection-like than some Docker use cases.
 
15. Q: **Can we distribute a script that downloads Apache2 licensed code and downloads GPLv2 code from the same site, then scripts their static linking into an executable?**
    A: Technically yes; it is the “user” that is doing the linking, and a combined “unitary” binary is never distributed. The GPL does not apply to binaries that are not distributed as a single program. Further, the GPL places no restrictions on the end user’s ability to combine the GPL-licensed and non-GPL-licensed programs into a single binary at runtime.  
    Full disclosure, though - there are a number of people who consider this workaround questionable.
 
16. Q: **So what should we do?**.  
    A:
    1. Evaluate your usage. From the back-and-forth, it seems most likely that your proposed usage is more like a “collection” of things that work together, not a single monolithic binary. This is true particularly for ggplot2 and TBB, which are both clearly independent right now.  
    2. For any places where you would need to create a binary, distribute a script that has the *user* create the binary, consistent with other examples on CRAN.  
    3. Publish your usage, highlighting the separateness if possible: “We may bundle a set of applications and libraries into a single image or file for user convenience when downloading and installing the various packages. However, the different parts of the package are separate and can be extracted and used independently. There are also directions (at [LINK]) showing how to use system-provided libraries to provide the same functionality.

