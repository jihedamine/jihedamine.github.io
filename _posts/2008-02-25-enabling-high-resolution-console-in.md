---
title: "Enabling high resolution console in Ubuntu"
description: "How to get a high resolution boot console"
layout: "post"
comments: true
date: "2008-02-25 19:32:00"
updated: "2012-12-17 20:00:27"
categories: [usplash, Linux]
---
I personally don't like the usplash displayed at bootup/shutdown in Ubuntu, I rather prefer seeing console messages.  
However, the low resolution of the console make the displayed fonts too big which is a little bit ugly.

To enable high resolution console, open a terminal and type :
{% highlight bash %}
sudo gedit /etc/modprobe.d/blacklist-framebuffer
{% endhighlight %}

then comment the line :
{% highlight bash %}
blacklist vesafb
{% endhighlight %}

Save the modifications and close the text editor.

Now, still in a gnome terminal, type :
{% highlight bash %}
sudo gedit /etc/initramfs-tools/modules
{% endhighlight %}

At the end of the file, add these two separate lines :
{% highlight bash %}
fbcon
vesafb
{% endhighlight %}

Close the edited file and type :
{% highlight bash %}
sudo update-initramfs -u
{% endhighlight %}

Finally, open your menu.lst file :
{% highlight bash %}
sudo gedit /boot/grub/menu.lst
{% endhighlight %}

In the kernel line for ubuntu delete 'splash' (so you no longer have the usplash displayed) and add vga=791 (to have a 1024*768 resolution)

That's it ! One last tip, to have more messages displayed you can change 'quiet' to 'services' or to 'verbose' in the kernel line.
