---
layout: posts
title: Java 13 Release Features
permalink: /posts/java-13-release-features
published: true
---

Ever since Oracle took over the long-term advancement of the Java programming language, Java has been upgraded on a semi-annual release schedule.  Tuesday, September 17th marks the official release date of Java 13.  This release is not LTS (Long-Term Support); many of its features lay the groundwork for larger transitions in the languages long-term development, as seen between versions 8 and 12.  Additionally, it is worth mentioning this release falls under the OpenJDK.

# Useful Links
* [JDK 13 JEP Summary](https://openjdk.java.net/projects/jdk/13/)
* [JDK 13 Download Link](https://jdk.java.net/13/)
* [JDK 13 API Document](https://docs.oracle.com/en/java/javase/13/docs/api/index.html)
* [Comparing Java LTS Releases](https://blog.ippon.tech/comparing-java-lts-releases/)

# Feature Summary
Let's take a closer look at some of the features added to Java 13.  These features are broken out by JEP number (JDK Enhancement Proposals).  The JEP indexing system is similar to the more established [RFC protocal](https://www.rfc-editor.org/rfc-index.html) used industry-wide to define well-established definitions like TCP ([as well as some less known memos](https://en.wikipedia.org/wiki/April_Fools%27_Day_Request_for_Comments)).  The JEP index defines the roadmap for future Java releases, though it is worth mentioning not every JEP makes it into a release ([JEP 352](https://openjdk.java.net/jeps/352), postponed to Java 14).

We'll start this article with some of the sexier additions to Java in this release, and finish up with features more interesting to the die-hard Java developers out there.
## JEP 354 - Switch Expressions (Preview)
JEP 354 is my personal favorite because it is another leap forward in the quest to make Java more functional, and less procedural.  Traditionally, `switch` statements perform a sequence of operations based on the value held by the variable you are "switching" on.  Using the `case` keyword, developers can execute statements that are dependant on the value of the so-called "switch".  

This paradigm has always possessed a few crucial flaws.  The first being the required `break` statement at the end of each case.
Without the `break` statement, a logical switch becomes a sequence of statements.  For example,
```java
String foo = "foo";
switch(foo) {
    case "foo":
        foo = "I'm a foo."
    case "bar":
        foo = "I'm a bar."
}
System.out.println(foo);
```
will not print out "I'm a foo."  Instead, it prints out "I'm a bar." because there was no `break` statement halting the execution from going forward.  For large `switch` blocks, the extra and unnecessary `break` statements can quickly bloat your code.  

The other crucial flaw with traditional `switch` statements is scope is shared across every `case` statement in the block.  This means any variable declared in one `case` block exists in another `case` block.
```java
String foo = "foo";
boolean semaphore;
switch(foo) {
    case "foo":
        boolean result = true;
        semaphore = result;
    case "bar":
        System.out.println(result);
        boolean result = false;
        semaphore = result;
```
In this example, the first `case` block is executed.  The variable `result` is initialized, and it is used to assign a value to a boolean outside the switch's scope.  Then, the second `case` block is executed.  Technically, an error would be thrown because we are trying to create a variable `result` when one already exists.  But if the error was not thrown, the application would print "true" because the value of `result` is passed down from the first `case` statement.  To get around this, you would have to create a variable `result2`.  This creates additional unnecessary lines of code and those extra, hardly used variables create more overhead for the garbage collector.

In Java 13, the new `switch` statement does away with the `break` statement, and it creates an individual scope for each `case` clause.  It also introduces the `yield` keyword, which is a great way to return values from `switch` clauses.  The `yield` keyword also allows `switch` blocks to be used declaratively as statements.  Take the above example, if we were to rewrite it in Java 13, we would have the following:
```java
String foo = "foo";
boolean semaphore = switch(foo) {
    case "foo" -> yield true;
    case "bar", "foobar" -> yield false;
    default -> yield true;
};
System.out.println(semaphore);
```
The output of this snippet is "true," thanks to the `yield` keyword.  Without using any `break` statements and without creating additional variables, we've assigned a conditional value to another variable using syntax similar to a lambda or anonymous class, without actually creating a lambda or anonymous class.  This new design of the switch statement makes it a valuable tool for making readable code which is easy to extend and modify.

Some long-time Java developers may be hesitant to use a new keyword like `yield`, and for good reason.  The `yield` keyword is a hair's width away from the `return` keyword in terms of functionality.  They perform very similar tasks, the main difference is when you would use them.  To truly elucidate the reason for the `yield` keyword, let's rearrange the above example slightly:
```java
String foo = "foo";
boolean semaphore = switch(foo) {
    case "foo" -> true;
    case "bar" -> false;
    default -> {
        int sample = 100;
        yield true;
    }
};
```
The `default` case in the above example shows the situation where there are multiple steps to complete in this case.  To accommodate the passing of a value from the individual cases context back to the super-context of the `switch`, we must `yield` that value.  In the cases of `"foo"` and `"bar"` this step is unnecessary, as there is only one possible value to be yielded up to the super-context.
## JEP 355 - Text Blocks (Preview)
This feature of Java 13 is more cosmetic than any other feature in this release.  Effectively, it replaces line-breaks in Java via the `+` sign with simple carriage returns.  To utilize text blocks, you must initialize a string with three quotation mark symbols.  This makes text blocks like 
```java 
String text = "This is a former example\n" +
                " of multi-line text in\n" +
                " pre-Java 13 times."
```
capable of being re-written like this:
```java
String text = """
                This is a modern text block.
                There is no need for the newline character at all.
                We just add a new line and it is processed automatically.
                No more crazy escape sequences!
""";
```
This syntax is great for writing application code in other languages like Scala and passing it to an interpreter object within your application.  Another great use case is embedding formatted HTML in your Java application.  The rules for text blocks are just like string literals.  You can concatenate text blocks with String literals, you can insert variables into String literals, you can even insert the single quote " into text blocks without having to escape it; even escape sequences are supported if you wish to add them in!
## JEP 353 - Reimplement the Legacy Socket API
JEP 353 covers the new implementation of the Socket API.  To be more precise, an additional, more modern implementation of the Socket API (which was introduced in Java 1), has been added to the core libraries.  The old Socket API is available via a JDK system property and the `PlainSocketImpl` class (but not for long).

The first, and in my opinion most important feature of the new implementation, is that sockets are polled in non-blocking mode and are accessed via an operation limited by a timeout by default.  Why is this important?  Effectively, it means you can perform operations on a Socket object without having to wait for the Socket to respond before making an additional operation on said Socket.  Consider an application that relies on external APIs for a subset of its functionality.  If you start the application with a blocking Socket, you will have to wait for your Socket operations to complete before continuing your application.  But if you were to initiate a connection to an API service using a non-blocking socket and store the result in a Future<> object, you can continue your application's initialization steps without waiting for a response.  This is particularly convenient when decoupling your application's initialization and third-party dependencies.
## JEP 350 - Dynamic CDS Archives
This particular enhancement modifies the JRE more than anything; most Java developers will not notice this enhancement in their day to day development.  The origins of this JEP start in Java 10, with [JEP 310](https://openjdk.java.net/jeps/310).  Class-Data Sharing has been around since JDK 5, but it was codified as "Application Class-Data Sharing" for Java 10.  Effectively what happens is meta-data across a developer-specified list of classes are shared in an archive file.  This archive file is then loaded and referenced by the JVM.  If this archive file is built from 100 class files, you save the JVM a lot of time and energy by referencing just the one file.  This reduces memory footprint and application load times.

So what feature has been added to CDS in Java 13?  Java 13 introduces dynamic archiving.  Before Java 13, making a CDS archive required the completion of several steps.  As outlined in JEP 350, the steps are:
1. Do one or more trial runs to create a class list
2. Dump an archive using the created class list
3. Run with the archive
These steps are all achievable via java CLI options.  Depending on the size of the application, these commands may be difficult to run.  This is where Dynamic CDS comes in.  This enhancement performs step 1 at the end of your application's execution, let's say in your QA build process.  From here its just a matter of building your archive file and dropping it into a CI/CD pipeline for deployment.  The manual steps required to build the class list required to create the archive is completely gone with Dynamic CDS Archives.
## JEP 351 - ZGC: Uncommit Unused Memory
ZGC was a new garbage collector introduced to Java in Java 11.  It was designed to work on environments with massive compute capacity and memory requirements, significantly greater than desktop computing.  While other garbage collectors have their perks, ZGC is the best choice for applications with significant memory and compute requirements.  

ZGC is not without limits though.  Before Java 13, ZGC never returned memory to the OS without a restart.  This feature is common to most garbage collectors; most traditional GCs return memory to the OS by default as they were designed to run on commodity or even embedded hardware with severely restricted memory and compute requirements.  Not being able to return memory to the OS would have significant repercussions on application performance.  If we consider the origins of ZGC though, it is easy to see why this feature was not built-in.  If you have thousands of gigabytes of RAM installed on your host machine, you likely do not need to return it to the OS.  Regardless, this feature was added to ZGC in Java 13, making it an option for applications that do not always run on specialized, enterprise-level machines.
# Future Directions
Java 13 is the next chapter in the quest to make Java a viable functional language.  Since Oracle has taken over the development of Java, it has gradually evolved into a modern language, capable of keeping up with the hottest trends in application development.  Since Java 8, the language has steadily evolved into a discrete, easily readable, easily extensible language capable of keeping up with other languages like Python, Scala, and Kotlin.  

At the time of this post's writing, Java 14 is scheduled to be released in March of 2020.  [JEP 352](https://openjdk.java.net/jeps/352) is the only addition scheduled for [Java 14](https://openjdk.java.net/projects/jdk/14/), but that last will likely grow to include several more JEPs from the index before the end of 2019.
