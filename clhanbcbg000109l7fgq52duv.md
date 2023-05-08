---
title: "13 Reasons To Move From Java 8 To Java 11"
datePublished: Sat May 16 2020 15:40:35 GMT+0000 (Coordinated Universal Time)
cuid: clhanbcbg000109l7fgq52duv
slug: java-11
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683215460061/f918794b-775d-410e-bd8e-04745f862fd1.png
tags: java, java8, java11

---

Despite Java 11 being released in September 2018, [64% of developers report that Java 8 remains the most often used release](https://snyk.io/blog/developers-dont-want-to-leave-java-8-as-64-hold-firm-on-their-preferred-release/).

A further 27% of people said that there are "no features you need in later versions".

In this blog post, I will go over why I think this is wrong by showing some of the new exciting features post Java 8!

# Changes to the Release Model

Before we get started, let's discuss briefly the changes to Java's release model.

In the past, Java hasn't had particularly fast release times. Java 6 came out in December 2006, Java 7 wasn't released until July 2011, and Java 8 wasn't released until March 2014! These were big bang changes and followed a waterfall pattern. If a feature didn't make it into the release you could be waiting quite a few years before you'd see it.

This is something Oracle has changed with Java 9 going forward. Releases now have a 6-month cadence, with a Long Term Support (LTS) release happening every 3 years. This is why you hear people talk about Java 11 rather than 9, 10, or 12. Java 11 was the first release with LTS since Java 8. The next release with LTS will be Java 17.

Whilst you probably won't migrate to the versions in between Java 11 and Java 17 (at least not on enterprise applications), you can definitely check out the JDKs in between and play around with the features!

# Java 9

## No More Java EE

JavaEE is no longer part of the core JDK. JavaEE included features such as:

* XML Bindings
    
* JPA
    
* Dependency Injection
    

This means that the JDK is smaller, and frees up Oracle to focus on features that improve the language, rather than the surrounding tools.

When you migrate from Java 8 to Java 11, most likely this will be your biggest pain point. You'll see errors such as `ClassNotDef` or `MethodNotFound`. There are ways to get around [this](https://www.jesperdj.com/2018/09/30/jaxb-on-java-9-10-11-and-beyond/) without making many changes, but it's still a pain.

So what has happened to all of the Java EE features?  
Eclipse is now handling them! JavaEE has been rebranded as Jakarta EE, and can be read about [here](https://jakarta.ee/). This is a good thing, as Eclipse is focusing on making Java a Cloud Native platform, something that Oracle couldn't keep up with.

## A Better HTTP Client

Not many people use the default `HttpURLConnection` API in Java. Most people will use Apache HttpClient, Jetty, or RestTemplate. This is because `HttpUrlConnection` isn't very user-friendly.

However, in Java 9, a new HTTP client was included which is much easier to use.

The new API has 3 core classes:

* `HttpRequest`
    
* `HttpClient`
    
* `HttpResponse`
    

The new HTTP client supports:

* both synchronous and asynchronous requests
    
* HTTP/2 and HTTP/1.1 compatibility
    

HTTP/2 is a big step up from HTTP/1.1, which was last released in 1997. HTTP/3 is getting more support now as well, so hopefully we should see support for this in Java in the future too.

Here's an example of setting up an HTTP client using HTTP2:

```java
HttpClient client = HttpClient.newBuilder()
      .version(Version.HTTP_2)
      .build();
```

and to send a request asynchronously:

```java
HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create("http://openjdk.java.net/"))
      .build();
client.sendAsync(request, BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println)
      .join();
```

Simple, right?

## Finally a REPL

REPL = **R**ead **E**val **P**rint **L**oop  
A lot of other languages have REPLs and now finally Java does too, with its REPL called jShell. If you don't know what a REPL is, Python is a good example. You can type `python` into your command prompt to start the language shell and execute each line you write.

The Java REPL is most useful for seeing the output of small amounts of code, or playing with a new feature.

Why is jShell easier than creating your own class and experimenting with an IDE?

* you don't have to write complete Java code
    
* You don't need to declare classes if you don't want to
    
* you don't need to include sem-colons
    
* You can define variables, methods, classes, imports, and expressions, and you can also completely overwrite any of these.
    

It's also really easy to use:

```bash
> jshell
|  Welcome to JShell -- Version 11
|  For an introduction type: /help intro
jshell> String myStr = "hello stack to basics!"
myStr ==> "hello stack to basics!"

jshell> myStr
myStr ==> "hello stack to basics!"

jshell> myStr + " How are you?"
$3 ==> "hello stack to basics! How are you?"

jshell> $3
$3 ==> "hello stack to basics! How are you?"
```

This article [here](https://developers.redhat.com/blog/2019/04/05/10-things-developers-need-to-know-about-jshell/) shows some of the things you can do with jShell.

## Emojis4J

Unicode 7 is here in Java, which means you can finally use emojis! (This feature is not really called Emojis4J, but I wish it was).  
We have to be careful here though. Lets look at an a exmaple:

```java
public class Emojis4J {
    public static void main(String[] args) {
        String bear = "🐻";
        char bearChar = bear.charAt(0);

        String aString = "a";
        char aChar = aString.charAt(0);

        System.out.println("bear: " + bear)
        System.out.println("bearChar: " + aChar);
        System.out.println("a: " + bearChar);
    }
}
```

This prints out:

```plaintext
bear: 🐻
bearChar: ?
a: a
```

The bear char didn't print properly. Why not?

This is because emojis are 5 digit hex numbers. Each hex number needs 4 bits, therefore a 5 digit hex number needs 5x4=20 bits, with the bear emoji represented by `\u1f43b`. A java character is 2 bytes=2x8 bits = 16 bits. Not big enough to hold an emoji!  
To get around this you can represent emojis with two 16 bit hexadecimal values. The bear is represented with the string `\ud83d\udc3b`.  
These are called surrogate pairs, and are used to solve the problem in languages where chars are not big enough to hold some unicode characters.  
We can convert the bear to 2 chars properly like so:

```java
public class Emojis4J {
    public static void main(String[] args) {
        String bear = "🐻";
        int bearCodepoint = bear.codePointAt(bear.offsetByCodePoints(0, 0));
        char[] bearSurrogates = {Character.highSurrogate(bearCodepoint),
                Character.lowSurrogate(bearCodepoint)};
        System.out.println("Value of bearSurrogates: " +
                String.valueOf(bearSurrogates));
    }
}
```

This prints:

```plaintext
Value of bearSurrogates: 🐻
```

That's better.

## Smarter Strings

In Java strings are stored as an array of chars, with each char taking 2 bytes. However, [research](https://openjdk.java.net/jeps/254) shows two things:

* strings are a major component of heap usage
    
* most String objects contain only Latin-1 characters.
    

Latin-1 characters only require 1 byte.

This feature adds an encoding flag to Strings which, based on the content of the string, will use either two bytes per character or one byte.

This can save you a lot of space!

## Factory methods for collections

Finally, java offers a standard way to instantiate collections.  
We also now have immutable collections which implement the Collection interface but don't implement any mutable methods.

```java
public class JavaCollections {
    public static void main(String[] args) {
        //Immutable Collections:
        // Empty List
        List.of();

        // Easy initialisation of list, array-based List 
        // implementation (if more than 2 elements, check out the docs )
        final List<String> immutableList = List.of("String1", "String2");
        // The line below would throw the exception: Exception in thread 
        // "main" java.lang.UnsupportedOperationException:
        // immutableList.add("String3");

        // Empty map
        Map.of();

        // Easy initialisation of map, array-based Map implementation
        Map.of("My key1", 1, "My Key2", 2);

        // Mutable Collections
        // Not great, this way might end up creating up to 3 arrays
        List<String> mutableList = new ArrayList<>(List.of("String1"));

        // This will work
        mutableList.add("String4");
    }
}
```

## Experimenting with Ahead-of-Time compilation

Another new feature of Java is Ahead-of-Time compilation.

To understand why this is an exciting addition, let's look at the way most JVMs roughly work:

* byte code is executed by the JVM
    
* The JVM will either:
    
    * interpret the byte code
        
    * compile it to machine code using what is known as the Just-In-Time (JIT) compiler
        
        * with HotSpot JVMs they'll only use JIT if some piece of code is used enough
            

This is great, but it can mean that for large programs the JIT can take a long time to "warm up", and code may never be compiled at all (so it will execute slower).

Ahead-of-Time compilation allows us to compile Java classes to native code prior to launching the JVM. This would mean that our program should run a lot faster, as there is no interpretation or compilation happening whilst the program is running.

As an example, let's try and compile the following class:

```java
public class MyClassForAOT {
    public static void main(String[] args) {
        System.out.println("Hello from StackToBasics");
    }
}
```

We can compile this using AOT like so:

```plaintext
>javac MyClassForAOT.java
>jaotc --output MyClassForAOT.so MyClassForAOT.class
>java -XX:AOTLibrary=./MyClassForAOT.so MyClassForAOT
Hello from StackToBasics
```

AOT compilation could be particularly useful for cloud service, especially when using AWS Lambdas or Google's Cloud Functions, where faster start-up time and having a smaller code footprint is important.

A lot more can be said on AOT compilation, but that shall be saved for a separate post 😃

# Java 10

## Finally, the var keyword

Type inference is now extended to the declaration of local variables with initialisers.

```java
public class TypeInference {

    public static void main(String[] args) {
        var hello = "Hello Stack to Basics!";
        // hello = 5; This will not work, hello's type has been inferred as string, it cannot be changed to int
        System.out.println(hello);
    }
}
```

A misconception is that type inference == dynamically typed. This is not true. Each variable still requires a type, it is just that the type can be inferred rather than explicitly stated. Java is still a statically-typed language.

Remember that type inference already existed in Java, when using lambdas. We can see below that we did not need to declare the type for `b`, rather it was inferred.

```java
int maxWeight = blocks.steam()
    .filter(b -> b.getColor() == BLUE)
    .mapToInt(Block::getWeight)
    .max();
```

# Dogfooding Their Own Language

Before Java 9, the JIT compiler was written in C++. Not just C++, but a specific dialect of C++, which made it difficult for developers to make improvements.

Now they've finally switched to a different language and they're using... you guessed it, Java.

This is still in the early experimental stages, but wouldn't it be great to have a JIT compiler that developers can improve?

## Var for Lambdas

Following on from local variable type inference, we can now use `var` in lambdas:

```java
(var s1, var  s2) -> s1 + s2
```

Which is equivalent to the following:

```java
(String s1, String s2) -> s1 + s2 //or
(s1, s2) -> s1 + s2
```

Why bother with `var` for lambdas? Well, we can now do things like this which we couldn't before (without explicitly declaring `s1` and `s2` as strings):

```java
(@Nonnull var s1, @Nullable var s2) -> s1 + s2
```

## Emojis4J2

There are 56 more emojis!  
You can now display the bitcoin symbol. How exciting.

## Launching Single-File Programs

You can now run single-file java programs without compiling them, with any arguments placed after the file name being treated as program arguments. As example, lets say we have the following class:

```java
public class Factorial {

    public static void main(String[] args) {
        if(args.length != 1) throw new RuntimeException("Please provide 1 number");

        int number = Integer.parseInt(args[0]);
        System.out.println("Answer: " + calculateFactorial(number));
    }

    public static int calculateFactorial(int number) {
        if(number < 0) throw new RuntimeException("number must be positive");

        if(number == 0) return 1;
        if(number == 1) return 1;
        if(number == 2) return 2;
        else return calculateFactorial(number-1) * number;
    }
}
```

Normally we would have to compile this to byte code and then run it, like so:

```plaintext
>javac Factorial.java
>java Factorial 5
Answer: 120
```

But now, we can skip out the first step:

```plaintext
>java Factorial.java 5
Answer: 120
```

## Epsilon, The Garbage Collector for AWS Lambda and Google Cloud Functions?

A new gabage collector has been added! ... Which doesn't collect any garbage.

It handles memory allocation but does not implement any actual memory reclamation mechanism. Once the heap is exhausted, the JVM will shut down.

The main usages of this will most likely be for:

* testing how your program uses memory
    
* (Extremely) short-lived jobs, such as lambdas.
    

# Conclusion

This wraps up some of my favourite features of Java 9 to 11. I'm excited for the future of Java, and it's great to see some well-requested features finally making it into the JDK. Hopefully, this blog convinces you to explore some of these new features and take the plunge to upgrade to Java 11 if you haven't already.

Happy Coding!