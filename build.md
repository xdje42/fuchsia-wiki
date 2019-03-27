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
curl -s "https://fuchsia.googlesource.com/fuchsia/+/master/scripts/bootstrap?format=TEXT" | base64 --decode | bash
```

Here's my wrapper, which is in ~/bin/scripts, which is already in my $PATH.

```
#! /bin/sh

RELJIRI=./.jiri_root/bin/jiri

if [ -x "$RELJIRI" ]
then
  JIRI="$RELJIRI"
elif [ -x "fuchsia/$RELJIRI" ]
then
  cd fuchsia
  JIRI="$RELJIRI"
else
  echo "Must be run from top of fuchsia tree." >&2
  exit 1
fi

$JIRI "$@"
```

To download a snapshot from a build bot:
[TODO: these instructions need updating]

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

And if that doesn't work, copy the contents of $URL to .jiri_manifest.

**Ninja**

The next step is to run gn to take all the BUILD.gn files and generate
.ninja files. The author uses the "fx" set of utilities.
If you want to see what's going on underneath, pass -x to fx.

The author has a wrapper in $PATH the invoke fx of the current tree:

```
#! /bin/sh

RELFX=./scripts/fx

if [ -x "$RELFX" ]
then
  FX="$RELFX"
elif [ -x "fuchsia/$RELFX" ]
then
  if false
  then
    FX="fuchsia/$RELFX"
    # cd fuchsia
  else
    FX="$RELFX"
    cd fuchsia
  fi
else
  echo "Must be run from top of fuchsia tree." >&2
  exit 1
fi

exec $FX "$@"
```

The argument to "fx set" is PRODUCT.BOARD.
To get list of possibilities:
```
fx list-products
fx list-boards
```

This gets a basic system working:

```
fx set terminal.x64
```

For arm64:

```
fx set terminal.arm64
```

If you want to have multiple builds, you can specify a build directory:

```
fx set terminal.x64 --build-dir out/x64
```

And then pass the same directory to every fx command, with --dir <DIR>.

**Build**

With that done we can now build the rest of Fuchsia.

```
fx full-build
```

Building specific packages can be done with, e.g.,

```
fx set terminal.x64 --with gn-rule1,gn-rule2,...
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
fx clean-build terminal.x64
```

Working with jiri itself requires some discussion, for another day.
[TODO: These instructions need vetting.]

**Going Back In Time**

1) Go to the build status page.
2) Click on a link, say topaz-x86-debug(sp).
3) Click on "200" at the bottom.
4) Scroll down to find a build back in time that you want.
5) Click on the link of the build you want.
6) The "upload jiri.snapshot", "jiri.snapshot" link is what you
   want to pass to jiri update.
   Note: A usable link on 18/09/13 was the _snapshot_contents_ link.
7) Create empty fuchsia dir.
8) Copy .jiri_root from some other fuchsia tree.
9) jiri update <link-from-step-6>

**References**

https://fuchsia.googlesource.com/fuchsia/+/master/docs/development/source_code/README.md
https://fuchsia.googlesource.com/fuchsia/+/master/docs/getting_started.md
https://fuchsia.googlesource.com/fuchsia/+/master/docs/README.md

... and follow the various links referenced there.
