---
layout: post
title: Setting up apache, tomcat and railo (+ cfwheels)
published: true
date: 2011-1-26
categories: [coldfusion,railo,cfwheels]
uniqueid: setupapachetomcatrailo
---

[railo]: http://www.getrailo.org/ "Railo"
[tomcat]: http://tomcat.apache.org/ "Tomcat"
[apache]: http://httpd.apache.org/ "Apache"
[cfwheels]: http://cfwheels.org/ "CFWheels"

Due to some limitations I've recently encountered with Adobe's Coldfusion server, I decided it was time to make the switch to [Railo][railo] - at least for personal development. Railo has gained a lot of popularity in the past several years, and with so many top developers in the field using it, I thought I would try it out for awhile. I have been extremely impressed with the results. More on that later.

Railo offers several options for installing and running the server:

 * Railo Server with Tomcat
 * Railo Express with Jetty
 * Railo Custom

The [Tomcat][tomcat] package is meant for production use, the express edition for testing/development, and the custom package for unique setups (Different application server, etc). The custom package is either a WAR file that can be deployed on your J2EE server, or as Jar files that can be included in your classpath.

Getting the express install working was effortless. I literally had a functioning cfml server up in running in minutes (I had to account for the download time). The express install merely has a `startup` command that need to be run, and then you have a fully functional Railo install running on top of Jetty. This works great for quick demos or tests, but I wanted something a little more permanent, and something that would give me more flexibility in the future.

Install Tomcat
-------------------

During this writeup, Tomcat 7 was officially released, so I've modified the tutorial to work with the new version. To install tomcat, you simply need to extract the archive to the folder of your choice. For my purposes, I chose `/Library/tomcat`. In order to startup Tomcat, shell scripts are provided in the `bin` folder within the Tomcat install. Simply run `startup.sh` to start it up and `shutdown.sh` to shut it down. To verify that you install is working correctly, simply go to [http://localhost:8080](http://localhost:8080) To make things easier, I found a simple tomcat control script online, and modified it slightly:

{% highlight bash %}
#!/bin/sh
# Tomcat Startup Script

#Make sure that an icon doesn't show on the dock
export JAVA_OPTS=-Djava.awt.headless=true

start() {
        echo "Starting Tomcat"
        /Library/tomcat/bin/startup.sh
}
stop() {
        echo "Stopping Tomcat"
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
		sleep 5
        start
        ;;
  *)
        echo $"Usage: tomcat {start|stop|restart}"
        exit
esac
{% endhighlight %}

This let's us start, stop and restart tomcat from a single script. To make it even easier, I added an alias to my bash profile that let's me control tomcat easily on the command line: `alias tomcat=/Library/tomcat/bin/tomcat.sh`. Now I can simply call `tomcat {stop|start|restart}` anytime from the command line.

Proxy with Apache
--------------------

For my purposes, I decided that it would be easiest to use virtual hosts in apache and proxy the requests on to tomcat based on the server name. Depending on your setup, virtual hosts entries might be placed in different configuration files. In OS X, there is a file made specifically for listing virtual hosts found in: `/etc/apache2/extra/httpd-vhosts.conf`. Before we edit this file however, you need to make sure that the main `httpd.conf` file is including it. It is commented out by default in OS X. Look for this line:

{% highlight apache %}
Include /private/etc/apache2/extra/httpd-vhosts.conf
{% endhighlight %}

Make sure it is not commented out. Ok, now we are ready to edit the virtual hosts file. First you'll want to make sure that Apache knows we are using name based virtual hosting. Uncomment the follow line in the file:

{% highlight apache %}
NameVirtualHost *:80
{% endhighlight %}

Next, we need to make sure that we add a virtualhost for localhost. Make sure that this points to your default web root.

{% highlight apache %}
<VirtualHost *:80>
	ServerAdmin admin@localhost
	DocumentRoot "/Library/WebServer/Documents"
	ServerName localhost
	ErrorLog "/private/var/log/apache2/localhost-error_log"
	CustomLog "/private/var/log/apache2/localhost-access_log" common
</VirtualHost>
{% endhighlight %}

Now we can add a specific vhost for tomcat. This virtual host will proxy all requests on to tomcat.

{% highlight apache %}
<VirtualHost *:80>
    ServerAdmin admin@localhost
    DocumentRoot "/Library/tomcat/webapps/ROOT"
    ServerName tomcat.dev
    ErrorLog "/private/var/log/apache2/tomcat.dev-error_log"
    CustomLog "/private/var/log/apache2/tomcat.dev-access_log" common
	
	<LocationMatch "^[^/]">
		Deny from all
	</LocationMatch>
	
	ProxyPreserveHost On
	ProxyPass / ajp://localhost:8009/
	ProxyPassReverse / ajp://localhost:8009/
</VirtualHost>
{% endhighlight %}

NOTE: If you have any issues with Content-Type being changed to text/plain on your documents, you might need to change your proxy to go through http instead:

