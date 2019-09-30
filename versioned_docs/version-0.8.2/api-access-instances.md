---
title: Access API instances
id: version-0.8.2-api-access-instances
original_id: api-access-instances
---

## From the REPL

These can be accessed directly by their names, like

```scala
interp
repl
kernel
```

or via implicits, like

```scala
implicitly[ammonite.interp.InterpAPI]
implicitly[ammonite.repl.ReplAPI]
implicitly[almond.api.JupyterAPI]
```

## From a library

To access API instances from libraries, depend on either
- `com.lihaoyi:ammonite-interp_2.12.8:1.7.4` for [`ammonite.interp.InterpAPI`](api-ammonite.md#interpapi),
- `com.lihaoyi:ammonite-repl_2.12.8:1.7.4` for [`ammonite.repl.ReplAPI`](api-ammonite.md#replapi),
- `sh.almond:scala-kernel-api_2.12.8:0.8.2` for [`almond.api.JupyterAPI`](api-jupyter.md#jupyterapi).

You can depend on those libraries as "provided" dependencies, as these libraries
are guaranteed to be already loaded by almond itself.

In practice, you can add something along those lines in `build.sbt`,

```scala
libraryDependencies ++= Seq(
  ("com.lihaoyi" % "ammonite-interp" % "1.7.4" % Provided).cross(CrossVersion.full), // for ammonite.interp.InterpAPI
  ("com.lihaoyi" % "ammonite-repl" % "1.7.4" % Provided).cross(CrossVersion.full), // for ammonite.repl.ReplAPI
  ("sh.almond" % "scala-kernel-api" % "0.8.2" % Provided).cross(CrossVersion.full) // for almond.api.JupyterAPI
)
```

You can then write methods looking for implicit
API instances, like

```scala
def displayTimer()(implicit kernel: almond.api.JupyterApi): Unit = {
  val count = 5
  val id = java.util.UUID.randomUUID().toString
  kernel.publish.html(s"<b>$count</b>", id)
  for (i <- (0 until count).reverse) {
    Thread.sleep(1000L)
    kernel.publish.updateHtml(s"<b>$i</b>", id)
  }
  Thread.sleep(200L)
  kernel.publish.updateHtml(Character.toChars(0x1f981).mkString, id)
}
```

Users can then load the library defining this method, and call it themselves
from a notebook. The library can interact with the front-end, without overhead
for users.

A library defining this method, along with instructions to set up the library and
a notebook using it, are available in
[this repository](https://github.com/almond-sh/example-library-jupyter-api).
