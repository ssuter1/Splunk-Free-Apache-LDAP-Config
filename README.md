Unfortunately, Splunk Free does not allow for any type of authentication to the web interface.  Here's how to secure access via apache reverse proxy to Splunk Web.  

Add the following two lines to /opt/splunk/etc/system/local/web.conf.  I had to modify root_endpoint because of strange redirect issues after authentication via the reverse proxy.  
```
root_endpoint = /splunk
tools.proxy.on = True
```
