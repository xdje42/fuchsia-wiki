<!--
// Copyright 2017 The Fuchsia Authors. All rights reserved.
-->
# Building Fuchsia

There are some subtleties in the current Fuchsia build process
that deserve mention. Plus the instructions are scattered through
a couple of different files, it's nice to have a quick-start guide
in the first place I look.

The basic steps are as follows.
The default target is amd64. Instructions for arm64 may get added,
or you can just look for them in the Fuchsia docs.

**Checkout**

```
curl -s https://raw.githubusercontent.com/fuchsia-mirror/jiri/master/scripts/bootstrap_jiri | bash -s fuchsia
cd fuchsia
.jiri_root/bin/jiri import fuchsia https://fuchsia.googlesource.com/manifest
.jiri_root/bin/jiri update
```

These steps are slightly different than the official ones as I don't
like adding sandbox-specific stuff to $PATH.
Yeah, one might type .jiri_root/bin/jiri every time, but it's easy to
write a wrapper script. The official instructions suggest copying jiri
to /usr/local/bin and chmod 755. You can do that, but you have to remember
to keep it up to date, even jiri is changing as we go. It's easier to just
use the jiri in the sandbox (but not have to hack $PATH).

**Sysroot**

The next step is building sysroot. "sysroot" is, essentially, what one thinks
of as /usr/include and /usr/lib of a base system (e.g., libc, libc++, and
misc base libraries/headers).
The name "sysroot" in "build-sysroot.sh" is a bit of a misnomer as this is
also where magenta, the kernel, is built.

```
./scripts/build-sysroot.sh
```

WARNING: Don't forget to rerun this step after you do a full source update.
This compiles, among other things, libc and various magenta libraries and puts
them in out/sysroot. If you forget this step, and something like a new syscall
is added or changed, then if you're lucky the build will fail. If not then
things may not work in inexplicable ways (because the stale libraries in
out/sysroot are, for example, using the wrong syscall numbers).
ABI stability is not a priority yet.

**Ninja**

The next step is to run gn to take all the BUILD.gn files and generate
.ninja files.

```
./packages/gn/gen.py
```

**Build**

With that done we can now build the rest of Fuchsia.

```
./buildtools/ninja -C out/debug-x86-64
```

One can make an alias for that or write a wrapper script or whatever.

**Workflow**

After that is done, when you need to update and rebuild:

```
./.jiri_root/bin/jiri update
./scripts/build-sysroot.sh
./buildtools/ninja -C out/debug-x86-64
```

Working with jiri itself requires some discussion, for another day.

WARNING: Rebuilding sysroot can often trigger a full rebuild due to
how dependencies are managed. There are often ways to avoid having to
rebuild sysroot. A topic for another day.

**References**

https://fuchsia.googlesource.com/docs/+/master/getting_started.md
https://fuchsia.googlesource.com/docs/+/master/README.md

... and follow the various links referenced there.
