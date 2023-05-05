---
title: Other
id: version-0.13.13-install-other
original_id: install-other
---

## All-included launcher

When launched by other users, the kernel installed by the default command
downloads its dependencies in the user-specific [coursier cache](https://get-coursier.io/docs/cache.html#location)
upon first launch.

You may prefer to have the launcher embed all these JARs,
so that nothing needs to be downloaded or picked from a cache upon launch. Passing
`--standalone` to the `coursier bootstrap` command generates such a launcher,
```bash
$ coursier bootstrap --standalone \
    almond:0.13.13 --scala 2.13.10 \
    -o almond
$ ./almond --install
$ rm -f almond # the generated launcher can be removed after install, it copied itself in the kernel installation directory
```

