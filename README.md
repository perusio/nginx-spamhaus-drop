# Nginx configuration for using the Spamhaus DROP list

## Introduction

This is a simple shell script that downloads the Spamhaus
[DROP and EDROP](http://www.spamhaus.org/drop/) lists and creates a
file that is used by Nginx
[`geo`](http://nginx.org/en/docs/http/ngx_http_geo_module.html#geo)
directive.

It's up to you to handle the **reloading** of the configuration so that
new netblocks are taken in consideration. The script obeys the lists
**expiration** date. So when you run the script new lists are
downloaded if the expiration has already passed.

The `geo` directive is only allowed at the `http` level context. So
you have to include the file `nginx/nginx_spamhaus_drop_list.conf` at
that level:

    http {
    
        ...
    
        ## Include the Spamhaus DROP lists.
        include nginx_spamhaus_drop_list.conf;
    
        ...
    }
    
If using a debian like architecture where there's a `/etc/nginx` the
`nginx_spamhaus_drop_list.conf` file should be in `/etc/nginx`. As
per the file with the list I recommend that you place it also
there. This way only a privileged user can update the list.

## Installation

