Linux kernel compile benchmarks kcbench & kcbenchrate
=====================================================

Kcbench and kcbenchrate are simple benchmarks scripts that compile a Linux
kernel a few times in a row and measure the time it takes. See INSTALL.md
for the various ways to download and install them. For an ad hoc use of
kcbench this is all you need:

```
curl -O https://gitlab.com/knurd42/kcbench/-/raw/master/kcbench
bash kcbench
```

This will download and execute kcbench, which will do these things:

* download and extract a suitable Linux kernel sourcecode to ~/.cache/kcbench/
* create a temporary directory like `/tmp/kcbench.xxxx`
* create a Linux kernel configuration using `make O=/tmp/kcbench.xxxx defconfig`
* compile Linux using `make -j <number> O=/tmp/kcbench.xxxx vmlinux`; it will
  try various setting for `<number>`, as what works best depends on your
  machine.

In the end it will produce results that look like this:

```
[cttest@localhost ~]$ bash kcbench
Processor:           Intel(R) Core(TM) i7-8700K CPU @ 3.70GHz [12 CPUs]
Cpufreq; Memory:     Unknown; 15934 MByte RAM
Linux running:       5.6.0-0.rc2.git0.1.vanilla.knurd.2.fc31.x86_64
Compiler used:       gcc (GCC) 9.2.1 20190827 (Red Hat 9.2.1-1)
Linux compiled:      5.3.0 [/home/cttest/.cache/kcbench/linux-5.3/]
Config; Environment: defconfig; CCACHE_DISABLE="1"
Build command:       make vmlinux
Run 1 (-j 12):       92.55 seconds / 38.90 kernels/hour
Run 2 (-j 15):       91.91 seconds / 39.17 kernels/hour
Run 3 (-j 6):        113.66 seconds / 31.67 kernels/hour
Run 4 (-j 9):        101.32 seconds / 35.53 kernels/hour

```

Or this:

```
[cttest@localhost ~]$ bash kcbench
Processor:           AMD Ryzen Threadripper 3990X 64-Core Processor [128 CPUs]
Cpufreq; Memory:     Unknown; 15934 MByte RAM
Linux running:       5.6.0-0.rc2.git0.1.vanilla.knurd.2.fc31.x86_64
Compiler used:       gcc (GCC) 9.2.1 20190827 (Red Hat 9.2.1-1)
Linux compiled:      5.3.0 [/home/cttest/.cache/kcbench/linux-5.3/]
Config; Environment: defconfig; CCACHE_DISABLE="1"
Build command:       make vmlinux
Run 1 (-j 128):      26.16 seconds / 137.61 kernels/hour
Run 2 (-j 136):      26.19 seconds / 137.46 kernels/hour
Run 3 (-j 64):       21.45 seconds / 167.83 kernels/hour
Run 4 (-j 72):       22.68 seconds / 158.73 kernels/hour
```

