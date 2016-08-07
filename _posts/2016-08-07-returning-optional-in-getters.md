---
title: "Returning an optional in getters"
description: "Why wrapping the returned object in an optional is better"
layout: post
comments: true
date: 2016-08-07
categories:
permalink: "returning-optional-in-getters"
tags: [java]
---
A practice I've used recently and found quite helpful is to wrap the return type of newly created getters inside an Optional.

With the pre-java 8 approach of not wrapping in Optional, we end up with verbose defensive code that contains a lot of nested if (xx != null), or worse, NPEs in production when some getter in the call chain returns a null, which can happen in a lot of scenarios.

As an example:
{% highlight java %}
class Trip {
   private TripDetails tripDetails;

   public TripDetails getTripDetails() {
      return tripDetails;
   }
}

class TripDetails {
   private Traveller traveller;

   public Traveller getTraveller() {
      return traveller;
   }
}

class Traveller {
   private String name;

   public String getName() {
      return name;
   }
}
{% endhighlight %}

Having a trip object, we would usually get the traveller name either by using defensive programming, or not doing checks for nulls:

The first option produces code that is clearly too verbose with three nested ifs that hide the business logic and clutter the code.
{% highlight java %}
String travellerName = null;
if(trip != null) {
   TripDetails tripDetails = trip.getTripDetails();
   if(tripDetails != null) {
      Traveller traveller = tripDetails.getTraveller();
      if(traveller != null) {
         travellerName = traveller.getName();
      }
   }
}
{% endhighlight %}

On the other hand, just forgetting to check for nulls would have the following code throw NPEs in production for some use cases:
{% highlight java %}
String travellerName = trip.getTripDetails()
      .getTraveller().getName();
{% endhighlight %}

The alternative is to wrap the return object inside an Optional, so the getters would become
{% highlight java %}
public Optional<TripDetails> getTripDetails() {
   return Optional.ofNullable(tripDetails);
}

public Optional<Traveller> getTraveller() {
   return Optional.ofNullable(traveller);
}

public Optional<String> getName() {
   return Optional.ofNullable(name);
}
{% endhighlight %}

That way we would have less verbose code that doesn't throw NPEs.

If it's okay that the travellerName is null, we'd write something like this:
{% highlight java %}
String travellerName =
      trip.getTripDetails()
            .flatMap(TripDetails::getTraveller)
            .flatMap(Traveller::getName)
            .orElse(null);
{% endhighlight %}

If we need to throw an exception when we don't have a travellerName, we'd use the orElseThrow method:
{% highlight java %}
String travellerName =
  trip.getTripDetails()
        .flatMap(TripDetails::getTraveller)
        .flatMap(Traveller::getName)
        .orElseThrow(() ->
            new RuntimeException("Traveller name can't be null"));
{% endhighlight %}

When using Optional, if any of the intermediate calls returns a null value, the following intermediate calls aren't executed, and only the terminal operation orElse / orElseGet / orElseThrow is executed, preventing an NPE exception while having clear and concise code.
