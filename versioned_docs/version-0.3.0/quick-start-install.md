---
title: Installation
id: version-0.3.0-quick-start-install
original_id: quick-start-install
---

## Create a launcher

1. Set the desired version of Scala and Almond as environment variables:

```bash
$ SCALA_VERSION=2.12.8 ALMOND_VERSION=0.3.0
```

The available versions of Scala and Almond are can be found [here](https://github.com/almond.sh/almond/releases).
Adjust `ALMOND_VERSION` and `SCALA_VERSION` at your convenience to meet your
needs. Not all combinations are guaranteed to be available. See the available
combinations [here](install-versions.md)).

2. Create a launcher via [coursier](http://get-coursier.io):

```bash
$ coursier bootstrap \
    -r jitpack \
    -i user -I user:sh.almond:scala-kernel-api_$SCALA_VERSION:$ALMOND_VERSION \
    sh.almond:scala-kernel_$SCALA_VERSION:$ALMOND_VERSION \
    -o almond
```

## Install the almond kernel

3. Run the launcher to install the almond kernel:

```bash
$ ./almond --install
```

4. Once the kernel is installed, you can use it within Jupyter or nteract.

If you are satisfied that the kernel is working properly, you may safely
remove the almond launcher: `rm -f almond`

## Getting help about the launcher

- Help: `./almond --help`
- Available options:

Once the kernel is installed, the generated launcher can then be safely removed, with `rm -f almond`.
