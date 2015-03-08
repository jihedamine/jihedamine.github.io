---
title: "Differences between two versions of the same latex document"
description: "How to compare two versions of a latex document"
layout: "post"
comments: true
date: "2010-04-15 21:41:00"
updated: "2012-12-17 19:59:23"
categories: [latex, latexdiff]
---
[Latexdiff](http://www.ctan.org/tex-archive/support/latexdiff/) is a free perl script that highlights what has been added and what has been deleted between two versions of a latex document.

The script is used as follows:

{% highlight bash %}
latexdiff oldversion.tex newversion.tex > diffversion.tex
{% endhighlight %}

oldversion.tex is the old version of the latex document, newversion.tex is the newer one and diffversion.tex is a generated version that uses some custom latex commands to highlight the differences between the two documents passed as parameters.

While preprocessing a document, Latexdiff script replaces the \^ escape character with a custom command \SUPERSCRIPT (or SUPERSCRIPTNB).

In postprocessing, it reverts back to the \^ character.

However, as I am writing a latex document in french, I didn't mean to use ^ to write a superscript but to write the circumflex accent.

For example, t\^{e}te meant tête, not t<sup>e</sup>te.

As a consequence, the generated diff document had errors everywhere  \^ was used to write circumflex accents.

To fix the issue, I simply commented, in the latexdiff script, the lines that switched between the \^ regular expression and the SUPERSCRIPT custom command.

The following preprocessing instructions were commented:

{% highlight bash %}
# Convert ^n into \SUPERSCRIPTNB{n} and ^{nnn} into \SUPERSCRIPT{nn}
s/\^([^{\\]|\\\w+)/\\SUPERSCRIPTNB{$1}/g ;
s/\^{($pat4)}/\\SUPERSCRIPT{$1}/g ;
{% endhighlight %}

And those postprocessing instructions were commented too:

{% highlight bash %}
# Convert \SUPERSCRIPTNB{n} into ^n and \SUPERSCRIPT{nn} into ^{nnn}
s/\\SUPERSCRIPTNB{($pat0)}/^$1/g ;
s/\\SUPERSCRIPT{($pat4)}/^{$1}/g ;
{% endhighlight %}

Thus if you want to generate the diff between two latex documents containing circumflex accents, you can still use latexdiff. Just comment the conflictual instructions.