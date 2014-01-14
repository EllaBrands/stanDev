For developers of CmdStan in Windows, you will need to install make 3.81 or later. This does not apply to users of CmdStan in Windows.

Steps:

0. **Prerequisite** Install Rtools. See user installation directions. Make sure all tools are on the path correctly.

1. Download and install make 3.81 (or higher). It is currently located at [GNU Make 3.81](http://gnuwin32.sourceforge.net/packages/make.htm). I was able to install using the setup file.

2. Edit the PATH environment variable so the installation location of make appears before RTools.

3. Remove / move Window's find.exe from C:\Windows\system32. This seems to be a bug in this version of make. Even though it should pick out find.exe from Rtools, it seems to pick out C:\Windows\system32 first. I moved mine to C:\.


If these directions are unclear, please write to stan-users@googlegroups.com asking for clarification.
