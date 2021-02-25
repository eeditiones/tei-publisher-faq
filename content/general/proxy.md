+++
title = "Proxy and URL encoding"
date =  2021-02-25T17:15:41+01:00
tags = ["proxy", "api"]
+++

If eXist-db is hosted behind a proxy you might encounter issues with the API. Instead of the desired content you get "the server did not return any content" and several 404 errors on the console.  
You have to tell you proxy to avoid decoding the query prameters:
  - nginx
  ```
  location / {
    proxy_pass  http://host/webapp$request_uri;
  }
  ```
  - Apache
  
     Use 'nocanon' option with ProxyPass https://httpd.apache.org/docs/2.4/mod/mod_proxy.html#proxypass and 'AllowEncodedSlashes' directive http://httpd.apache.org/docs/2.2/mod/core.html#allowencodedslashes

See https://stackoverflow.com/questions/20496963/avoid-nginx-decoding-query-parameters-on-proxy-pass-equivalent-to-allowencodeds
