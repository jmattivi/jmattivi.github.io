---
layout: single
title: High Availability for Runbook Servers on Invoking Runbooks
date: '2012-06-02T04:33:00.002-07:00'
modified_time: '2012-06-08T09:52:51.150-07:00'
toc: true
tags:
- Variables
- invoke
- workflows
- Orchestrator
- scorch
- System Center Orchestrator 2012
- runbook
- start runbook
blogger_id: tag:blogger.com,1999:blog-603465820850819373.post-8419480516672226183
blogger_orig_url: http://jmattivi.blogspot.com/2012/06/high-availability-for-runbook-servers.html
---

[Original Post](http://jmattivi.blogspot.com/2012/06/high-availability-for-runbook-servers.html)

Here's just a quick tidbit for specifying runbook servers used in the Invoke Runbook standard activity.  It’s beneficial to keep the runbook server names stored in Global Variables.  This way you can insert the variable in the Invoke Runbook activity instead of hardcoding the server name.

In case a runbook server has issues and is powered down or unavailable, the invoke runbook activity will automatically start the invoked runbook on the next runbook server in line.  Another benefit is the ability to add/remove runbook servers and just update a variable than trying to find/replace every invoke runbook activity w/ the new server name.

You can also use the variables to group runbook servers based on their role in the environment.  So to load balance runbooks, depending on how many runbook servers you have (for this example we’ll say three internal).  You could specify three variables which have the following values.

### Variables
Primary:  myrunbookserver1;myrunbookserver2;myrunbookserver3
Secondary:  myrunbookserver2;myrunbookserver3;myrunbookserver1
Tertiary:  myrunbookserver3;myrunbookserver1;myrunbookserver2

### Examples
So for runbook servers interacting w/ servers on an internal domain you could use:

![](http://jmattivi.files.wordpress.com/2012/05/var1.jpg?w=1000 "var1")

So for runbook servers interacting w/ servers on a dmz domain you could use:

![](http://jmattivi.files.wordpress.com/2012/05/var2.jpg?w=1000 "var2")

Finally, subscribe to the variable in the Invoke Runbook activity.

![](http://jmattivi.files.wordpress.com/2012/05/invoke.jpg?w=1000 "invoke")