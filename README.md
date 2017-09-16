# Nginx

It deals with requests asynchronously.
All requests to a dynamic content is dealt by a different process.
It serves static content without involving the application server.

## Instalation
In `centos` yum (the package manager) does not come with the package nginx. To get this working we need to install a collection of packages first:
```sh
yum install epel-release
```
then
```sh
yum install nginx
```

Ps: Modules can be added to nginx. [List of modules](https://www.nginx.com/resources/wiki/modules)

## Commands
Command | Description
---|---
`service nginx status` | Check nginx status
`service nginx start` | Start nginx server
`service nginx reload` | Kills the nginx master process and another one takes its place (0 downtime)
`nginx -t` | Test if nginx config is valid

Ps: `/etc/init.d` is where init scripts for linux server lives. 

## Nginx Folders structure
- `/etc/nginx` - All related to nginx.
- `/etc/nginx/conf.d` - Nginx config
- `/var/log/nginx/accessl.log` - Nginx log

# Definitions
### Contexts: 
Sections in config file where directives can be applied to. Structure: `context_name { ... }`
- Main context: Where options for the master process get set. E.g., `worker_processes 1`;
- `http`: Where we specify anything http related;
- `server`: Equivalent to apache vhost (virtual host);
- `location`: Matching url locations on incoming requests to the parent server context;
    - Types of match:
        - Prefix: Anything starting with given location. `location /greet`;
            - Matches: `/greet` `/greeting` `/greeting/inexistent_path`;
            - Does not match: `/gree`
        - Exact match: `location = /greet`;
            - Matches: `/greet`
            - Does not match: anything different
        - Regex: We use `~` (case sensitive) or `~*` (case insensitive) to indicate it is a regex. E.g., `location ~ /greet[0-9]`
            - Matches: `/greet0`
            - Does not match: `/greeting`
        - Preferential prefix: It works the same way `prefix` but with higher priority than regex match. E.g., `location ^~ /greet`
    - Maching priority order:
    
        Order | Name | Modifier
        ---|---|---
        1 | Exact match | `=`
        2 | Preferential match | `^~`
        3 | Regex match | `~` or `~*`
        4 | No modifier | ` `
- `types`: Nginx will serve static files as `text/plain`. To get files being served the right format we should use this context;
### Directives:
Specific configuration options set in config file. Structure: `opt_name opt_value`
- `include`: Imports a file from `/etc/nginx`. E.g., `include mime.types`;

## Logging
Nginx provides two types of logs: Access and Error logs;

We can also define the log level by setting a [flag](https://nginx.org/en/docs/ngx_core_module.html#error_log)

We can change where logs are writen and their level;
We can ignore logs for a specific location;

