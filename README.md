# MetaCall Scala Port

A library for calling NodeJS, Python, and Ruby functions from Scala.

```js
// myfunctions.js

function hello(x) {
    return 'Hello, ' + x
}

module.exports = { hello }
```
```scala
// Main.scala

import metacall._, instances._
import java.nio.file.Paths
import scala.concurrent.{Future, Await, ExecutionContext}
import scala.concurrent.duration._
import ExecutionContext.Implicits.global

object Main extends App {
  Caller.start(ExecutionContext.global)

  val future: Future[Value] = for {
    _      <- Caller.loadFile(Runtime.Node, Paths.get("./myfunctions.js").toAbsolutePath.toString)
    result <- Caller.call("hello", "World!")
  } yield result

  println(Await.result(future, 1.second))
  // metacall.StringValue("Hello, World!")

  Caller.destroy()
}
```

## Development
### Setup

To set up Scala & SBT, use [Coursier](https://get-coursier.io/docs/cli-installation). After getting the `cs` executable, run `cs setup` and follow the prompt.

Compiling requires setting either the `GITHUB_TOKEN` environment variable, or a `github.token` Git configuration. Use `export GITHUB_TOKEN=<token>` or `git config --global github.token <token>`, where `<token>` can be generated in your GitHub account's [settings](https://github.com/settings/tokens).

### Testing

To run the tests, run `sbt test` in this README's directory.

Don't forget to set these environment variables:
```
LOADER_SCRIPT_PATH
LOADER_LIBRARY_PATH
CONFIGURATION_PATH
SERIAL_LIBRARY_PATH
DETOUR_LIBRARY_PATH
PORT_LIBRARY_PATH
```

These variables are set automatically if MetaCall is intalled from source, i.e. using `sudo make install`:
```sh
cd ./core
mkdir build && cd build
cmake .. # Use loader flags as specified in https://github.com/metacall/core/blob/develop/docs/README.md#6-build-system
sudo make install
```

> You need to set `LOADER_LIBRARY_PATH` to the build directory created in the script above before running `sbt`, i.e. `LOADER_LIBRARY_PATH=path/to/core/build sbt`

To run the tests in Docker, run `sbt` then `docker` to build the image (must run `docker` from within the SBT session), and then `sbt dockerTest` to run it. Note that you should build the `metacall/core:dev` image locally since the published one might not be up to date by running `./docker-compose.sh build` in `metacall/core`'s root. Pay attention to SBT's error messages.

### Debugging

Uncomment this line in `build.sbt`:
```
"-Djava.compiler=NONE",
```

Build the project:
```
sbt compile
```

For runing valgrind with the correct classpath, run:
```
sbt "export test:fullClasspath"
```

Then copy the classpath into the valgrind command:
```
valgrind --tool=memcheck --trace-children=yes --error-limit=no scala -Djava.compiler=NONE -cp <classpath> src/test/scala/MetaCallSpecMain.scala
```

# Publishing

Use `sbt publish` to publish to GitHub Packages using [sbt-github-packages](https://github.com/djspiewak/sbt-github-packages). Make sure your GitHub token is set correctly according to [Setup](#setup).