---
title: Advanced
id: version-0.14.1-install-advanced
original_id: install-advanced
---

Before diving into how Almond starts, a few words about the Almond installation.
The installation of Almond can look complex by some aspects. Beyond that, customizing
the installation, making sense of what it does in detail, can be tricky.
This originates from two concerns that Almond tries to reconcile:
- isolation of the internal dependencies of Almond from users
- allowing the customization / tweaking of the Almond class path upon installation

## Isolation of Almond internal dependencies

Almond depends on fs2 and cats-effect. In order not to force its own cats / cats-effect /
fs2 versions upon users, these internal dependencies of Almond are effectively hidden
from users.

In more detail, isolation in Almond works this way: the
[`scala-kernel-api` module](https://repo1.maven.org/maven2/sh/almond/scala-kernel-api_2.13.16)
and its dependencies are user-facing. The rest of the Almond class path
(the [`scala-kernel` module](https://repo1.maven.org/maven2/sh/almond/scala-kernel_2.13.16)
and its dependencies, but for everything already pulled by `scala-kernel-api`) are internal
dependencies, hidden from users.

Isolation between these two set of dependencies is achieved by starting Almond using
[launchers generated by coursier](https://get-coursier.io/docs/cli-bootstrap). (Note that
the coursier documentation doesn't really detail how dependency isolation works. We detail
that right below here.)

If we put dependencies isolation aside, these launchers are a small Java application, alongside a
resource file listing URLs of the JARs that should be loaded. Upon startup, the launcher reads
the URL list, check if these are in the coursier cache and downloads them if they're not. Then it
creates a `ClassLoader`, loads the local copies of the JARs in it, and loads and calls the main
class of the application from it.

With dependencies isolation enabled, these launchers actually embed two lists of JARs: a "top"
one (corresponding to user dependencies for Almond), and a "bottom" one (for Almond, internal
dependencies, with cats-effect, fs2, etc.). Upon startup, they create a first class loader
with the top dependencies, and
a second one, having the first class loader as a parent, with the bottom dependencies. In such
cases, the way class loaders work on the JVM makes the classes loaded in the bottom class loader
"see" the classes in the top one, but not the other way around. Later on,
when the app runs, it can ask for the class loader of a class it knows is part of the top
dependencies, which gives it a reference to the top class loader, that knows nothing about the
bottom dependencies.

Almond relies on that mechanism to get a class loader that only knows about user-facing
dependencies. That class loader is used as a parent of the class loader that's going to
load the classes generated during the session (those corresponding to the input user
code in the notebook). That way, users only "see" user-facing dependencies, not internal
ones, and they can load whichever other version of the same internal dependencies as Almond.

## About the installation process

When passed the `--install` option, Almond tries to determine the path of its own launcher,
and copies it alongside the `kernel.json` file it generates for Jupyter to be able to launch
Almond. That way, the launcher used during the installation can be safely deleted right after
the installation.

## Creating an Almond launcher and installing it

This section describes how to install Almond for a specific Scala version. Even
though the instructions below may rely on newer coursier features, installing Almond
this way has been the recommended way to proceed since Almond exists. In the next section,
we'll describe a novel way to install Almond, relying on an intermediate launcher, that allows
notebook users to customize the Scala version, the JVM they use, Java options (including
memory options), on a per notebook basis.

### `cs launch --use-bootstrap`

The easiest way to generate an Almond launcher consists in… not generating one. The
`cs launch` command, when passed `--use-bootstrap`, launches an application via a
temporary launcher it generates on-the-fly.

```text
$ cs launch --use-bootstrap almond:0.14.1 --scala 2.13.16 -- --install
```

Note the use of `--` - arguments before it are arguments for `cs`, those after are arguments
for Almond.

To list the options that the launcher accepts, run
```text
$ cs launch --use-bootstrap almond:0.14.1 --scala 2.13.16 -- --help
```

### `cs bootstrap`

Alternatively to using `cs launch --use-bootstrap`, you can generate a launcher with
`cs bootstrap`, then launch it on your own:
```text
$ cs bootstrap almond:0.14.1 --scala 2.13.16 -o almond
$ ./almond --install
$ rm -f almond
```

## Creating an Almond launcher and installing it - newer launcher

The newer launcher allows users to configure Almond in the first cells of notebooks
(more precisely, before any actual code - that is, not comments or blank lines - needs to be compiled), with
directives like
```scala
//> using scala "2.12"
//> using scala "2.12.20"
//> using jvm "17"
//> using javaOpt "-Xmx10g"
//> using javaOpt "-Dfoo=bar"
```

The newer launcher can be installed with
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- --scala 2.13.16 --install
```
(`sh.almond:launcher_3:0.14.1` also works as a dependency in case `cs` is having issues finding out the
`_3` suffix on its own)

Just like above, you can also generate a launcher on your own first, then use it install Almond:
```text
$ cs bootstrap sh.almond::launcher:0.14.1 -- --scala 2.13.16 -o almond --install
$ ./almond --install
$ rm -f almond
```

Important thing to note here: the Scala version must be passed as argument to Almond itself, rather than
to `cs`. The launcher uses its own Scala version (Scala 3), then, later on, spawns the same Almond kernel as above
on its own, that takes over as a kernel. The argument passed via `--scala` corresponds to the default
Scala version that will be used, if users don't specify a version of their own via a directive like
`//> using scala "2.13.16"`. Specify that option is optional: without it, Almond will default
to the Scala 3 version that Almond uses at the time of its release (which should be the latest stable Scala 3
version at the time of the release).

To list the options that the launcher accepts, run
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- --help
```

Note that some options can be passed directly to the kernel that the launcher will spawn at the beginning
of notebooks, after another `--`, like
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- --scala 2.13.16 --install -- --toree-api
```

To list the options that can be passed this way, run
```text
$ cs launch --use-bootstrap almond:0.14.1 --scala 2.13.16 -- --help
```

## Custom URL protocol support

Custom protocol support for `java.net.URL` needs the JARs supporting it to be passed to the `-cp`
option of `java`. Using the new launcher above, such JARs can be passed via `--extra-startup-class-path`, like
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- \
  --install \
  --scala 2.13.16 \
  --extra-startup-class-path "$(cs fetch io.get-coursier:s3-support:0.1.0)"
```
(In this example, we pass the JARs of [coursier s3-support](https://github.com/coursier/s3-support) to `-cp` via
`--extra-startup-class-path`.)

## Enabling Toree compatibility

### Core magics

Almond has some support for [Toree](https://github.com/apache/incubator-toree) ["magics"](https://github.com/apache/incubator-toree/blob/5b19aac2e56a56d35c888acc4ed5e549b1f4ed7c/etc/examples/notebooks/magic-tutorial.ipynb).
This support makes it easier to migrate from Toree to Almond for example.

Enable it with the `--toree-magics` option, that both the former and the new Almond launcher accept:
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- \
  --install \
  --scala 2.13.16 \
  --toree-magics
```

Alternatively to `--toree-magics`, pass `--toree-compatibility` that enables both `--toree-magics`
and `--toree-api` (see below).

You can then use magics like `%AddDeps`, `%AddJar`, `%LsMagic`, etc., in notebook cells.

### Spark magics

Support for the `%sql` magic, relying on Spark, needs to be enabled from a "predef" script. It also
requires parts of the Toree magics support to be put in the user-facing part of the Almond class path
(so that calls to it from the user side are picked up by Almond internals later on).

To enable that from a former launcher, pass `--shared sh.almond::toree-hooks` when generating the launcher:
```text
$ cs launch --use-bootstrap almond:0.14.1 --shared sh.almond::toree-hooks --scala 2.13.16 -- --install
```

To enable it from a new launcher, pass `--shared-dependencies sh.almond::toree-hooks:_` to the launcher, like
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- \
  --install \
  --scala 2.13.16 \
  --shared-dependencies sh.almond::toree-hooks:_ \
  --toree-magics
```

Then, from a predef script, call
```scala
almond.spark.ToreeSql.setup()
```

Complete example with the new launcher:
```text
$ cat predef.sc
import $ivy.`sh.almond::almond-toree-spark:0.14.1`
almond.spark.ToreeSql.setup()

$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- \
  --install \
  --scala 2.13.16 \
  --shared-dependencies sh.almond::toree-hooks:_ \
  --toree-magics \
  --predef predef.sc
```

### Toree API

Enable basic support for the Toree API with `--toree-api`.

Enable it with the `--toree-magics` option, that both the former and the new Almond launcher accept:
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- \
  --install \
  --scala 2.13.16 \
  --toree-api
```

Alternatively to `--toree-api`, pass `--toree-compatibility` that enables both `--toree-magics` (see above)
and `--toree-api`.

Some Toree API calls are then supported:
```scala
kernel.display.html("<b>foo</b>")
```

## Using custom Maven repositories

If you want to use custom Maven repositories rather than Maven Central, you need to set
[`COURSIER_REPOSITORIES`](https://github.com/coursier/coursier/blob/3e212b42d3bda5d80453b4e7804670ccf75d4197/doc/docs/other-repositories.md)
at few places. It needs to be set when installing Almond, and when Jupyter starts Almond.

This can be achieved the following way, with the former launcher:
```text
$ export COURSIER_REPOSITORIES="ivy2Local|https://artifacts.company.com/maven"
$ cs launch --use-bootstrap almond:0.14.1 --scala 2.13.16 -- \
    --env "COURSIER_REPOSITORIES=$COURSIER_REPOSITORIES"
```

`--env` sets environment variables in the kernel spec that gets written when installing Almond. Jupyter
sets those prior to launching Almond when users open notebooks.

## Available options

To list the options that the former launcher accepts, run
```text
$ cs launch --use-bootstrap almond:0.14.1 --scala 2.13.16 -- --help
```

To list the options that the newer launcher accepts, run
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- --help
```

Note that the newer launcher also accepts the former launcher options after an extra `--`, like
```text
$ cs launch --use-bootstrap sh.almond::launcher:0.14.1 -- --scala 2.13.16 --install -- --toree-api
```
