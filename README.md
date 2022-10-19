# How to Configure Load Balancing with HAProxy

HAProxy is an open source software that functions as a load balancing and proxy for TCP and HTTP. Load balancing is a method for distributing traffic to multiple servers.

![out](https://user-images.githubusercontent.com/52296424/196805772-4074e317-d9aa-4106-b862-6c816298e8cb.jpg)

# 0.Equipment used
Equipment used in this tutorial:

    OS Ubuntu 18.04 LTS
    HAProxy
    Nginx web server
    PHP-FPM 7.2
    Node1: 10.130.127.167
    Node2: 10.130.128.35
    LoadBalancer: 128.199.187.215
    Domain: example.com
    Node1 and Node2 have already installed Nginx web server and PHP-FPM 7.2. Each node is made index.php file containing the text node1 and node2 as a test page to find out which pages are read from which node.

# 1.Install HAProxy
Update and install HAProxy.

    sudo apt update
    sudo apt install haproxy -y    

# 2.Configure HAProxy
Open configuration file of HAProxy.

    sudo vim /etc/haproxy/haproxy.cfg

Default file configuration haproxy.cfg.

    global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

    # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL). This list is from:
        #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
        # An alternative list with additional directives can be obtained from
        #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
        ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
        ssl-default-bind-options no-sslv3

    defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http
    
# Add a configuration for the HAProxy listener.
    
    frontend http_front
       bind *:80
       mode http
       default_backend http_back
       
       
# Add configuration for web server backend.   
    backend http_back    
        mode http
        balance roundrobin
        option forwardfor
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request add-header X-Forwarded-Proto https if { ssl_fc }
        option httpchk HEAD / HTTP/1.1rnHost:localhost
        server node1 10.130.127.167:80
        server node2 10.130.128.35:80
       
# Additional configuration for HAProxy statistics.

    listen stats 
        bind  *:1234
        stats enable
        stats hide-version
        stats refresh 30s
        stats show-node
        stats auth username:password
        stats uri /stats   
        
# The final result is HAProxy configuration.

    global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

    # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL). This list is from:
        #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
        # An alternative list with additional directives can be obtained from
        #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
        ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
        ssl-default-bind-options no-sslv3

    defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

    frontend http_front
       bind *:80
       mode http
       default_backend http_back

    backend http_back    
        mode http
        balance roundrobin
        option forwardfor
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request add-header X-Forwarded-Proto https if { ssl_fc }
        option httpchk HEAD / HTTP/1.1rnHost:localhost
        server node1 10.130.127.167:80
        server node2 10.130.128.35:80

    listen stats 
        bind  *:1234
        stats enable
        stats hide-version
        stats refresh 30s
        stats show-node
        stats auth username:password
        stats uri /stats   
   
# Verify the configuration and restart HAProxy.

    sudo haproxy -c -f /etc/haproxy/haproxy.cfg
    sudo systemctl restart haproxy
 
# 3.Testing
    Browse the domain, refresh the page repeatedly until it displays the index.php file of Node1 and Node2.

 ![02 cara-setting-load-balancing-dengan-haproxy_web-node1](https://user-images.githubusercontent.com/52296424/196808827-264d8240-b023-455b-8e4a-ac045d6a09e4.jpg)
 
                                                 index.php page from Node1
 
 ![03 cara-setting-load-balancing-dengan-haproxy_web-node2](https://user-images.githubusercontent.com/52296424/196808975-5fdb273f-75e8-4150-903f-91413d302358.jpg)

                                                 index.php page from Node2
 
 
# 4.Statistics
    Browse http://domain.com:1234/stats to read HAProxy statistics.
    
 ![04 cara-setting-load-balancing-dengan-haproxy_statistik-haproxy](https://user-images.githubusercontent.com/52296424/196809129-95b34fc2-8d6a-4cbe-af17-4c21dbd778db.jpg)



    https://youtu.be/idFwu2z5-zY

 
 
 
 
 
