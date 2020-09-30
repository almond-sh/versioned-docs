---
title: docker
id: version-0.10.7-try-docker
original_id: try-docker
---

Docker images of almond are automatically published on
[dockerhub](https://hub.docker.com/r/almondsh/almond) upon each release.

Run the latest version with

```
$ docker run -it --rm -p 8888:8888 almondsh/almond:latest
```

Run a specific version with

```
$ docker run -it --rm -p 8888:8888 almondsh/almond:0.10.7
```

Run a specific version with a specific Scala version with

```
$ docker run -it --rm -p 8888:8888 almondsh/almond:0.10.7-scala-2.12.8
```

See [here](install-versions.md) for the compatible Almond versions / Scala
versions.
