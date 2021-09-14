# Compiling and Running Kotlin Programs

Consider the following Kotlin program:

```kotlin
// Hello.kt

fun main() = println("Hello world!")
```

You can produce a jar file via the command line:

```console
> kotlinc-jvm Hello.kt -d Hello.jar
```

This compiles the code in `Hello.kt` into [Java bytecode](https://en.wikipedia.org/wiki/Java_bytecode) and places that
into the `Hello.jar` file.

To run with the java tool:

```console
> java -jar Hello.jar
```

However, any nontrivial program will make use of classes and functions from the Kotlin standard library and, in that
case, the above execution using the java tool will fail with a `java.lang.NoClassDefFoundError` exception. To avoid
this, include the `kotlin-stdlib.jar` file to the classpath, like so:

```console
> kotlinc-jvm Hello.kt -d Hello.jar
> java -classpath Hello.jar:$KOTLIN_PATH/lib/kotlin-stdlib.jar HelloKt
```

Instead of using the `java` tool, one may also use the `kotlin` tool. In this case, there is no need to refer to
`kotlin-stdlib.jar`. Just run the code using `kotlin`:

```console
> kotlinc-jvm Hello.kt -d Hello.jar
> kotlin -classpath Hello.jar HelloKt
```

*Note*: Since the file `Hello.kt` contains only the main function and not a class, the Kotlin compiler, `kotlinc-jvm`,
automatically creates a class named after the file name, without the `.kt` extension, with an added `Kt` suffix.

## Scripts

Running a script is slightly different:

```console
> kotlinc-jvm -script kts-script.kts
```

*Note*: The `kts` extension indicates this file is a script.

Another option:

Check out `greet.kts` and see the use of the "shebang" facility:

```kotlin

// greet.kts

#!/ usr / bin / env kotlinc - jvm - script

println("hello!")

```

With `chmod +x greet.kts`, one can then just run `./greet.kts`.