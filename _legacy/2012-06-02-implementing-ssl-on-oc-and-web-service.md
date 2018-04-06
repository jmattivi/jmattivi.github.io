---
layout: single
title: Implementing SSL on the OC and Web Service
date: '2012-06-02T04:33:00.001-07:00'
modified_time: '2012-06-08T09:52:32.416-07:00'
toc: true
tags:
- https port
- ssl
- scorch
- System Center Orchestrator 2012
- runbook
- https
- workflows
- Orchestrator
- Orchestrator Web Service
- ssl redirection
- xml
- orchestrator.svc
- web service
- Orchestration Console
blogger_id: tag:blogger.com,1999:blog-603465820850819373.post-6316877823091652069
blogger_orig_url: http://jmattivi.blogspot.com/2012/06/implementing-ssl-on-oc-and-web-service.html
---

This post I'll explain how to setup SSL on the Orchestration Console (OC) and Web Service (WS) while also redirecting http traffic to https.  For these examples, I’ve setup the OC on port 443 and the WS on 8443.

Since the OC and WS are now hosted through IIS, all the settings will be set through IIS.  First off you need to setup the bindings on the sites for each like below.  I'll explain in a bit why we want to leave 80 enabled in the bindings.

### IIS Configuration
Orchestration Console Bindings
![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/bindings.jpg "Bindings")

Within the https port from Edit, you can upload and select the cert you wish to use on the server from the drop down list.
![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/cert.jpg "Cert")

Web Service Bindings
![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/wsbindings.jpg "WSBindings")

### HTTP Redirection
Now that the OC and WS are enabled for https, we need to setup redirection from port 80 over to port 443 on the Orchestration Console.  Based on what was previously done, we only need to do this for the OC since the WS is only enabled for port 8443.  For a great resource on the redirection, see this link that explains how to do this using the URL Rewrite tool – [http://www.jppinto.com/2010/03/automatically-redirect-http-requests-to-https-on-iis7-using-url-rewrite-2-0/](http://www.jppinto.com/2010/03/automatically-redirect-http-requests-to-https-on-iis7-using-url-rewrite-2-0/).

To have the redirection work successfully using the URL Rewrite tool, it’s necessary that the SSL settings are left at the default NOT to require SSL on the OC.  This allows the traffic to hit port 80 and then the redirection will kick in.  This is why port 80 is left on the bindings as well.

![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/ssl.jpg "SSL")

### Web Config Updates
Finally, we need to edit the OC’s web.config file with the new "https://" address and fqdn path of the WS.  You can also see at the bottom of the screenshot below where the URL Rewrite tool adds the redirection.

![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/webcon.jpg "webcon")

If you don't edit the OC’s web.config file, you'll get this error after opening/logging into the OC since it can’t find the web service.

![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/ocerror.jpg "ocerror")

Now when you browse to the Orchestration Console on port 80 using the server's host name, IIS will automatically redirect the connection over to https.

By default, the Orchestration Console and Web Service also have pass through authentication enabled.  If you use privileged accounts to perform administrative tasks besides the account you regularly login to Windows with, you’ll also want to setup Basic Authentication on both the OC and WS.  This way when you browse to the site, it will prompt for a username and password to login with.

![](/assets/images/2012-06-02-implementing-ssl-on-oc-and-web-service/auth.jpg "Auth")