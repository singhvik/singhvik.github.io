---
layout: post
title: "What's New in Java 12"
date: 2020-09-21 00:00:26 +1200
categories: [technical, java]
tags: [java, java-upgrade]
---

`Java 12` is a `Non-LTS` version of Java.

# New Java 12 features <!-- omit in toc -->

Some of the main features introduced in Java 12 are following:

- [JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)](#jep-189-shenandoah-a-low-pause-time-garbage-collector-experimental)
- [JEP 230: Microbenchmark Suite](#jep-230-microbenchmark-suite)
- [JEP 325: Switch Expressions (Preview)](#jep-325-switch-expressions-preview)
- [JEP 340: One AArch64 Port, Not Two](#jep-340-one-aarch64-port-not-two)
- [JEP 341: Default CDS Archives](#jep-341-default-cds-archives)
- [JEP 344: Abortable Mixed Collections for G1](#jep-344-abortable-mixed-collections-for-g1)
- [JEP 346: Promptly Return Unused Committed Memory from G1](#jep-346-promptly-return-unused-committed-memory-from-g1)
- [New Api changes](#new-api-changes)
  - [Compact Number Formatting](#compact-number-formatting)
  - [Files.mismatch()](#filesmismatch)
  - [New Teeing Collector](#new-teeing-collector)
  - [New String methods](#new-string-methods)
    - [String indent()](#string-indent)
    - [transform​()](#transform)
    - [describeConstable()](#describeconstable)
    - [resolveConstantDesc​()](#resolveconstantdesc)

## JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)

`Java 12` adds an experimental Garbage Collector(GC) called Shenandoah. This GC does evacuation in parallel to the running java threads. Pause times in this GC are independent of heap size, i.e pause times will be consistent whether you have heap size of 100MB or 100 GB.

This GC can come in handy for application requiring high responsiveness and availability since it will have a very small pause time.

## JEP 230: Microbenchmark Suite

This feature add a suite of benchmarks to the JDK source code and also provide developers ability to add new benchmarks. This comes in handy when some one wants to check the performance of JDK after a change they made.

These bench marks are based on Java Microbenchmark Harness (JMH).

## JEP 325: Switch Expressions (Preview)

> This is a preview feature and can be changed/removed in upcoming releases. Don't use this in production until its finalized.

This is quite an interesting change to the switch case statement.

Traditionally switch cases looks like this :

```java
switch(day){
  case "M" : System.out.println("Start of week");
            break;
  case "T" :
  case "W" :
  case "TH": System.out.println("Mid week");
             break;
  case "F" : System.out.println("Last work day");
             break;
  default : System.out.println("Its not a week day, must be weekend. woot! woot!");
}

```

In `Java 12` you can use the new syntax.

```java
switch(day){
  case "M" -> System.out.println("Start of week");
  case "T", "W", "TH" -> System.out.println("Mid week");
  case "F" -> System.out.println("Last work day");
  default -> System.out.println("Its not a week day, must be weekend. woot! woot!");
}
```

**Things to note**

- Cascading cases from our traditional switch cases have been combined using comma,
- Break statement is not required as cases don't fall through like in traditional switch case
- each case has the syntax `case Label -> `

Another change in the new syntax of switch is that switch can be used as an expression and return a value.

```java
String str = switch(day){
  case "M" -> "Start of week";
  case "T", "W", "TH" -> "Mid week";
  case "F" -> "Last work day";
  default -> "Its not a week day, must be weekend. woot! woot!";
};
```

In the above example each case return a value, and based on input variable `day` a value is returned and assigned to str.
Notice the semicolon after the closing brace of switch completing the assignment to variable `str`.

> Note: A single line Lambda expression returns the result of expression by default.

When using switch as an expression to return value. It is required that every case returns a value. The `break` keyword has been enhanced to return a value when a code block is assigned to a case for example:

```java
String str = switch(day){
  case "M" -> {
    String returnValue = "Start of week";
    break returnValue;
  }
  case "T", "W", "TH" -> "Mid week";
  case "F" -> "Last work day";
  default -> "Its not a week day, must be weekend. woot! woot!";
};
```

`break` is returning value from `case "M"` without break statement, this code will not compile.

`break` can be used to return values from traditional style switch expressions as well.

```java
String str = switch(day){
  case "M": break "Start of week";
  case "T", "W", "TH": break "Mid week";
  case "F": break "Last work day";
  default : break "Its not a week day, must be weekend. woot! woot!";
};
```

When assigning value using switch case, all possible cases or a default case should be specified.

Lets summarize all the rules

- Cases can be defined using new syntax `case Label -> `
- With the new syntax cases don't fall through, i.e explicit break is not needed.
- Multiple cases can be combined using comma (`,`) both in traditional style switch and new style switch.
- Switch can be used to return a value. In this case below additional rules apply.
  - All cases must return a value or throw an error.
  - Cases must be exhaustive,ie all possible cases present, or a default case should be present.
  - Single line case with new syntax `case Label -> ` automatically return a value.
  - For multiline cases `break` can be used for returning values.
  - `continue`, `return` can not be used to break out of normal switch workflow.

## JEP 340: One AArch64 Port, Not Two

Before Java 12, JDK had two different ports or implementations for the 64-bit ARM architecture, located at :

- `src/hotspot/cpu/arm` (contributed by Oracle)
- `src/hotspot/cpu/aarch64`

Java 12 removed the Oracle version `src/hotspot/cpu/arm port` and there by making the default (and only) build for the 64-bit ARM architecture.

## JEP 341: Default CDS Archives

CDS stands for Class Data Sharing, its is used to improve the startup time of applications. This was introduced in `Java 8`. To enable CDS there was an extra command requried to be run.

```
java -Xshare:dump
```

Starting `Java 12` CDS is enabled by default and there is no need to explicitly enable it.

## JEP 344: Abortable Mixed Collections for G1

This change is targeted to meet `G1` garbage collector pause time, supplied by the user. Idea here is to split the collection set for collecting, into two parts

- Mandatory part
- Optional part

If time taken to collect Mandatory part exceeds the pause time, `G1` ignores the optional part.

## JEP 346: Promptly Return Unused Committed Memory from G1

This is performance enhancement change. If `G1` notices very little application activity, it releases unused Java heap Memory automatically.

## New Api changes

### Compact Number Formatting

Java 12 supports human readable formatting of number for eg. 1000 as 1K

```java
import java.text.NumberFormat;
import java.util.Locale;
.
.
NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
String result = fmt.format(1000); // result = 1K
result = fmt.format(1100); // result = 1K
```

By default decimal places are ignored. We can enable decimal places as shown below :

```java
fmt.setMaximumFractionDigits(1);
result = fmt.format(1100); // result = 1.1K

// Long format shows full words of unit eg. million, thousand.
NumberFormat fm2 = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.LONG);
result = fmt2.format(1100); // result ="1 thousand"

```

### Files.mismatch()

New method in `java.nio.file.Files` class.

```java
public static long mismatch​(Path path, Path path2)
```

Returns value of `-1` signifies no mismatch, ie. equal files.

Two files are considered to match if following conditions are met:

- The two paths are of the same file, even if the target files does not exist
- The two files are the same size, and every byte in the first file is identical to the corresponding byte in the second file.

If above conditions are not met, then there is a mismatch.
Method will return :

- The position of the first mismatched byte (0 based index).
- Or, the size of smaller file, if all bytes of smaller file are equal to the corresponding byte of larger file.

```java
Path p = Paths.get("/home/vik");
Path p2 = Paths.get("/home/vik");
long result = Files.mismatch(p,p2); // result = -1 : same path

Path p = Paths.get("/home/test1");
Path p2 = Paths.get("/home/test2");
long result = Files.mismatch(p,p2); // 0 : First byte is mismatched
```

### New Teeing Collector

Teeing Collector is the new collector in Streams Api in `Java 12`.

Its full and correct prototype is

```java
public static <T,​R1,​R2,​R> Collector<T,​?,​R> teeing​(
                                          Collector<? super T,​?,​R1> downstream1,
                                          Collector<? super T,​?,​R2> downstream2,
                                          BiFunction<? super R1,​? super R2,​R> merger
                                          )
```

In simplified form you can think of it as below.

```java
Collector teeing​ (Collector downstream1,
                  Collector downstream2,
                  BiFunction merger);
```

Teeing collector takes 2 collectors i.e downstream1 and downstream2 in input.
Output of downstream1 and downstream2 into `BiFunction` merger and finally returns a Collector.

Lets see an example

```java
 double avg = Stream.of(10,20,30).collect(
                                    Collectors.teeing(
                                      Collectors.summingDouble(number->number),
                                      Collectors.counting(),
                                      (total,count) -> total/count
                                    )
                                  )
// avg is 20;
```

### New String methods

#### String indent()

```java
public String indent​(int n)
```

This method is used for adjusting indentation of String and normalizing the line termination character.

if n>0, then n spaces are added to the beginning of string.
if n<0, then up to n whitespace characters are removed from beginning of String
if n=0 then indentation of string is not changed.

Whatever the value of n, line terminator is always normalized.

```java
String str = "vik";  // str = "vik"
str.indent(5);       // str = "      vik\n"

str = " vik";        // str = " vik"
str.indent(-5)       // str = "vik\n"

```

#### transform​()

```java
public <R> R transform​(Function<? super String,​? extends R> f)
```

This function transforms the string using the supplied function

Below example transform a string to its length

```java
"vik".transform(s->s.length()); // returns 3
```

#### describeConstable()

```java
public Optional<String> describeConstable()
```

Returns an Optional containing the nominal descriptor for this instance, which is the instance itself.

```java
String test = "This is a lovely day";
test.describeConstable(); // returns  Optional[This is a lovely day]
```

#### resolveConstantDesc​()

```java
public String resolveConstantDesc​(MethodHandles.Lookup lookup)
```

Resolves this instance as a ConstantDesc, the result of which is the instance itself.

```java
String test = "This is a lovely day";
test.resolveConstantDesc(null);  // returns "This is a lovely day"

```
