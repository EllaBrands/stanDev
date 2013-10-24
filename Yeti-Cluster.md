This is instructions for users from the Dept. of Statistics for the new (as of Oct 2013) Torque-based Linux cluster at Columbia.

1. Online user documentation can be found here:

 https://wikis.cuit.columbia.edu/confluence/display/rcs/Yeti+HPC+Cluster+User+Documentation

2. All questions regarding use of the cluster can be sent to the usual
address: hpc-support@columbia.edu.  As we are now supporting both
Hotfoot and Yeti, it will make things easier for us if you mention
which of the two clusters your question involves.

3. Yeti can be accessed by making an ssh connection to ```yetisubmit.cc.columbia.edu```

4. Your Yeti group for submitting jobs is ```yetistats```

5. Scratch storage is located at ```/vega/stats```

6. In order to make transition easier, Hotfoot scratch directories
have been mounted on ```/hotfoot``` on the Yeti head nodes.  This means, for
example, that if you work on ```/hpc/stats/users/rl2226``` on Hotfoot you'll
find your files at ```/hotfoot/stats/users/rl2226``` on Yeti.  Note that
```/hotfoot``` is being made available for file transfer only - your jobs
will not have access to these directories.

Questions or comments can be sent to hpc-support@columbia.edu.
