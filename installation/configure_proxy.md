# Configure a Proxy

It is sometimes useful to front Go with a proxy server. In this section, we give you some tips and examples on how to achieve this.

## GoCD with Apache

An example of how to configure Go with Apache is shown below.

**Assumptions:**

-   You have Apache with mod\_proxy installed
-   The Apache server sits on the same machine as the Go server (localhost)

```apache
Listen nnn.nnn.nnn.nnn:80
NameVirtualHost nnn.nnn.nnn.nnn:80

<VirtualHost nnn.nnn.nnn.nnn:80>
  ServerName go.yourdomain.com
  DocumentRoot /var/www/html

  ProxyPass         /  http://localhost:8081/
  ProxyPassReverse  /  http://localhost:8081/
  ProxyPreserveHost On
</VirtualHost>
```

If you're additionally using SSL (highly recommended), you may use the following snippet -

```apache
Listen nnn.nnn.nnn.nnn:80
NameVirtualHost nnn.nnn.nnn.nnn:80


<VirtualHost nnn.nnn.nnn.nnn:80>
  ServerName gocd.example.com

  # Redirect any http requests to https
  RewriteEngine On
  RewriteRule ^/(.*)$ https://%{SERVER_NAME}/$1 [R=permanent,L]
</VirtualHost>

<VirtualHost nnn.nnn.nnn.nnn:443>
  ServerName gocd.example.com

  # Proxy everything over to the GoCD server
  ProxyPass         /  http://localhost:8153/
  ProxyPassReverse  /  http://localhost:8153/
  ProxyPreserveHost On
  RequestHeader set X-Forwarded-Proto "https"
  
  <Location />
    Order allow,deny
    Allow from all
  </Location>

  # SSL configuration
  SSLEngine on

  SSLCertificateFile /etc/pki/tls/certs/gocd.example.com.pem
  SSLCertificateKeyFile /etc/pki/tls/private/gocd.example.com.key
  SSLCertificateChainFile /etc/pki/tls/certs/gocd.example.com.pem.chained.pem
</VirtualHost>
```

## GoCD with NGINX

```nginx
server {
  # Redirect any http requests to https
  listen         80;
  server_name    gocd.example.com;
  return 301     https://gocd.example.com$request_uri;
}

server {
  listen                    443 ssl;
  server_name               gocd.example.com;

  ssl_certificate           /etc/pki/tls/certs/gocd.example.com.chained.pem;
  ssl_certificate_key       /etc/pki/tls/private/gocd.example.com.key;

  # Proxy everything over to the GoCD server
  location / {
    proxy_set_header        Host            $host;
    proxy_set_header        X-Real-IP       $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;
  }
}
```

<a name="agents-and-custom-ssl-ports"></a>
## Agents and custom SSL ports

Keep in mind that the agents must still be able to connect to the SSL port of the server (8154 by default), bypassing the proxy. The Go server itself needs to terminate the TLS connections of the agents, because they each use TLS client certificates to authenticate themselves to the server. So you have a firewall between your agents and your server, you must allow incoming traffic on the Go server SSL port, not just on the proxy server SSL port.

The initial communication of the agent to the server happens over HTTP, and this can go via the proxy, but afterwards all traffic will go directly via a TLS connection to the Go server (in fact, configuring the agent with the the SSL port instead of the HTTP port of the server will give an error for this initial connection).

## Also see...

-   [Configure site URLs](../installation/configuring_server_details.md#configure-site-urls)
