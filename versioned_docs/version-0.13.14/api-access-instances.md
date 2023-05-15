---
title: Access API instances
id: version-0.13.14-api-access-instances
original_id: api-access-instances
---

## From the Jupyter cells

These can be accessed directly by their names, like
```scala
interp
repl
kernel
```

or via implicits, like
```scala
implicitly[ammonite.interp.api.InterpAPI]
implicitly[ammonite.repl.api.ReplAPI]
implicitly[almond.api.JupyterApi]
```

## From a library

To access API instances from libraries, depend on either
- `com.lihaoyi:ammonite-interp_2.13.10:3.0.0-M0-23-f664d7ef` for [`ammonite.interp.api.InterpApi`](api-ammonite.md#interpapi),
- `com.lihaoyi:ammonite-repl_2.13.10:3.0.0-M0-23-f664d7ef` for [`ammonite.repl.api.ReplAPI`](api-ammonite.md#replapi),
- `sh.almond:scala-kernel-api_2.13.10:0.13.14` for [`almond.api.JupyterApi`](api-jupyter.md#jupyterapi).

You can depend on those libraries as "provided" dependencies, as these libraries
are guaranteed to be already loaded by almond itself.

In practice, you can add something along those lines in `build.sbt`,
```scala
libraryDependencies ++= Seq(
  ("com.lihaoyi" % "ammonite-interp" % "3.0.0-M0-23-f664d7ef" % Provided).cross(CrossVersion.full), // for ammonite.interp.api.InterpApi
  ("com.lihaoyi" % "ammonite-repl" % "3.0.0-M0-23-f664d7ef" % Provided).cross(CrossVersion.full), // for ammonite.repl.api.ReplAPI
  ("sh.almond" % "scala-kernel-api" % "0.13.14" % Provided).cross(CrossVersion.full) // for almond.api.JupyterApi
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
