# Flask Traditional Deployments: Zero to One &lt;Explore HTTPS&gt;

> The following article wil introduce how to config HTTPS servers.

> Author: v1siuol
>
> Last-Modified: 2019.09.05

### Background 

- This series is a supplyment for *Tradition Deploments* in *Chapter 17. Deployment* in *[Flask Web Development, 2nd Edition][url_flask_web_dev]* written by *Miguel Grinberg*. 
- Project address: https://github.com/v1siuol/v1siuol-site

### Steps

```bash
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot

$ sudo certbot certonly --webroot -w /var/www/example -d example.com -d www.example.com
```



`/var/www/example` should be the root path under `location ~ /.well-known` .

`example.com` and `www.example.com` should be your domain name.

> Saving debug log to /var/log/letsencrypt/letsencrypt.log
>
> Plugins selected: Authenticator webroot, Installer None
>
> Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org
>
> Obtaining a new certificate
>
> Performing the following challenges:
>
> http-01 challenge for v1siuol.com
>
> http-01 challenge for www.v1siuol.com
>
> Using the webroot path /home/ubuntu/flask/v1siuol-site/veri for all unmatched domains.
>
> Waiting for verification...
>
> Cleaning up challenges
>
>
>
> IMPORTANT NOTES:
>
>  \- Congratulations! Your certificate and chain have been saved at:
>
>    /etc/letsencrypt/live/v1siuol.com/fullchain.pem
>
>    Your key file has been saved at:
>
>    /etc/letsencrypt/live/v1siuol.com/privkey.pem
>
>    Your cert will expire on 2018-11-17. To obtain a new or tweaked
>
>    version of this certificate in the future, simply run certbot
>
>    again. To non-interactively renew *all* of your certificates, run
>
>    "certbot renew"
>
>  \- If you like Certbot, please consider supporting our work by:
>
>
>
>    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
>
>    Donating to EFF:                    https://eff.org/donate-le

As mentioned, the pem files are stored at `/etc/letsencrypt/live/v1siuol.com/fullchain.pem` and `/etc/letsencrypt/live/v1siuol.com/privkey.pem` .



Make SSL more stronger

```bash
$ openssl dhparam -out /path/to/dhparam.pem 2048
```



Now, modified your nginx config to fit into https. 

In `nginx_df` :

```
upstream uwsgicluster {
  server unix:///home/ubuntu/flask/v1siuol-site/logs/uwsgi.sock;
}

server {
    listen 80;
    server_name v1siuol.com
                www.v1siuol.com
                ;
    charset       utf-8;

    location ~ /.well-known {
        root /home/ubuntu/flask/v1siuol-site/veri;
    }
    location / {
        return 301 https://www.v1siuol.com$request_uri;
    }
}

server {
    listen  443 ssl;
    server_name v1siuol.com
                www.v1siuol.com
                ;

    charset       utf-8;

    ssl_certificate /etc/letsencrypt/live/v1siuol.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/v1siuol.com/privkey.pem;
    ssl_dhparam /home/ubuntu/flask/v1siuol-site/veri/dhparam.pem;

    # 减少点击劫持 Secure Nginx from Clickjacking  
    add_header X-Frame-Options DENY;
    # 禁止服务器自动解析资源类型 
    # when serving user-supplied content, include a X-Content-Type-Options: nosniff header along with the Content-Type: header,
    # to disable content-type sniffing on some browsers.
    # https://www.owasp.org/index.php/List_of_useful_HTTP_headers
    # currently suppoorted in IE > 8 http://blogs.msdn.com/b/ie/archive/2008/09/02/ie8-security-part-vi-beta-2-update.aspx
    # http://msdn.microsoft.com/en-us/library/ie/gg622941(v=vs.85).aspx
    # 'soon' on Firefox https://bugzilla.mozilla.org/show_bug.cgi?id=471020
    add_header X-Content-Type-Options nosniff;

    # 防XSS攻击
    # This header enables the Cross-site scripting (XSS) filter built into most recent web browsers.
    # It's usually enabled by default anyway, so the role of this header is to re-enable the filter for 
    # this particular website if it was disabled by the user.
    # https://www.owasp.org/index.php/List_of_useful_HTTP_headers
    add_header X-Xss-Protection 1;

    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:!DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_stapling on;
    ssl_stapling_verify on;

    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    add_header Strict-Transport-Security max-age=15768000;
    keepalive_timeout    70;

    root  /home/ubuntu/react/v1siuol-site-front-end/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /files {
        alias /home/ubuntu/flask/v1siuol-site/app/files;
    }

    location /api {
        include      uwsgi_params;
        uwsgi_pass   uwsgicluster;
    }
}
```



Thank you. 



Reference: 

- https://uwsgi-docs.readthedocs.io/en/latest/Nginx.html
- https://certbot.eff.org/

