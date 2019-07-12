Unfortunately, Splunk Free does not allow for any type of authentication to the web interface.  Here's how to secure access via apache reverse proxy to Splunk Web.  

Add the following two lines to /opt/splunk/etc/system/local/web.conf.  I had to modify root_endpoint because of strange redirect issues after authentication via the reverse proxy.  
```
root_endpoint = /splunk
tools.proxy.on = True
```

Now, create a virtualhost in apache.  This also enables TLS otherwise your LDAP creds will be sent in plaintext.  A few othber options enabled due to using self-signed certs.  For example: /etc/httpd/conf.d
```
<virtualhost *:443>
     ServerAdmin user@domain.com
     ServerAlias hostname.com
     SSLEngine on
     SSLProtocol all -SSLv2 -SSLv3
     SSLCipherSuite HIGH:3DES:!aNULL:!MD5:!SEED:!IDEA
     SSLCertificateFile /etc/httpd/ssl/apache-selfsigned.crt
     SSLCertificateKeyFile /etc/httpd/ssl/private/apache-selfsigned.key
     SSLProxyEngine on
     SSLProxyVerify none
     SSLProxyCheckPeerCN off
     SSLProxyCheckPeerName off
     SSLProxyCheckPeerExpire off
     ProxyPass /splunk https://127.0.0.1:8000/splunk
     ProxyPassReverse /splunk https://127.0.0.1:8000/splunk
</virtualhost>
<proxy https://127.0.0.1:8000/splunk/*>
     Order deny,allow
     Deny from all
     Allow from all
     AuthName "Secure Splunk Login"
     AuthType Basic
     AuthBasicProvider ldap
     AuthLDAPURL ldap://admin-01/dc=hostname,dc=com?uid?sub?(objectClass=*)
     Require ldap-filter objectClass=posixAccount
</proxy>
```
Feel free to modify "Require ldap-filter" to your own requirements.  
