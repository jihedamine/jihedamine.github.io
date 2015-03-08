---
title: "Joshua Bloch, Chess and Scala"
description: "Analogies between chess and software development"
layout: post
comments: true
date: 2010-07-28
categories:
tags: [java, scala]
---
##Joshua Bloch
Joshua Bloch was a senior staff engineer at Sun Microsystems and an architect in the core Java platform group. He designed and implemented the Collections Framework and contributed to many other parts of the platform. He is now a Chief Java architect at Google. He holds a Ph.D. in computer science from Carnegie-Mellon University. During the defense of his dissertation, which was open to the public, he arranged for his mother to ask a long technical question that he answered flawlessly after saying, "Awww, Mom!". He responded to another prepared question with a rap song, backed by a recorded rhythm track played on a boom box concealed under the desk. After all these years, he didn't lose this entertaining and engaging attitude and you will probably laugh a time or two watching his talks.

I am currently reading Joshua Bloch's [Effective Java 2nd Edition](http://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683/ref=sr_1_1?ie=UTF8&amp;s=books&amp;qid=1280285741&amp;sr=8-1) and I'm enjoying it. Given his deep knowledge of the language and his top level and long experience, Joshua justifies the use of each item in the book with both very practical low level arguments and high level, design arguments. The eventual disadvantages and caveats of a decision, if any, are explained and we get out of every item with a "moral" that's widely applicable for Java programming and sometimes for other languages.

I think, the most valuable teaching of the book, beyond the principles to apply in everyday programming, is how to think. You benefit from Joshua's aptitude to assemble pieces, to weight choices and to learn lessons from bad decisions made in previous Java versions.

Beyond the knowledge, this guy is very skilled at communicating and engaging his audience. I saw three presentations of him that I recommend to everyone:

* [This one](http://www.youtube.com/watch?v=V1vQf4qyMXg) is about the book, Effective Java. Joshua begins with two nice puzzlers demonstrating the difference between what you _think_ something is and what it is _actually_. He goes on with Generics (a long section), Enums, Varags, Concurrency and Serialization (this one is nice too). The "dessert" is a final puzzler.
* [The second presentation](http://www.youtube.com/watch?v=RR1E5zO-eBo) is about GWT or "gwit" as they call it at Google. Very interactive session where Josh responds to the host and to attendees.
* [The third one](http://www.youtube.com/watch?v=aAb7hSCtvGw) gives a lot of precious advice on good API design.

The three talks are highly engaging. You don't get bored a single minute. At times, I stopped and repeated sections in order to better understand them. It is obvious that the guy knows what he is talking about. It's like getting advice on how to play soccer from Maradona (or Basketball from Michael Jordan if you're a North American reader)

##Chess analogy
Watching and reading Joshua Bloch made me think of an analogy with Chess Grandmasters. I think software development is similar to chess as it is at the same time a science and an art:

In chess, everybody knows how to move pieces, what are their values, what are the main openings and attacking strategies (pins, discovery attacks, etc). But the art resides in _how_ and _when_ you use these techniques. Even though the rules are the same, there is no chess game like the other. Experts develop knowledge from deep theory understanding and heavy practice. They have a global vision of the state of the art, they know the pitfalls, they can see where they're heading earlier than less skilled players. This experience combined with creativity produces "masterpieces".

The same holds true for software development. We all know how to declare a class, how to do inheritance, we know some design patterns, etc. But when people like Joshua Bloch or Martin Fowler explain their design decisions, it looks more like art than science. They address the low level issues like JVM behavior and hardware constraints while, at the same time, being careful about the higher level requirements like modularity, coherence and extensibility. They produce elegant and efficient solutions. They get the hole picture and they see the long term implications of their decisions.

Another similarity is that both Chess and Software Development are constantly reinventing themselves. The possibilities seem infinite and there are always new ways of doing things, using the same core principles.

##Scala language
The more I heard from Josh, the more I was eager to learn Scala. Not that he promotes it, but he points out a lot of bad decisions made through the long life of the Java language that sticked with it for backward compatibility reasons.

In an interview, he quoted someone saying that every seven years, it is a good idea to hit the reset button. The Java language is nearly 15 years old and it's really time to hit the reset button! So why Scala ?

* Scala runs on the JVM and is able to call Java libraries: The JVM is a very mature and performant platform and the Java ecosystem is very rich with a bunch of libraries for nearly everything. It would be a mess not to reuse these valuable assets.
* Scala is statically typed: In his talks, Joshua repeatedly stresses the benefits of catching errors as soon as possible. It is ugly to throw a runtime error and the sooner the fix comes, the better it is.
* Scala has an advanced typing system: Scala is able to do type inference, resulting in more readable and concise code, while getting the safety of statically typed languages.
* Scala has dedicated and efficient libraries for concurrent programming: Joshua talks about concurrency in Efficient Java. He acknowledges that nowadays, with multicore processors, concurrency is a crucial aspect. He criticizes shortcomings in the original java.util.concurrent library while praising the newer constructs like ConcurrentMap because they are higher level than the "synchronized" keyword, lifting the burden of synchronization management from the developer to the language. Scala actors are a very nice concurrency abstraction. Actually, they are one of the main assets of the language.
* Scala natively supports functional programming: Joshua Bloch is against adding closures to Java. He justifies his opinion saying that Java grew far more complex in Java 5 with generics and autoboxing and varags. If you worry about the interaction between all these features, you get an exponential complexity growth. Closures would add a structural type system to an already complex nominal type system. Scala has been designed from the start as a mixture of object oriented and functional paradigms, so the interaction between both is pretty well handled from the start.
* Scala has other advantages like allowing to write powerful DSLs (Domain Specific Languages).

So I feel that an aging Java has become less and less prone to innovation and that Scala has all it takes to fullfill developer needs in building robust, scalable and highly maintainable enterprise level applications for the years to come.

##Conclusion
It is very enjoyable and motivating to read and hear from proficient programmers. It makes me realize that I am lucky to have chosen a field that allows creativity and is an inexhaustible source of excitement.
