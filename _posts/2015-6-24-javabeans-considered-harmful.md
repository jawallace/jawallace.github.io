---
layout: base
active: blog
title: JavaBeans Considered Harmful
---
JavaBeans Considered Harmful
============================
Since their introduction by Oracle in 1997, JavaBeans have been a common idiom of Java programming, seeing widespread adoption in many popular web and persistence framework, surviving 18 years of evolution and improvement of programming paradigms. Although JavaBeans' longevity and proliferation would seem to be an indication of the standard's robustness and usefulness, this is, in my opinion, not at all the case. Instead, I believe extensively utilizing JavaBeans reduces programmers' productivity and the quality and maintainability of the code. I hope the following discussion will educate readers who are considering using JavaBeans or popular web frameworks, such as Spring or Struts, on potential problems they may face because of JavaBeans. While I concede that it remains impractical to completely avoid JavaBeans (especially in a web server context), knowing the weaknesses of the standard can assist programmers to write safe, quality code.

What are JavaBeans?
-------------------
First it is important to understand the JavaBean specification and how it originated. At its simplest, the standard defines a JavaBean (or just a Bean) as a Java class that:

* Exposes a public constructor that takes no arguments
* Provides properly named public getters and setters for each property of the class
* Implements the java.io.seriazable interface
* Contains only other JavaBeans or primitive types as properties

Additionally, the specification provides an API for operations over JavaBeans as well as a simple event framework. 

Oracle introduced the JavaBeans standard to facilitate the construction of highly customizable (usually) visible components, generally by graphical tools but possibly by a scripting language or other programmatic methods. Accordingly, Sun designed JavaBeans as "reusable software component[s] that can be manipulated visually in a builder tool.”  Despite the original intention of the specification, many frameworks adopted JavaBeans in different contexts; therefore imposing flawed programming standards on the projects that use them.

The Use of JavaBeans
--------------------
Several large Java programming frameworks, notably server side web frameworks such as Spring  and Apache Struts , rely heavily on the use of JavaBeans. Both Spring and Struts encourage developers to design request and response objects as Beans for ease of serialization to and from JSON, XML, or other data transfer languages. Because data transfer objects frequently include domain model objects, this dictates that the domain model objects must also conform the JavaBean standard. Furthermore, the framework usually manages the instantiation of web resource controllers and additional services. The framework usually requires these objects to be Beans whose properties are configured in XML files. 

At this point, these frameworks have dictated that the following classes of objects subject to the JavaBean specification:

* Domain Model Objects
* Data Transfer Objects
* Services and Controllers

It is important to note that not a single one of these classes of objects can be "manipulated visually" like Oracle intended. Finally, considering the popularity of Java as a web server programming language and the high percentage of Java developers who choose Spring as their web framework of choice (40%) , it is evident that a significant portion of JavaBean use has diverged from Oracle's original vision in 1997.

The Dangers of JavaBeans
------------------------
Extensive use of JavaBeans diminishes code quality and reduces programmer productivity. A discussion of some of the dangers of JavaBeans and their use in popular web frameworks follows.

### Construction of Invalid Objects
All JavaBeans must provide a public constructor with no arguments. This immediately allows for the construction of semantically invalid objects. For example, imagine a simple class representing a Person. The class could have a few simple fields: a String name and an integer age. Calling the default constructor will result in a Person object that has a null name and an age of 0. Does this represent a real, valid person? The answer is, of course, no. One could argue that providing an additional constructor that must take all required fields, which the JavaBean specification allows, solves this. While this does allow the programmer to construct valid objects, it requires that he or she decide to use the correct constructor. Although proper documentation should eliminate possible confusion, it still allows the developer the ability to make a mistake and construct an invalid object, either because he or she did not read the documentation, or he or she mistakenly believed the default constructor would result in a valid object. Removing the ability to construct invalid objects removes the possibility of programmer error. 

### Multi-Step Construction
The requirement of a default constructor provides additional headache. When instantiating an object by calling the default constructor, constructing the object becomes a multi-step process. This can cause several problems. 