These two examples already show: the optimal number of jobs used to get the
best results depends on the system, mainly its processor. For more details about
this see the
[man page for kcbench](https://gitlab.com/knurd42/kcbench/-/raw/master/kcbench.man1.md).
There you'll also find a strong recommendation to why you want to use the same
operating system (or at least the same compiler) on all machines you want to
compare.

Note, kcbench tries to compile one kernel really quick and will put a lot of
load on your machine's processor, which is called 'speed run'. Nevertheless,
sometimes all CPU cores except one will idle for a few seconds, as a few parts
of the Linux kernel build process are single-threaded (like linking 'vmlinux').
This is unwanted when you want to create as much system load as possible or try
to max out the system. In this situations you might want to use kcbenchrate,
which is installed in parallel to kcbench. If you downloaded kcbench directly
with curl for ad hoc use (as outlined above) you need to do this to run kcbench:

```
ln -s kcbench kcbenchrate
bash kcbenchrate
```

When running as kcbenchrate the script will perform a 'rate run', which starts
one worker per CPU core that compiles one kernel with one job. This should keep
all CPUs busy nearly all the time.

Note that a 'rate run' approach takes a lot longer to generate results and needs
a lot more storage space. For more details about these two approaches
see the man pages linked and the end of this file.

To outline the potential of kcbench briefly, here is what a 'kcbench --help'
will show:

```
Usage: kcbench [options]

Compile a Linux kernel and measure the time it takes.

Available options:
 -b, --bypass                 -- bypass cache fill run and measure immediately
 -d, --detailedresults        -- print more detailed results
 -i, --iterations <int>       -- number or iterations per job
 -j, --jobs <int> (*)         -- number of jobs to use ('make -j #')
 -m, --modconfig              -- build using 'allmodconfig vmlinux modules'
 -o, --outputdir <dir>        -- compile in <dir>/kcbench/ ('make O=#')
 -q, --quiet                  -- quiet
 -s, --src (<dir>|<version>)  -- take Linux sources from <dir>; if not found
                                 try ~/.cache/kcbench/linux-<version>/ and
                                 /usr/share/kcdata/linux-<version>/; if still
                                 not found download <version> automatically.
 -v, --verbose (*)            -- increase verboselevel

     --add-make-args <str>    -- pass <str> to make call ('make <str> vmlinux')
     --cc <exec>              -- use specified target compiler ('CC=#')
     --cross-compile <arch>   -- cross compile for <arch>
     --crosscomp-scheme <str> -- naming scheme for cross compiler
     --hostcc <exec>          -- use specified host compiler ('HOSTCC=#')
     --infinite               -- run endlessly
     --llvm                   -- sets 'LLVM=1' to use clang and LLVM tools
     --no-download            -- never download anything automatically
     --savefailedlogs <dir>   -- save log from failed compilations in <dir>

 -h, --help                   -- show this text
 -V, --version                -- output program version

(*) -- option can be passed multiple times

On this machine kcbench by default will use a Linux kernel 5.7 configured by
'make defconfig'. It first will compile this version using 'make -j 4 vmlinux'
for 2 times in a row; afterwards it will repeat this with 6, 2 jobs ('make
-j #') instead, to check if a different setting delivers better results
(see manpage for reasons why).

Note: defaults might change over time. Some of them also depend on your
machines configuration (like the number of CPU cores or the compiler being
used). Thus, hardcode these values when scripting kcbench.
```

Kcbench and kcbenchrate will tell you if you need to install any additional
tools or libraries to perform their duty. But you can also install anything
upfront if you like:

 * On Fedora you should be able to install everything required by runnning
 this command:

 ```
sudo dnf install /usr/bin/bc /usr/bin/bison /usr/bin/curl /usr/bin/flex /usr/bin/lscpu /usr/bin/make /usr/bin/time /usr/bin/perl /usr/bin/pkg-config /usr/bin/pkill /usr/include/openssl/pkcs7.h /usr/include/libelf.h /usr/bin/gcc /usr/bin/ld
```

 * On Ubuntu this should do the trick:

 ```
sudo apt install bc bison curl flex util-linux make time perl-base pkg-config procps libssl-dev libelf-dev gcc binutils
```

 * On openSUSE and SUSE Enterprise Linux this should do the trick:

 ```
sudo zypper install bc bison curl flex util-linux make time gawk perl-base pkgconf-pkg-config procps openssl-devel libelf-dev gcc binutils
```

For more details about kcbench and kcbenchrate and their command line options
see their man pages, which are shipped alongside and available online in
markdown:

* [Manpage for kcbench](https://gitlab.com/knurd42/kcbench/-/raw/master/kcbench.man1.md)
* [Manpage for kcbenchrate](https://gitlab.com/knurd42/kcbench/-/raw/master/kcbenchrate.man1.md)

Both mentioned a few caveats you should be aware of when comparing results
from different systems. The two most important:

 * The Linux kernel version downloaded and compiled depends on the compiler and
 its version.
 * Do not compare results from systems that are using different compilers (like
 gcc9 and gcc10).

Kcbench and kcbenchrate were started by Thorsten Leemhuis and are available
under the MIT license – a permissive free software license which puts only
very limited restriction on reuse.
