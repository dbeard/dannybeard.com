---
layout: post
title: Setting up apache, tomcat and railo
published: false
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

The [Tomcat][tomcat] package is meant for production use, the express edition for testing/development, and the custom package for unique setups (Different application server, etc). The custom package is either WAR file that can be deployed on your application server, or as Jar files that can be included in your lib folder.

Getting the express install working was effortless. I literally had a functioning cfml server up in running in minutes.