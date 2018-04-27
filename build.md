<!--
// Copyright 2017 The Fuchsia Authors. All rights reserved.
-->
# Building Fuchsia

There are some subtleties in the current Fuchsia build process
that deserve mention. Plus the instructions are scattered through
a couple of different files, it's nice to have a quick-start guide
in the first place I look. This is my cheat sheet. If it works for
you, awesome.

The basic steps are as follows.

**Checkout**

```
curl -s "https://fuchsia.googlesource.com/jiri/+/master/scripts/bootstrap_jiri?format=TEXT" | base64 --decode | bash -s fuchsia
cd fuchsia
layer=<layer> ; jiri import -name="${layer}" "manifest/${layer}" "https://fuchsia.googlesource.com/${layer}"
jiri update
```

These steps are slightly different than the official ones as I don't
like adding sandbox-specific stuff to $PATH.
Yeah, one might type .jiri_root/bin/jiri every time, but it's easy to
write a wrapper script. The official instructions suggest copying jiri
to /usr/local/bin and chmod 755. You can do that, but you have to remember
to keep it up to date, even jiri is changing as we go. It's easier to just
use the jiri in the sandbox (but not have to hack $PATH).

Here's my wrapper, which is in ~/bin/scripts, which is already in my $PATH.

```
#!/bin/sh

JIRI=./.jiri_root/bin/jiri

if [ ! -x "$JIRI" ]
then
  echo "Must be run from top of fuchsia tree." >&2
  exit 1
fi

$JIRI "$@"
```

To download a snapshot from a build bot:

- First go to the build status page: https://fuchsia-dashboard.appspot.com/
  and click on the build you want.
- Then click on the link at the top with the builder line.
  E.g., Builder topaz-x86_64-linux-debug Build 3a6a4fdb419e3910
- Then scroll down and click on, say, "200" to display more builds.
- Then scroll down and click on the link of the build you want.
  E.g., if you want to go back roughly a week in time, find a build from that time.
- The URL you want to pass to `jiri update` is the "jiri.snapshot" link
  under "upload jiri.snapshot".
- Then in a shell:

```
curl -s "https://fuchsia.googlesource.com/jiri/+/master/scripts/bootstrap_jiri?format=TEXT" | base64 --decode | bash -s fuchsia
cd fuchsia
.jiri_root/bin/jiri update $URL
```

**Ninja**

The next step is to run gn to take all the BUILD.gn files and generate
.ninja files. The author uses the "fx" set of utilities.
If you want to see what's going on underneath, pass -x to fx.

The author has a wrapper in $PATH the invoke fx of the current tree:

```
#!/bin/sh

FX=./scripts/fx

if [ ! -x "$FX" ]
then
  echo "Must be run from top of fuchsia tree." >&2
  exit 1
fi

exec $FX "$@"
```

```
fx set x64
```

For arm64:

```
fx set arm64
```

By default the build configuration is written to ./.config.
If you want to have multiple builds, you can specify a config file:

```
fx --config x64.cfg set x64
```

And then pass the same config file to every fx command.

**Build**

With that done we can now build the rest of Fuchsia.

```
fx full-build
```

Building specific packages can be done with, e.g.,

```
fx set x64 --packages p1,p2,...
```

**Workflow**

After that is done, when you need to update and rebuild:

```
jiri update -gc
fx full-build
```

Sometimes dependencies change such that it's best to wipe out
the previous build first:

```
jiri update -gc
fx clean-build x64
```

Working with jiri itself requires some discussion, for another day.

**Going Back In Time**

1) Go to the build status page.
2) Click on a link, say topaz-x86-debug(sp).
3) Click on "200" at the bottom.
4) Scroll down to find a build back in time that you want.
5) Click on the link of the build you want.
6) The "upload jiri.snapshot", "jiri.snapshot" link is what you
   want to pass to jiri update.
7) Create empty fuchsia dir.
8) Copy .jiri_root from some other fuchsia tree.
9) jiri update <link-from-step-6>

**References**

https://fuchsia.googlesource.com/docs/+/master/getting_started.md
https://fuchsia.googlesource.com/docs/+/master/README.md

... and follow the various links referenced there.
