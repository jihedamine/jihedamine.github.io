---
title: "Specialization in object oriented languages"
description: "Specialization principles in OO"
layout: "post"
comments: true
date: "2010-04-15 21:40:00"
updated: "2012-12-18 02:02:29"
permalink: "specialization-in-object-oriented"
categories: [Java, design, oo]
---
In this post, we will briefly introduce Object Oriented fundamental concepts and talk about specialization in O.O. paradigm.

## Fundamental O.O. Concepts
Object Oriented programming had a lot of success in the last decades mainly because it is equivalent to the way humans think. In fact, people categorize and classify entities, associate a state to an entity and distinguish some operations this entity is able to do. Moreover, O.O. paradigm is adequate with software engineering requirements. It is evolutionary, reusable, expressive, easily maintainable...  The main concept in O.O. paradigm is the *object*.
It is a capsule that wraps *properties* and has an identity.
Properties can be *attributes*, *methods* or other things (virtual types...).
A *class* stores and describes properties for its *instances*.
It declares the attributes and implements the methods. The definition of a class has to maximize its reusability and extensibility.
Classes are organized according to a *hierarchy*.
Hierarchy introduces the notions of specialization / generalization and super / sub classes.
Therefore, classes have a modelizing purpose. They can also be used as naming spaces (e. g. System and Math classes in Java) and as units of compilation (a class definition is mapped to a compiled unit).

<u>Message sending:</u> is the way to invoke an object's property. The object is an instance of a class that declares the invoked attribute or method, or inherits it from one of its super classes.
Let A be a class and foo() a method declared in class A.
{% highlight java %}
A a = new A();
a.foo();
{% endhighlight %}
A is called the *fonreceiver*, foo is called the *message*.

## Specialization
Specialization relationship establishes the concepts of super classes and sub classes. A class can *inherit* the properties of the superclass it *specializes* and *extend* it by redefining the inherited properties and introducing new ones.

<u>Extensions:</u> A class extension is the set of its instances.
Instances of a class are instances of all of its super classes. Therefore, having C' a subclass of class C, the extension of C' is included in the extension of C. Extensions are said to be *covariant* (they vary in the same way as the specialization relationship does).

<u>Intensions:</u> A class intension is the set of its properties. A subclass inherits all the properties of its super classes and eventually adds its own properties. Therefore, having C' a subclass of class C, the intension of C is included in the intension of C'. Intensions are said to be *contravariant* (they vary in the opposite way as the specialization relationship)

<u>Local property:</u> is a property (attribute or method) that is *defined* in a class and that belongs to a global property. A local property can redefine a local property inherited from a super class.

<u>Global property:</u> is *introduced* in a class and regroups local properties.

<u>Late binding:</u> The behaviour of a method is determined by the dynamic type of the receiver at execution time.  In statically typed languages, the global property that will be used in this call is determined by the static type of the receiver a and by the message foo (its name, signature... depending on the language). In dynamically typed languages, the receiver isn't considered when selecting the global property. The local property corresponding to the selected global property will be the most specific one accordingly to the receiver's dynamic type.  If the global property is ambiguous (i.e. a class inherits from two classes that use two distinct global properties that have the same signature), a global property conflict happens.  If the global property designated by the message invocation isn't ambiguous, but more than one local property belonging to the selected global property are at the highest level of specificity accordingly to the receiver's dynamic type, a local property conflict happens.  Let A be a class, B be a subclass of A and foo() a method declared in class A and redefined in class B.

{% highlight java %}
A a;
a = new B();
a.foo();
{% endhighlight %}

Here, method foo() that was redefined in class B will be invoked because the dynamic type of the receiver is B. Dynamic type is the actual type of the object {% highlight java %}a = new B();{% endhighlight %} whereas its static type is the type specified when declaring the object {% highlight java %}A a;{% endhighlight %}
In this example, method foo is a global property introduced in class A. Class A defines a local property foo, belonging to the global property foo it introduced. Class B redefines the local property foo inherited from A. 

The call {% highlight java %}a.foo();{% endhighlight %} designates a global property foo. As both local properties defined in classes A and B belong to this global property foo, the most specific one according to the dynamic type of the receiver of the message will be chosen. In this case, instance a's dynamic type is B, so the local property foo defined in class B will be invoked.

## References
* [Ducournaud 2008] R. Ducournaud "Programmation par objets, les concepts fondamentaux", UFR des sciences, Université Montpellier II, 2008.

* [Privat 2006] J. Privat "De l'expressivité à l'efficacité. Une approche modulaire des langages à objet", Université Montpellier II, 2006.