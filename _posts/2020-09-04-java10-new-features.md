---
layout: post
title: "What's New in Java 10"
date: 2020-09-04 02:52:26 +1200
categories: [technical, java]
tags: [java, java-upgrade]
---

`Java 10` was one of the fastest release of a new Java version. It was released just 6 months after `Java 9`. Starting this release Java moved to a 6 month release model, meaning a Java version will be released every 6 months. Release months would be March and September.

# New Java 10 features <!-- omit in toc -->

Some of the main features introduced in Java 10 are following:

- [JEP 322: Time-Based Release Versioning](#jep-322-time-based-release-versioning)
- [JEP 286: Local-Variable Type Inference](#jep-286-local-variable-type-inference)
  - [Rules of using type inference](#rules-of-using-type-inference)
  - [Tips and Traps](#tips-and-traps)
- [Api changes](#api-changes)
  - [New orElseThrow method in Optional class](#new-orelsethrow-method-in-optional-class)
  - [Apis for Creating Unmodifiable Copy of Collections](#apis-for-creating-unmodifiable-copy-of-collections)
  - [New Apis for Creating Unmodifiable Collections from Stream](#new-apis-for-creating-unmodifiable-collections-from-stream)

### JEP 322: Time-Based Release Versioning

With the new release cycle, java introduced a new versioning model.
New versioning as follows.

```
$FEATURE.$INTERIM.$UPDATE.$PATCH
```

**\$FEATURE**: Feature release counter. This is the main java version, eg. for Java 10 it's 10. It will changes with every new release of java eg. for Java 11, 12, and so on. This was formerly called `$MAJOR` version in earlier releases.

**\$INTERIM**: It is the interim release counter, for non feature releases. In the current release model this will always be `0`. It was added to make this versioning system future proof. This was formerly called `$MINOR`.

**\$UPDATE**: It is the update release counter. Its increased when fixes to security issues, regressions, and bugs in newer features are added to release. This was formerly called `$SECURITY`. This will increment 1 month after `$FEATURE` is incremented and then every 3 months after that. Eg. March 2018 release was `10.0.0`, April 2018 release was `10.0.1` , July 2018 release was `10.0.2` and so on.

**\$PATCH**: It is the emergency patch-release counter. Its incremented only when there is an emergency release to fix a critical issue.

You can see your java version on command line

```shell
root@5843268d0e41:/# java -version
openjdk version "10.0.2" 2018-07-17
OpenJDK Runtime Environment (build 10.0.2+13-Debian-2)
OpenJDK 64-Bit Server VM (build 10.0.2+13-Debian-2, mixed mode)

```

There were some changes to `Version` Class to support the new release versioning.
Following new methods were introduced:

```
int 	feature()
int 	hashCode()
int 	interim()
```

```shell
root@5843268d0e41:/# jshell
jshell> import java.lang.Runtime.Version

jshell> Version version = Runtime.version();
version ==> 10.0.2+13-Debian-2

jshell> version.feature()
$3 ==> 10

jshell> version.interim()
$4 ==> 0

jshell> version.update()
$5 ==> 2

jshell> version.patch()
$6 ==> 0
```

### JEP 286: Local-Variable Type Inference

Type inference of local variables is a very exciting feature. Similar feature is present in languages like `C#` and `go` since some time. Its addition to java is a welcome change.

Idea behind this feature is to make code both easier to type and read.
Type inferred variables are created using `var` keyword.
For eg.

**Java 9** (no type inference)

```java
List<Integer> list = List.of(1,2);
ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
List<Map<String,String>> listOfMaps = List.of(Map.of("k1","v1"),Map.of("k2","v2"));
```

**Java 10** (using type inference)

```java
var intList = List.of(1,2);
var outputStream = new ByteArrayOutputStream();
var listOfMaps = List.of(Map.of("k1","v1"),Map.of("k2","v2"));
```

You can see above using `var` keyword improves the readability, as information on left hand side of equals is often repeated on right hand side

#### Rules of using type inference

- Type inference can be only used for
  - Local variables with initializers.
  - Index variable in enhanced for-loop and traditional for-loop.

#### Tips and Traps

- When declaring variable always assign a value.
  ```java
  var a        // error : cannot use 'var' on variable without initializer
  var a = null // error : variable initializer is 'null'
  var a = 1    // good
  ```
- Lambda or method reference is not a valid value.
  ```java
  var b = ()->5;         // error: lambda expression needs an explicit target-type
  var b = Object::equals // error: method reference needs an explicit target-type
  ```
- Array initializer should be explicit.
  ```java
  var arr = { 1, 2 }    // error array initializer needs an explicit target-type
  var arr = new int[]{ 1, 2 } // good
  ```
- Take care when using diamond or Generic methods

  ```java
  var list = List.of(1,2);  // infers List<Integer>
  var list = List.of();     // infers List<Object>

  var students = new ArrayList<Students>(); // infers ArrayList<Students>
  var students = new ArrayList<>(); // infers ArrayList<Object>
  ```

- Numeric literals can silently widened or narrowed

```java
byte flag = 0
long longValue = 1

// unexpected widening or narrowing
var flag = 0 // inferred as int
var longValue = 1 // inferred as int.
```

You can see in general using type-inference reduces clutter and improves readability, but it should be used with care to avoid unforeseen side affects.

### Api changes

#### New orElseThrow method in Optional class

A new method `orElseThrow` has been added to the `Optional` class. This works exactly same as already existing `get` method in `Optional` class. It just highlights the fact that an exception (NoSuchElementException) is thrown if optional is empty.

Use of `orElseThrow` is preferred over `get` method

```java
jshell> Optional optional = Optional.of(1)
optional ==> Optional[1]

jshell> optional.orElseThrow()
$2 ==> 1

jshell> optional.get()
$3 ==> 1

jshell> Optional optional2 = Optional.empty()
optional2 ==> Optional.empty

jshell> optional2.orElseThrow()
|  java.util.NoSuchElementException thrown: No value present
|        at Optional.orElseThrow (Optional.java:371)
|        at (#6:1)

jshell> optional2.get()
|  java.util.NoSuchElementException thrown: No value present
|        at Optional.get (Optional.java:148)
|        at (#8:1)

```

#### Apis for Creating Unmodifiable Copy of Collections

- `List.copyOf`, `Set.copyOf`, and `Map.copyOf` methods create new unmodifiable collection instances from existing instances.

  ```java
  jshell>   List<Integer> list1 = new ArrayList<>();
  list1 ==> []

  jshell>   list1.add(1);
  $10 ==> true

  jshell>   var list2 = List.copyOf(list1)
  list2 ==> [1]
  // list2 is unmodifiable.
  jshell> list2.add(2)
  |  java.lang.UnsupportedOperationException thrown
  |        at ImmutableCollections.uoe (ImmutableCollections.java:71)
  |        at ImmutableCollections$AbstractImmutableList.add (ImmutableCollections.java:77)
  |        at (#12:1)
  ```

#### New Apis for Creating Unmodifiable Collections from Stream

New methods `toUnmodifiableList`, `toUnmodifiableSet`, and `toUnmodifiableMap` have been added to the Collectors class in the Stream package.

Before `Java 10` we could create a list from a Stream using `Collectors.toList()` method. This list could be modified.

```
jshell> var list1 = Stream.of(1,2,3).collect(Collectors.toList());
list1 ==> [1, 2, 3]

jshell> list1.add(4)
$18 ==> true
```

Using the newer api methods we can create an unmodifiable collection.

```java
jshell> var list1 = Stream.of(1,2,3).collect(Collectors.toUnmodifiableList());
list1 ==> [1, 2, 3]

jshell> list1.add(4)
|  java.lang.UnsupportedOperationException thrown
|        at ImmutableCollections.uoe (ImmutableCollections.java:71)
|        at ImmutableCollections$AbstractImmutableList.add (ImmutableCollections.java:77)
|        at (#21:1)
```

Similarly we can also create unmodifiable sets and maps.
