### Previous releases of Command-line Stan

All releases of Stan are available from
<ul>
<li>
<a href="https://github.com/stan-dev/stan/releases">Stan Releases</a>
</li>
</ul>

Two versions of the Stan source can coexist peacefully in separate directories.  Nothing in the Stan build affects anything outside of its own directory structure.  

Full instructions are part of the guide to
<ul>
<li><a href="http://mc-stan.org/start-command-line.html">Getting Started with Command-Line Stan</a>
</ul>

### Previous releases of RStan

All releases of RStan back to 1.3 are available from

<ul>
<li><a href="https://github.com/stan-dev/rstan/releases">All RStan Releases</a></li>
</ul>

#### Updating, Downdating, or Reinstalling RStan

This approach will work to (a) update the version of Stan to a newer version, (b) downdate the version of Stan to an older version, or (c) reinstall the same version.

To update, downdate, or reinstall RStan, you **MUST**:

1.   uninstall current version of RStan
2.   quit R
3.   restart R
4.   install new version of RStan
5.   quit R
6.   restart R

Full instructions are part of the guide to
<ul>
<li><a href="https://github.com/stan-dev/rstan/wiki/RStan-Getting-Started">Getting Started with RStan</a></li>
</ul>

#### Running Multiple Versions of RStan

The easiest way to do it is to install 1.3 to a different directory. By default, it will install to the first directory given by

```
.libPaths()
```

which looks like

```
[1] "/opt/site-library"             "/usr/local/lib/R/site-library" "/usr/lib/R/site-library"      
[4] "/usr/lib/R/library"          
```

Then

```
install_url('https://github.com/stan-dev/rstan/releases/download/v1.3.0/rstan_1.3.0.tar.gz', args = paste0("--library=", .libPaths()[2]))
Downloading rstan_1.3.0.tar.gz from https://github.com/stan-dev/rstan/releases/download/v1.3.0/rstan_1.3.0.tar.gz
Installing package from /tmp/RtmpJLwfA3/rstan_1.3.0.tar.gz
Installing rstan
```

You need to make sure that you have write permissions for the folder you are trying to install to. Then to use, for example, v1.3.0 instead of v2.0.0, you can do

```
library(rstan, lib.loc = .libPaths()[2])
```

or wherever you installed v1.3.0. It won't work to have two versions of the ```rstan``` package loaded at the same time.  It will not work in Windows to ```detach()``` one version and load the other in the same R session --- you have to restart R.