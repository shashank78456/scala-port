# MetaCall Scala Port

## Setup

To set up Scala & SBT, use [Coursier](https://get-coursier.io/docs/cli-installation). After getting the `cs` executable, run `cs setup` and follow the prompt.

## Testing

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

To run the tests in Docker, run `sbt docker` to build the image, and then `sbt dockerTest` to run it. Note that you should build the `metacall/core:dev` image locally since the published one might not be up to date by running `./docker-compose.sh build` in `metacall/core`'s root. Pay attention to SBT's error messages.

## Debugging

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
