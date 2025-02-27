---
title: Multiple kernels
id: version-0.14.1-install-multiple
original_id: install-multiple
---

Several versions of almond can be installed side-by-side. This is useful e.g. to have kernels
for several Scala versions, or to test newer / former almond versions.

To install several version of the kernel side-by-side, just ensure the different installed versions
have different ids (required) and display names (recommended).

For example, let's install almond for the scala `2.13.16` version,
```bash
$ cs launch almond --scala 2.13.16 -- --install
```

This installs almond with the default kernel id, `scala`, and default display name, "Scala".

Now let's *also* install almond for scala `2.12.20`,
```bash
$ cs launch almond:0.14.1 --scala 2.12.20 \
      -- --install --id scala212 --display-name "Scala (2.12)"
```

`--id scala212` ensures this kernel is installed along side the former, in a directory
with a different name.

`--display-name "Scala (2.12)"` ensures users can differentiate both kernels, with the latter
appearing in front-ends under the name "Scala (2.12)".