{% highlight apache %}
ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
{% endhighlight %}

I had this issue, and I'm not sure what the cause is. I think there is a problem with Tomcat 7 and the ajp connector since this seemed to be working fine under 6. You'll need to make sure that the servername you choose for the virtual host (e.g. `tomcat.dev`) exists in your hosts file as well. If everything worked out, you should be able to hit the site you setup, and see the old familiar Tomcat start page.

Install Railo
-------------------

Installing Railo is fairly straightforward. Download the JARs distribution of Railo from their download page, and simply add them to your classpath. The easiest way to do this is by dragging the jars directly into the `lib` folder inside the tomcat install.

Before you can actually start using Railo however, you need to add a servlet definition for Railo, and instruct it that it can handle Coldfusion files (e.g. *.cfm, *.cfc, etc). Open up `web.xml` located in the `conf` folder of tomcat. Underneath `web-app` add the following:

{% highlight xml %}
<!-- Railo Servlet -->
<servlet>
	<servlet-name>CFMLServlet</servlet-name>
	<servlet-class>railo.loader.servlet.CFMLServlet</servlet-class>
	   <init-param>
	      <param-name>configuration</param-name>
	      <param-value>{web-root-directory}/WEB-INF/railo/</param-value>
	      <description>Configuraton directory</description>
	   </init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
   <servlet-name>CFMLServlet</servlet-name>
   <url-pattern>*.cfm</url-pattern>
   <url-pattern>*.cfml</url-pattern>
   <url-pattern>*.cfc</url-pattern>
</servlet-mapping>
{% endhighlight %}

You'll also want to add an entry for coldfusion files in the welcome file list. You should see this at the bottom of `web.xml`. Simply add entries for both `index.cfm` and `index.cfml`. With that, you're done, and ready to test it out. Go to the domain you setup previously: [http://tomcat.dev/railo-context/admin/](http://tomcat.dev/railo-context/admin/) and you should see the Railo administrator. Set a password for the admin portal, and you can start adding coldfusion files to test it out!

Use CFWheels with URL Rewriting
-----------------------

The coldfusion framework I've been using recently is [CFWheels][cfwheels]. If you haven't checked it out before, I highly recommend it - I'll go into this more on another blog post... To start off with, we need to make a new virtual host that we'll use for our wheels application. Again, go into the apache vhosts config file, and add a new entry:

{% highlight apache %}
<VirtualHost *:80>
    ServerAdmin admin@daemon.local
    DocumentRoot "/Path/To/My/App"
    ServerName cfwheels.dev
    ErrorLog "/private/var/log/apache2/cfwheels.dev-error_log"
    CustomLog "/private/var/log/apache2/cfwheels.dev-access_log" common
	
	<LocationMatch "^[^/]">
		Deny from all
	</LocationMatch>
	
	ProxyPreserveHost On
	ProxyPass / http://cfwheels.dev:8080/
	ProxyPassReverse / http://cfwheels.dev:8080/

	RewriteEngine On
	ProxyRequests Off
	RewriteCond %{REQUEST_URI} !^.*/(flex2gateway|jrunscripts|cfide|cfformgateway|railo-context|files|images|javascripts|miscellaneous|stylesheets|robots.txt|sitemap.xml|rewrite.cfm)($|/.*$) [NC]
	RewriteRule "^/(.*)" ajp://cfwheels.dev:8009/rewrite.cfm/$1 [P,QSA,L]
</VirtualHost>
{% endhighlight %}

The main difference here is that we've added the url rewriting of the cfwheels app right into the virtual host definition. Apache won't read the `.htaccess` that comes with cfwheels since we're just proxying the call. We also need to add a virtual host entry in tomcat as well. Open up `server.xml` in tomcat's `conf` folder, and add this below the localhost Host entry:

{% highlight xml %}
<Host name="cfwheels.dev" appBase="webapps"
	unpackWARs="true" autoDeploy="true"
	xmlValidation="false" xmlNamespaceAware="false">
	<Context path="" docBase="/Path/To/My/App"/>
</Host>
{% endhighlight %}

If you go to the index file, you should see the default wheels index page. However, if you try to go to any controllers, tomcat will give you a 404 error. This is because tomcat is not sending the `PATH_INFO` correctly. To fix this, you only need to add two more patterns to the servlet mapping of the CFMLServlet:

{% highlight xml %}
<servlet-mapping>
   <servlet-name>CFMLServlet</servlet-name>
   <url-pattern>*.cfm</url-pattern>
   <url-pattern>*.cfml</url-pattern>
   <url-pattern>*.cfc</url-pattern>
   <url-pattern>/index.cfm/*</url-pattern>
   <url-pattern>/rewrite.cfm/*</url-pattern>
</servlet-mapping>
{% endhighlight %}

Restart tomcat and now, everything should work perfectly - even the url rewriting!

I hope that someone can find this useful. I know there are a thousand different ways to set this up, but this is what worked out best for me.


