---
layout: post
title: "What's New in Java 9"
date: 2020-08-29 16:52:26 +1200
categories: [technical, java]
tags: [java, java-upgrade]
---

# New Java 9 features <!-- omit in toc -->

Some of the main features introduced in Java 9 are following:

- [Jshell a.k.a REPL](#jshell-aka-repl)
- [Convenience methods for creating unModifiable List, Set, Map](#convenience-methods-for-creating-unmodifiable-list-set-map)
- [Private methods in interfaces... Wait What ?](#private-methods-in-interfaces-wait-what-)
- [Try With Resources Enhancement](#try-with-resources-enhancement)

> This is not an exhaustive list, its missing things like `Jigsaw Project`. This blog post contains some of the more commonly used new features.

### Jshell a.k.a REPL

With this new handy tool, you can test java code straight from command line without creating and compiling classes.

You can start jshell by typing `jshell` in the command line.

```shell
$ jshell
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro
```

To print something, you can write a println call without need of any classes.

```java
System.out.println("Vikram testing Java 9")
Vikram testing Java 9
```

Notice there is not need to put a semicolon at the end.

jshell remembers the variables you create, you can create as many variables you want and use them later.

```java
jshell> String str1="Be "
str1 ==> "Be "

jshell> String str2 ="Happy"
str2 ==> "Happy"

jshell> str1+str2
$6 ==> "Be Happy"
```

You can also create methods

```java
jshell> int sum(int a, int b) {
   ...> return a+b;
   ...> }
|  created method sum(int,int)

jshell> sum(5,10)
$9 ==> 15
```

There is no limit to what you can do in `jshell`, you can define classes, enums etc.

You can have a look all the variables defined in your `jshell` using `/var` command.

```java
jshell> /var
|    String str1 = "Be "
|    String str2 = "Happy"
|    String $6 = "Be Happy"
```

variables starting with `$` sign are called scratch variables. Theses are created by jshell to hold temporary values, which may be result of commands you execute etc.

All the methods defined can be seen using `/methods`

```java
jshell> /methods
|    int sum(int,int)
```

To list everything you have done till now use `/list`

```java
jshell> /list
   1 : System.out.println("Vikram testing Java 9")
   2 : String str1="Be ";
   3 : String str2 ="Happy";
   4 : str1+str2
   5 : int sum(int a, int b) {
       return a+b;
       }
   6 : sum(5,10)
```

`jshell` puts some imports by default, to see all the code in use, run `/list --all`

```java

jshell> /list --all

  s1 : import java.io.*;
  s2 : import java.math.*;
  s3 : import java.net.*;
  s4 : import java.nio.file.*;
  s5 : import java.util.*;
  s6 : import java.util.concurrent.*;
  s7 : import java.util.function.*;
  s8 : import java.util.prefs.*;
  s9 : import java.util.regex.*;
 s10 : import java.util.stream.*;
   1 : System.out.println("Vikram testing Java 9")
   2 : String str1="Be ";
   3 : String str2 ="Happy";
   4 : str1+str2
   5 : int sum(int a, int b) {
       return a+b;
       }
   6 : sum(5,10)
```

To exit `jshell` use `/exit`

```shell
jshell> /exit
```

### Convenience methods for creating unModifiable List, Set, Map

Before `Java 9` to create an unModifiable collection. Programmers first had to create a collection and then use Collection.unModifiableXXX() eg.

```java
// Java 8 or earlier
Set<Integer> set = new HashSet<>();
set.add(1);
set.add(2);
set.add(3);

Set<Integer> unModifiableSet = Collections.unmodifiableSet(set);
```

This is quite verbose.

`Java 9` adds `.of()` to List, Set, and Map interface.

```java
// Java 9 and higher
Set<Integer> unModifiableSet = Set.of(1,2,3);
List<Integer> unModifiableList = List.of(1,2,3,4);
```

For Map even number of arguments are needed.

```java
Map<String,String> emptyMap = Map.of(); // empty Map
Map<String,String> map = Map.of("Key1","Value 1", "Key 2", "Value 2");
```

Maximum 10 key-value pairs can be supplied in `Map.of` method.

For N number of key values you could use `ofEntries` method.

```shell
jshell> Map.ofEntries(entry("key1","value1"),entry("key2","value2"))
$16 ==> {key2=value2, key1=value1}
```

### Private methods in interfaces... Wait What ?

It could look surprising to have private methods in interfaces. Of course these private methods can't be overridden by implementing classes but they can serve some purpose.

Starting from `java 8` interfaces can have `default` and `static` methods with some code.
Newly introduced private methods can reduce duplicate code and break extremely large default methods into smaller chunks for better readability and ease of maintenance.

Rules for private methods in interface

- Private methods can't be abstract.
- Private methods can be either static or non-static.
- Static private methods can be used in any other method of interface (static, default, other private methods etc.)
- Non static private methods can't be used in static methods.
- Private members can't be overridden (obviously).

See this example interface below.

```java
public interface TestInterface {

    // simple abstract method
    public abstract void method1();

    // private method can be used inside non static members of this interface.
    private void method2(){
        System.out.println("private non static method 2");
    }

    // can only be used inside static or non static  methods of this interface.
    private static void method3(){
        System.out.println("private static method 3 ");
    }
    public default void method4(){
        method2(); // private non static member
        method3(); // private static member
        System.out.println("Default method 5");
    }
    public static void method5() {
        // method2(); // cant call non static member in static function
        method3(); // private static member
        System.out.println("Default method 5");
    }
}
```

### Try With Resources Enhancement

Try with resources was introduced in `java 7`. It is a fantastic way of managing `AutoClosable` resources.

Before `java 9` resource had to be created inside the `try`

**Java 8 Example**

```java
 try (BufferedReader bufferReader = new BufferedReader(new FileReader("vikram.md"));) {
   System.out.println(bufferReader.readLine());
 }
```

In Java 9 you can create resources outside `try` statement but remember that resources should be either `final` or effectively `final`.

**Java 9 Example**

```java
final BufferedReader bufferReader2 = new BufferedReader(new FileReader("vikram.md"));
 try (bufferReader2) {
   System.out.println(bufferReader2.readLine());
 }
```

You can see ability to create resources outside `try` statement greatly improves the readability. This is specially true in case multiple resources are needed within same try with resource.
