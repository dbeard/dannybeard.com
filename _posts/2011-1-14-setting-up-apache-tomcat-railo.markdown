---
layout: post
title: Setting up apache, tomcat and railo
published: true
categories: [coldfusion,railo]
---

[railo]: http://www.getrailo.org/ "Railo"
[tomcat]: http://tomcat.apache.org/ "Tomcat"
[apache]: http://httpd.apache.org/ "Apache"

Due to some limitations I've recently encountered with Adobe's Coldfusion server, I decided it was time to make the switch to [Railo][railo] - at least for personal development. Railo has gained a lot of popularity in the past several years, and with so many top developers in the field using it, I thought I would try it out for awhile. I have been extremely impressed with the results. More on that later.

Railo offers several options for installing and running the server:

 * Railo Server with Tomcat
 * Railo Express with Jetty
 * Railo Custom

The [Tomcat][tomcat] package is meant for production use, the express edition for testing/development, and the custom package for unique setups (Different application server, etc). The custom package is either WAR file that can be deployed on your application server, or as Jar files that can be included in your classpath.

Getting the express install working was effortless. I literally had a functioning cfml server up in running in minutes (I had to account for the download time). The express install merely has a `startup` command that need to be run, and then you have a fully functional Railo install running on top of Jetty. This works great for quick demos or tests, but I wanted something a little more permanent. Adobe's Coldfusion server integrates really well with Apache via the custom JRun Handler module they've created for Apache. I wanted something similar, essentially be able to drop `.cfc` or `.cfml` files into my web root, and be able to run them. After some research, I opted to use Tomcat, with mod_jk to integrate with Apache. This setup was done on my Macbook Pro running Snow Leopard, but should translate well to other systems.

Install Tomcat
-------------------

During this writeup, Tomcat 7 was officially released, so I've modified the tutorial to work with the new version. To install tomcat, you simply need to extract the archive to the folder of your choice. For my purposes, I chose `/Library/tomcat`. In order to startup Tomcat, shell scripts are provided in the `bin` folder within the Tomcat install. Simply run `startup.sh` to start it up and `shutdown.sh` to shut it down. To verify that you install is working correctly, simply go to [http://localhost:8080](http://localhost:8080) To make things easier, I found a simple tomcat control script online, and modified it slightly:

{% highlight bash %}
#!/bin/sh
# Tomcat Startup Script

start() {
        echo -n "Starting Tomcat:  "
        /Library/tomcat/startup.sh
        sleep 2
}
stop() {
        echo -n "Stopping Tomcat: "
        /Library/tomcat/bin/shutdown.sh
}

# See how we were called.
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart)
        stop
        start
        ;;
  *)
        echo $"Usage: tomcat {start|stop|restart}"
        exit
esac
{% endhighlight %}

This let's us start, stop and restart tomcat from a single script. To make it even easier, I added an alias to my bash profile that let's me control tomcat easily on the command line: `alias tomcat=/Library/tomcat/bin/tomcat.sh`. Now I can simply call `tomcat {stop|start|restart}` anytime from the command line.

Install Railo
--------------------

In order to install Railo, 

