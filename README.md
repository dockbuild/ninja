ninja-jobserver
===============

This **ONLY** purpose of this project is to distribute a version
of [ninja](https://ninja-build.org/) with GNU make jobserver client support.

### motivation

GNU make jobserver client support was implemented and contributed in 2016 by
[@stefanb2](https://github.com/stefanb2) but the corresponding pull
request has not yet been integrated into the upstream project.

Based on [ninja/issues/1139#issuecomment-215087721](https://github.com/ninja-build/ninja/issues/1139#issuecomment-215087721):

> Ninja works best if it knows about the whole build.

### the issue

_Text below copied from the issue [ninja-build/ninja#1139](https://github.com/ninja-build/ninja/issues/1139) description_

>
> Title: Add GNU make jobserver client support.
>
> As long as ninja is the only build execution tool, the current ninja -jN implementation works fine.
>
> But when you try to convert parts of an existing recursive GNU make based SW build system to ninja, then you have the following situation:
>
>    top-level GNU Make (with -jX, acts as job server)
>    M instances of GNU make (with -j, act as job server clients)
>    N instances of ninja (don't know anything about job server)
>
> Simply calling `ninja -jY' isn't enough, because then the ninja instances will try to run Y*N jobs, plus the X jobs from the GNU make instances, causing the build host to overload. Relying on -lZ to fix this issue is sub-optimal, because load average is sometimes too slow to reflect the actual situation on the build host.

> It would be nice if GNU make jobserver client support could be added to Ninja. Then the N ninja instances would cooperate with the M GNU make instances and on the build host only X jobs would be executed at one time.

### the pull-request

_Text below copied from the pull request [ninja-build/ninja#1140](https://github.com/ninja-build/ninja/issues/1140) description_

>
> Title: Add GNU make jobserver client support.
>
> * add new `TokenPool` interface
> * GNU make implementation for `TokenPool` parses and verifies the magic
>   information from the `MAKEFLAGS` environment variable
> * `RealCommandRunner` tries to acquire `TokenPool`
>   * if no token pool is available then there is no change in behaviour
> * When a token pool is available then `RealCommandRunner` behaviour
>   changes as follows
>   * `CanRunMore()` only returns true if `TokenPool::Acquire()` returns true
>   * `StartCommand()` calls `TokenPool::Reserve()`
>   * `WaitForCommand()` calls `TokenPool::Release()`
>
> Documentation for GNU make jobserver
>
> http://make.mad-scientist.net/papers/jobserver-implementation/
>

### generating the binaries

First, create the release tag:

```
git tag -s -m "v1.8.2-jobserver" v1.8.2-jobserver  topic-issue-1139-squashed
```

Then, build the packages and upload them as release assets:

* Linux:

```
mkdir /tmp/scratch && cd $_
git clone git://github.com/dockbuild/ninja-jobserver -b topic-issue-1139-squashed
cd ninja-jobserver

docker run --rm dockbuild/centos5-devtoolset2-gcc4 > ./dockbuild-centos5-devtoolset2-gcc4
chmod +x ./dockbuild-centos5-devtoolset2-gcc4

./dockbuild-centos5-devtoolset2-gcc4 ./configure.py --bootstrap
./dockbuild-centos5-devtoolset2-gcc4 strip ninja

zip ninja-jobserver-linux.zip ninja
```

* macOS:

*To be done*

* Windows:

*To be done*



### acknowledments

Thanks you to the ninja-build developers for making such an awesome tool. See [list of contributors](https://github.com/ninja-build/ninja/graphs/contributors).
