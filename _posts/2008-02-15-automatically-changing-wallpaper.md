---
title: "Automatically changing wallpaper relatively to daytime in Ubuntu"
description: "How to change wallpaper automatically"
layout: "post"
comments: true
date: "2008-02-15 15:24:00"
updated: "2012-12-19 02:49:40"
categories: [bash, Linux, cron]
---
Fedora 8 introduced a nice feature which is automatically changing wallpaper relatively to daytime.

Here's how to have this cool feature in Ubuntu (and any other linux distribution)  Select four wallpapers, one to be displayed at sunrise, the second during the day, third at sunset and the last at night. Then, in a terminal, create a bash script as follows :

{% highlight bash %}
gedit .change.sh
{% endhighlight %}

You'll get the newly created .change.sh file opened in a text editor. Copy the following text in the file (Change the PATH-TO-**-PICTURE with the appropriate path) :

{% highlight bash %}
#!/bin/bash
HOUR=$(date +%H)
case "$HOUR" in
04|05|06|07)
gconftool -t string -s /desktop/gnome/background/picture_filename PATH-TO-SUNRISE-PICTURE
;;
08|09|10|11|12|13|14|15)
gconftool -t string -s /desktop/gnome/background/picture_filename PATH-TO-DAY-PICTURE
;;
16|17|18)
gconftool -t string -s /desktop/gnome/background/picture_filename PATH-TO-SUNSET-PICTURE
;;
*)
gconftool -t string -s /desktop/gnome/background/picture_filename PATH-TO-NIGHT-PICTURE
;;
esac
{% endhighlight %}

As you can tell, this script sets the appropriate picture depending on the system's current hour. Now we'll create a cron job, which is a way to automatically run tasks.

{% highlight bash %}
gedit .change.cron
{% endhighlight %}

This will open the text editor to edit our cron job. Copy the following in the text editor (Replace YOUR-HOME-FOLDER by your home folder's absolute path):

{% highlight bash %}
* 4,8,16,19 * * * YOUR-HOME-FOLDER/.change.sh
{% endhighlight %}

As you see, the cron task will launch our previously created script at 4am, 8am, 4pm and 7pm to set the adequate wallpaper. (Of course, you can change the hours at your convenience).

Now, let's add these tasks to our gnome session so that they are automatically launched every time we login.

Open the System&gt;Preferences&gt;Sessions menu:

* Click the Add Button
- Name: Changing Wallpaper Cron
- Command: crontab PATH-TO-YOUR-HOME-FOLDER/.change.cron
* Click Ok

Now the job of automatically changing wallpaper is set as an automated job every time we login.

We also have to add the script at session startup so that the correct wallpaper is initialized when we login. (This means, that if you login at e.g. 5:30pm, you'll have the right wallpaper for that time). To do so:

* Click again the Add Button
- Name: Initializing Wallpaper
- Command: PATH-TO-YOUR-HOME-FOLDER/.change.sh
* Click Ok

Here you are ! Your desktop wallpaper is living and reflecting the daytime! :P

Feel free to send your comments if you have any issue.