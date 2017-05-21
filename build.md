<!--
// Copyright 2017 The Fuchsia Authors. All rights reserved.
-->
# Building Fuchsia

There are some subtleties in the current Fuchsia build process
that deserve mention. Plus the instructions are scattered through
a couple of different files, it's nice to have a quick-start guide
in the first place I look.

The basic steps are as follows.
The default target is amd64. For building for arm64, see below.

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

**Kernel+Sysroot**

The next step is building the kernel (magenta) + sysroot.
"sysroot" is, essentially, what one thinks of as /usr/include and /usr/lib
of a base system (e.g., libc, libc++, and misc base libraries/headers).

```
./scripts/build-magenta.sh
```

For arm64:

```
./scripts/build-magenta.sh -t aarch64
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

For arm64:

```
./packages/gn/gen.py --target_cpu=aarch64
```

**Build**

With that done we can now build the rest of Fuchsia.

```
./buildtools/ninja -C out/debug-x86-64
```

For arm64:

```
./buildtools/ninja -C out/debug-aarch64
```

One can make an alias for that or write a wrapper script or whatever.

Building specific modules can be done with, e.g.,

```
./packages/gn/gen.py --target_cpu x86-64 --modules fortune,modular -o out/your_output
```

**Workflow**

After that is done, when you need to update and rebuild:

```
./.jiri_root/bin/jiri update
./scripts/build-magenta.sh
./buildtools/ninja -C out/debug-x86-64
```

Sometimes changes are made that the dependencies don't/can't properly
track. If a build fails, "rm -rf out" and try again.

Working with jiri itself requires some discussion, for another day.

WARNING: Rebuilding sysroot can often trigger a full rebuild due to
how dependencies are managed. There are often ways to avoid having to
rebuild sysroot. A topic for another day, though as Fuchsia matures
the need for such hacks will go down.

**References**

https://fuchsia.googlesource.com/docs/+/master/getting_started.md
https://fuchsia.googlesource.com/docs/+/master/README.md

... and follow the various links referenced there.
