---
layout: post
title: "What's New in Java 11"
date: 2020-09-08 08:00:26 +1200
categories: [technical, java]
tags: [java, java-upgrade]
---

`Java 11` is the next `LTS`(Long Term Support) release after `Java 8`.

# New Java 11 features <!-- omit in toc -->

Some of the main features introduced in Java 11 are following:

- [New Licensing Model](#new-licensing-model)
- [JEP 330: Run Java with a Single Command and SheBang](#jep-330-run-java-with-a-single-command-and-shebang)
  - [Javac not Needed Anymore](#javac-not-needed-anymore)
  - [Use the SheBang in scripts](#use-the-shebang-in-scripts)
- [JEP 323: Local Variable Syntax for Lambda](#jep-323-local-variable-syntax-for-lambda)
- [JEP 320 Remove the Java EE and CORBA Modules](#jep-320-remove-the-java-ee-and-corba-modules)
- [JEP 328: Flight Recorder](#jep-328-flight-recorder)
- [Api changes](#api-changes)
  - [New String Methods](#new-string-methods)
  - [New Collection.toArray() function](#new-collectiontoarray-function)
  - [Files.readString() and Files.writeString()](#filesreadstring-and-fileswritestring)
  - [Optional.isEmpty method](#optionalisempty-method)
- [JEP 321: HTTP Client (Standard)](#jep-321-http-client-standard)
- [JEP 327: Unicode 10](#jep-327-unicode-10)
- [JEP 332: Transport Layer Security (TLS) 1.3](#jep-332-transport-layer-security-tls-13)
- [JEP 335: Deprecate the Nashorn JavaScript Engine](#jep-335-deprecate-the-nashorn-javascript-engine)
- [JEP 336: Deprecate the Pack200 Tools and API](#jep-336-deprecate-the-pack200-tools-and-api)

### New Licensing Model

Starting `Java 11` Oracle JDK Licensing has changed, you can use it for personal projects and learning for no cost.
If you want to use Oracle JDK for `Java 11` commercially, you will have to buy a commercial license from Oracle.

This is true for new updates to `Java 8` as well. You can continue using `Java 8` versions before `April 16, 2019` as you wish. But, if you want to get any newer updates you will have to buy a commercial license.

This doesn't mean you can't get Java for free now. Lots of companies, including `Oracle`, contribute to open source project called `OpenJDK`. `OpenJDK` is free alternative to `Oracle` JDK.

> You can get OpenJDK from [here](https://adoptopenjdk.net)

### JEP 330: Run Java with a Single Command and SheBang

#### Javac not Needed Anymore

These days not many people use command line to run and compile a Java file but still, some times it can be handy to run java program from a command line tool.

Before `Java 11` it was a 2 step process.

- first compile the file using `javac` command. E.g `javac MyJavaClass.java`.
- And then, run it `java MyJavaClass`

Starting `Java 11` you can compile and run a program with one command.
Eg. `java MyOtherClass.java`, no need to use `javac` anymore. This one command will both compile and run your class.

#### Use the SheBang in scripts

Single file java program can be run using `Shebang`.

> Shebang is the first line of a shell script, which specifies the binary to use for the script. Generally its used to define which shell will be used eg. bash, sh etc.

```shell
#!/opt/java/openjdk/bin/java --source 11
public class HelloWorld {

    public static void main(String[] args) {

        System.out.println("Hello World!,");

    }
}
```

Now you can run this program like you would run a script.

```
./HelloWorld.sh
```

### JEP 323: Local Variable Syntax for Lambda

`Java 10` introduced `var` keyword for automatic type inference. In `Java 11` Lambda's also support `var` keyword in lambda parameters.

Consider the following Lambda expression

```java
Function<String,String> convertCase = (input) -> input.toUpperCase();
```

This lambda takes a String and converts it too upper case. Note that `Function` is predefined functional interface in `java.util.function` package

```java
String convertedString = convertCase.apply("caps are considered shouting in chat");
// convertedString is set to CAPS ARE CONSIDERED SHOUTING IN CHAT
```

In `Java 11`, we could use `var` keyword.

```java
Function<String,String> convertCase = (var input) -> input.toUpperCase();
```

This isn't any better from the previous declaration of convertCase. But using `var` allows you to use annotations, which is an improvement

```java
Function<String,String> convertCase = (@NotNull var input) -> input.toUpperCase();
```

> NotNull is not part of standard java, although there are many common implementation of NotNull in various libraries.

### JEP 320 Remove the Java EE and CORBA Modules

Following packages were removed from `Java 11`.

- `java.xml.ws` (JAX-WS, plus the related technologies SAAJ and Web Services Metadata)
- `java.xml.bind` (JAXB)
- `java.activation` (JAF)
- `java.xml.ws.annotation` (Common Annotations)
- `java.corba` (CORBA)
- `java.transaction` (JTA)
- `java.se.ee` (Aggregator module for the six modules above)
- `jdk.xml.ws` (Tools for JAX-WS)
- `jdk.xml.bind` (Tools for JAXB)

### JEP 328: Flight Recorder

Flight Recorder is a profiling tool to diagnose a running Java application.
It has existed for many years and was previously a commercial feature of the Oracle JDK. Starting `Java 11` it is open source.

### Api changes

#### New String Methods

- `String repeat(int count)`

  As the name suggests it repeats the string its invoked on `count` number of times.

  ```java
  String str = "A".repeat(5)
  // str contains "AAAAA"
  ```

  If string is empty or count is 0 then empty string is returned.

- `boolean isBlank()`

  Returns true if string is empty or only contains [whitespace](<https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#isWhitespace(char)>).

  ```java
  boolean blank = "".isBlank();// true
  blank = " ".isBlank(); // true
  blank = " v ".isBlank(); // false
  ```

- `Stream<String> lines()`

  Returns a stream of lines extracted from the string it is called on.
  Lines in strings are separated by line terminators ie. `\n`, `\r`, `\r\n`.

  ```java
  "my \n name \n is \n vikram".lines().forEach(System.out::println)
  // prints following:
  // my
  // name
  // is
  // vikram

  ```

  above is equivalent to having a list of string, converting it to a Stream and then apply string operations, in this case foreach

  ```
  List<String> list = List.of("my ", "name ", "is ", "vikram");
  list.stream().forEach(System.out::println);
  ```

#### New Collection.toArray() function

New function has been added to `Collection` interface.

` <T> T[] toArray​(IntFunction<T[]> generator)`

`IntFunction` is an functional interface which allocates the returned array.

Lets convert a List of String to an array

```java
List<String> list = List.of("one","two", "three");
String[] array = list.toArray(String[]::new);
// array ==> String[3] { "one", "two", "three" }

```

#### Files.readString() and Files.writeString()

- Files.readString: reads a file to a String. This method ensures file is properly closed when reading is finished.

  It has 2 overloads

  - `public static String readString​(Path path, Charset cs) throws IOException`

    This method converts all the bytes read into the provided charset.

  - `public static String readString​(Path path) throws IOException`

    This is equivalent to calling first method with charset `UTF 8`.

- Files.writeString: As you might have guessed, this function can be used to write a String to a file.

  It also has 2 overloads.

  - `public static Path writeString​(Path path, CharSequence csq, Charset cs, OpenOption... options) throws IOException`
    Writes a the Character sequence specified at the path provided,in the given charset. Uses options to decide how the file should be opened.
  - `public static Path writeString​(Path path, CharSequence csq, OpenOption... options) throws IOException`
    In this overload version charsequence is `UTF 8`.

Lets see a code example

```java
jshell> String secret = "Secret to success is hard work!"
secret ==> "Secret to success is hard work!"
jshell> Path path = Files.writeString(Files.createTempFile("secret", ".txt"),secret);
path ==> C:\Users\vikramsingh\AppData\Local\Temp\secret3433019555033269602.txt

jshell> String s = Files.readString(path);
s ==> "Secret to success is hard work"
```

#### Optional.isEmpty method

`Optional` class already contains a method `isPresent` which is true when optional object contains a value.

`isEmpty` is opposite of `isPresent` i.e its true when optional object doesn't contains a value.

Idea behind is to remove negative conditions when checking value is not `isPresent` i.e instead of using `!optional.isPresent` now we can use use `optional.isEmpty`

```java
String name=null;
Optional<String> strOptional = Optional.ofNullable(name);

System.out.println(strOptional.isPresent())  // false
System.out.println(!strOptional.isPresent()) // true
System.out.println(strOptional.isEmpty());   // true

```

### JEP 321: HTTP Client (Standard)

New Http Client was added in `Java 9` as an early preview for feedback by users.
Starting `Java 11` this new Http Client standardized and can be used in commercial applications. Its available in package `java.net.http` package.

In `Java 9 and 10`, it was available at `jdk.incubator.http`, so if you were already trying the incubator version just update the import.

This client introduces support to HTTP/2 and allows non-blocking request and response semantics through `CompletableFutures`.

### JEP 327: Unicode 10

`Java 11` supports version 10 of Unicode Standard, which means more special symbols/emoji's available in Java.

### JEP 332: Transport Layer Security (TLS) 1.3

`Java 11` supports TLS 1.3, however not all TLS 1.3 features are supported.

### JEP 335: Deprecate the Nashorn JavaScript Engine

Nashorn JavaScript script engine and APIs and the jjs tool, introduced in `Java 8`, have been deprecated and would be removed in a future version of Java.

### JEP 336: Deprecate the Pack200 Tools and API

The pack200 and unpack200 tools, and the Pack200 API in java.util.jar package have been deprecated and it will probably remove in the future release.
