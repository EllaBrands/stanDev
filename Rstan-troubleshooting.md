This is a list of common problems and their solutions.

### Problem:  Stan used to run on my computer but it suddenly stopped running.

* Possible solution:  Your computer may have updated Xcode.  You may need to launch Xcode once to accept its license.  This can be done in the Terminal window by typing "sudo xcodebuild - license" and following the instructions from there.

### Problem: Errors regarding xcodebuild or xcode 

* Possible solutions: Install xcode via app-store, or do `xcode-select --install` in the R console

### Problem: Rtools is not working

* Possible solution: Make sure you have the right Rtools, (32 or 64bit) for your system. 

### Problem: I can't install Rtoolchain 

* Possible solution: If on macOS, find the `.pkg` in `/Library/Developer/CommandLineTools/Packages/` and double-clicking on that to install the C++ headers to /usr/include, then in `file.edit("~/.R/Makevars")`delete the part of CXX14FLAGS where it says `-isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include`.

### Problem: Makevars does not exist or other Makevars problems. 

* Possible solution: try `file.remove("~/.R/Makevars.win")` and then install stan headers 
 and rstan binary, e.g. on windows: `install.packages("https://win-builder.r-project.org/9lJAt18Tb6mf/StanHeaders_2.18.1-10.zip", repos = NULL)` and `install.packages("https://win-builder.r-project.org/tJ1528Lvo0H3/rstan_2.18.2.zip", repos = NULL)`.



 