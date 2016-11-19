---
title: "Linux shell on windows"
description: "How to get access to a linux shell from Windows"
layout: post
comments: true
date: 2016-10-02
categories:
permalink: "linux-shell-windows"
tags: [linux,vm]
---
Windows 10 Anniversary update brought a new feature called "bash on ubuntu on windows". This feature provides a basic bash environment that allows users to run common (basic) command line utilities such as grep, sed, awk via a ubuntu bash. 
However, the tool is very limited and far from being an alternative to a fully-fledged linux VM as it doesn't handle docker, Oracle Java or other significantly important tools for any developer.

On the other hand, the idea of accessing a bash terminal by simply running a windows app is a very nice one, as opposed to - for example - starting the VirtualBox app, selecting a VM, clicking on start, the VM itself boots a graphic environment to be able to have a high resolution fullscreen terminal, etc, etc.

To have the best of both worlds, i.e. a fully-fledged linux environment and a native, high resolution command line tool that seamlessly connects you to that environment, I used two free applications (<a href="https://www.virtualbox.org/" target="_blank">VirtualBox</a> and <a href="http://www.cmder.net" target="_blank">cmder</a>) and I made a handy batch script that checks if the linux VM is running, starts it if it isn't, then connects to it via ssh. All the end user does is start the command line tool and run the script and he gets a high resolution bash that is connected to a fully-fledged linux VM.


# Setup ssh to the VirtualBox VM

## Port forwarding
To establish an SSH connection from the host to the VM, we attach the network adapter to NAT in the VM network settings and, in port forwarding, we establish a TCP rule to forward connections to the windows host's port 2222 to the linux guest's port 22. That way, when we issue the following in our windows host {% highlight bash %}ssh jihedamine@127.0.0.1 -p 2222{% endhighlight %} it is forwarded to our linux VM as {% highlight bash %}ssh jihedamine@127.0.0.1 -p 22{% endhighlight %} Port 22 is the default port where openssh server listens to ssh connection requests.

## SSH key
If we don't want to be asked for our user password everytime we connect to the linux VM via ssh, we can generate an ssh key in our windows machine {% highlight bash %}ssh-keygen.exe{% endhighlight %} and append the content of the generated file ~\\.ssh\id_rsa.pub to the ~/.ssh/authorized_keys file in our linux VM.

# Create scripts to access to the VM
Now to the interesting part! 

## Start the VM and connect to it
run-ubuntu.bat:
{% highlight bash %}
@echo off 

for /f "tokens=\* usebackq" %%i in ( 
'"c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo Ubuntu ^| grep -c "running"' 
) do set var=%%i 

%var% > ubuntu.txt  

for /f "delims=" %%x in (ubuntu.txt) do set isUp=%%x  

if "%isUp%" == "0" ( 
"c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm Ubuntu --type headless 
) 

ssh jihedamine@127.0.0.1 -p 2222
{% endhighlight %}

## Stop the VM
stop-ubuntu.bat:
{% highlight bash %}
@echo off 

for /f "tokens=\* usebackq" %%i in ( 
'"c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo Ubuntu ^| grep -c "running"' 
) do set var=%%i 

%var% > ubuntu.txt  

for /f "delims=" %%x in (ubuntu.txt) do set isUp=%%x  

if not "%isUp%" == "0" ( 
"c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm Ubuntu poweroff 
)
{% endhighlight %}

# Advantages
In my opinion, this setup with cmder and a headless VirtualBox VM is a very nice way to get all the Unix tools power and a very powerful command line environment, via an application that runs natively on your Windows desktop environment (which is the OS that comes with most personal computers).

## How it looks
This video capture shows a cmder app running on windows, in which we execute the run-ubuntu.bat script. The script starts up a headless Ubuntu VM, or connects to the already started one. From there, interact with your Unix system as you wish.
![alt txt](/images/linux-shell-windows/cast.gif "Logo Title Text 1")


## Advantages over bash on ubuntu on windows
+ Full-blown Unix environment where you can run docker, postgresql and whatever app/tool that's available

## Advantages over using the VirtualBox GUI
+ Better integration with Windows OS (easy switching between cmder app and other windows apps like a running IntelliJ Idea instance, your chrome browser or your iTunes app.)
+ High resolution console (as opposed to the low resolution console displayed by VirtualBox when no XServer is running)
+ No separate desktop environment (as opposed to running Gnome/Kde/xfce on the VirtualBox VM)

# Conclusion
Bash on Ubuntu on Windows is a good feature, but it currently is a very limited tool. Running a Ubuntu desktop environment inside a VM manager on Windows isn't as seamless and straighforward as accessing your terminal app in a Linux or Mac OS. The cmder / VirtuablBox / Ubuntu setup uses free software, so feel free to test this config for yourself and see if it suits your needs.
