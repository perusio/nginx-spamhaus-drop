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
there. This way only a **privileged** user can update the list.

## Installation

 1. Clone the repository:
     
     git clone git://github.com/perusio/nginx-spamhaus-drop.git
   
 2. Invoke the script:
     
     ./nginx-drop-fetch
     
    This creates a file `drop_list.conf` in `/etc/nginx`. Note that
    you need to be **root** (or use `sudo`) to do this.
    
 3. Include the file in your Nginx configuration at the `http`
    level.
    
 4. For **each** vhost that you wish to be blocked by requests coming
    from the netblocks listed in the Spamhaus DROP lists add the
    following at the `server` level:
    
        if ($is_spamhaus_drop) {
            return 444;
        }
        
    This will make Nginx **close the connection** for all requests
    that come from IPs listed in the Spamhaus DROP list.
    
## Updating the list
 
 Updating the list requires **reloading** the Nginx configuration. If
 you run a **high-traffic** site reloading the configuration will make
 Nginx
 [creates new workers](http://nginx.org/en/docs/control.html#reconfiguration)
 and opens new sockets while servicing the requests in progress. Hence
 **reloading** the configuration is **expensive** and shouldn't be
 taken lightly. 
 
 My suggestion is to do it automatically once a day by
 a cronjob if your site is a low traffic site. Otherwise do it
 whenever you need to do it, manually.
 
 There's no one size fits all for the reloading (updating the
 configuration) it depends on your needs and setup.
 
 Another aproach is to use [monit](http://mmonit.com) with the
 [`timestamp`](http://mmonit.com/monit/documentation/monit.html#timestamp_testing)
 test and do a reload if the timestamp on the file changes. It assumes
 that the DROP list updated either automatically or manually. As soon
 as the timestamp of `drop_list.conf` changes the Nginx configuration
 is reloaded.
 
## Placing the DROP list elsewhere

By default the `drop_list.conf` file is written on `/etc/nginx` if you
wish to place it elsewhere than pass it to the script.

    ./nginx-drop-fetch /path/to/drop_list.conf
    
You have to adjust the line in `nginx_spamhaus_drop_list.conf` to
reflect the new path. Use an **absolute path**. For example:

    geo $is_spamhaus_drop {
        default 0;
        ## Including the list.
        include /path/to/drop_list.conf;
    }

## TODO

Include example monit config for reloading Nginx.


## License

Copyright (C) 2012 António P. P. Almeida

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
 
 1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
    
 2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY AUTHOR AND CONTRIBUTORS "AS IS" AND ANY
EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL AUTHOR OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
