# symfony-deploy
The deploy of web application based on Symfony Standard Edition using Ansible playbook.

## Deploy
For deploy to local machine use:

``` bash
$ ansible-playbook ansible/deploy.yml -i ansible/hosts.ini -l local -kK
```

## Config
There is example of `Nginx` config, which automatically enable maintnance mode when custom `maintnance.html` file exists in root dir:

``` ApacheConf
server {
    listen  80;
    server_name example.local;
    root /var/www/example.local;

    location / {
        # try to serve file directly, fallback to app.php
        try_files $uri /app.php$is_args$args;
    }

    # DEV
    # This rule should only be placed on your development environment
    # In production, don't include this and don't deploy app_dev.php or config.php
    location ~ ^/(app_dev|config)\.php(/|$) {
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
    }
    
    # PROD
    location ~ ^/app\.php(/|$) {
        if (-f $document_root/maintenance.html) {
            return 503;
        }

        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
        # Prevents URIs that include the front controller. This will 404:
        # http://domain.tld/app.php/some-path
        # Remove the internal directive to allow URIs like this
        #internal;
    }

    error_page 503 @maintenance;
    location @maintenance {
        rewrite ^(.*)$ /maintenance.html break;
    }
    
    error_log /var/log/nginx/example.local.error.log;
    access_log /var/log/nginx/example.local.access.log;
}
```