First, it introduces extra code that the programmer has to digest and understand. The additional code, although usually trivial, pollutes the visible instructions and, as a result, the programmer has more lines of code to look through, which can negatively impact productivity. 

Next, multi-step construction provides another way to create an invalid object. After calling the initial constructor, the programmer must remember to call the setter for each property of the Bean. If he or she misses even a single setter the object could be left in an invalid state. 

Finally, it removes the ability of the class to self-validate. When construction of the object is done via a single function call, the class can check the validity of the new object upon completion of the function. If the class wants to self-validate in a multi-step construction, it must provide an additional interface to allow the programmer to signal completion. The class must now not only provide an interface unrelated to the class's behavior, but also rely on the programmer to correctly call the verification routine. Furthermore, this results yet another instruction that will pollute the visible code and yet another avenue for programmer error. 

### Introduction of Mutability
Recently, the developer community has discussed the dangers of mutation and has concluded that while mutation has benefits in some situations, immutability should be favored . The specification requires JavaBeans to expose public setters; therefore JavaBeans are mutable. 

Unlike with immutable objects, which benefit from inherent thread safety, programmers must carefully handle mutable objects in multithreaded contexts to avoid race conditions. Considering the widespread use of Beans in web servers, which are without exception multithreaded, this seems like a horrible idea. Most web frameworks avoid race conditions by instantiating new controllers and services for each incoming request (and therefore for each thread). Therefore, frameworks waste memory and CPU resources simply to avoid race conditions due to the mutable nature of JavaBeans. Despite the large amount of memory available and Java’s garbage collection system, programmers should strive to conserve these resources to increase performance. Utilizing immutable services and controllers minimizes the consumption of memory and CPU, but unfortunately popular server frameworks do not easily allow for immutable objects.

Finally, the mutability of JavaBeans offers yet another avenue to create an invalid object. After an object is completely constructed, Beans allow the programmer to change their properties. Although some situations necessitate the use of this mutability, many Beans should remain static after construction and their mutability allows programmers to make an error and put a Bean back into an invalid state.

### Lack of Tooling Support
Finally, the use of JavaBeans in modern web frameworks removes many benefits of the tools Java programmers have come to rely on, such as IDE error highlighting and automatic refactoring. As stated previously, the framework often directly constructs JavaBeans as configured by an XML file. However, IDEs, even when configured with framework-specific plugins, cannot parse and understand this XML file. This can be massively detrimental to programmer productivity. For example, if a programmer adds an additional field to an existing JavaBean class, or adds a new JavaBean class, he or she must find and edit the XML configuration manually. Although most IDEs can check the syntactic validity of the XML, they cannot validate the configuration parameters added by the programmers in the context of the web application. Simple unit tests will easily catch any configuration problems, but even in a moderately sized project, catching a configuration problem, fixing it, and re-running the tests can result in 5-10 wasted minutes. Given the number of times a programmer can encounter this situation each day, this wasted time adds up. However, if a programmer constructs the objects, or configures the web framework programmatically rather than in XML, IDEs can immediately indicate if an error exists before compilation or testing, reclaiming these lost minutes.

The Alternative
---------------
Eschewing the standard and designing classes that follow a few simple guidelines can easily avoid the dangers of JavaBeans. 

First, design immutable classes unless a specific requirement necessitates mutability. Immutable classes provide thread safety by default and ensure the validity of instantiated objects.
Next, ensure that all methods of construction result in valid objects. This means that each construction method takes all necessary fields and validates the arguments. Additionally, fail fast by throwing an IllegalArgumentException rather than construct an invalid immutable object. If there are many parameters to construction, consider utilizing the Builder pattern, which provides an alternative to constructing complex immutable objects with one massive constructor. 

Finally, configure web application frameworks such as Spring and Struts to minimize the utilization of JavaBeans. Although it is not the preferred way according to the documentation, it is possible to use immutable objects rather than JavaBeans in Spring and Struts. An subsequent post will provide an example of a Spring application that minimizes the use of JavaBeans.

Thanks for reading! Do you agree that JavaBeans should be avoided? Is there a situation in which JavaBeans are the correct choice? Let me know in the comments below.

