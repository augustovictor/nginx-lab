# Nginx

It deals with requests asynchronously.
All requests to a dynamic content is dealt by a different process.
It serves static content without involving the application server.
[Nginx variables](http://nginx.org/en/docs/varindex.html)

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
`ulimit -n` | Find how many concurrent requests our server can handle per nginx instance
`nproc` | Number of cpu cores (ubuntu)
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
Directive | Context | Description | Example | Default
----------|---------|-------------|---------|---------
`include` | any |Imports a file from `/etc/nginx` | `mime.types;` | `text/plain`
`try_files` | any |Try to resolve a given `$uri;` | `$uri =404;` | -
`worker_processes` | parent | How many nginx instances will be starte | `auto;` | `1`
`worker_connections` | `events` | Concurrent connections simultaneously | `1000;` | -
`multi_accept` | `events` | Whether accept new connections simultaneously | `on;` | `on`
`charset` | `http` | Sets the default server charset | `utf-8;` | -
`open_file_cache` | `http` | Enables file descriptors cache | `max=1000 inactive=20s;` | -
`client_body_buffer_size` | `http` | Buffer size for client `post` requests | `16k;` | -
`client_header_buffer_size` | `http` | Headers buffer size received from client | `1k;` | -
`large_client_header_buffers` | `http` | Size and max allowed size for large client headers | `2 1k;` | -
`client_max_body_size` | `http` | Sets the max body size from a client | `8M;` | -
`client_body_timeout` | `http` | How long to wait for a body | `12;` | -
`client_header_timeout` | `http` | How long to wait for headers | `12;` | -
`keepalive_timeout` | `http` | How long to keep connections | `300;` | ?
`send_timeout` | `http` | How long connections can last | `10;` | ?
`add_header` | `http` | Add custom headers | `"Cache-Control" "no-transform";` | -

##### Directives types
- Standard directive: Can only be defined once in a context and gets inherited downwards;
- Array directive: Can be defined multiple times with different values;
- Action directive: Performs an action when hit;

## Logging
Nginx provides two types of logs: Access and Error logs;

We can also define the log level by setting a [flag](https://nginx.org/en/docs/ngx_core_module.html#error_log)

We can change where logs are writen and their level;
We can ignore logs for a specific location;

## Workers and Useful Directives

- worker_processes
    - At the top of any config we will have the `worker_processes` directive which tells how many nginx instances will be started. Nginx works async so there is no need to run more than 1 instance per cpu core.

- worker_connections
    - Defines how many concurrent connections can be handled simultaneously.
    - If we have 2 `worker_processes 1` and `worker_connections 1000` we can handle `2k` connections;

- multi_accept
    - Defines if nginx will accept all new connections simultaneously.

- charset
    - Defines the default server charset;

- open_file_cache (Open file cache)
    - Caching file descriptors. It will benefit our application if our system has lots of reading and writing;

- client_body_buffer_size (Buffer settings)
    - Buffer size received from a `post` from the client. E.g., form submitions;

- large_client_header_buffers (Buffer settings)
    - Defines size and max allowed size for large client headers;
    - Obs: Sometimes we might need to increase it;

- client_max_body_size:
    - Defines the max body size from a client. If the limit is exceeded nginx will respond with a 413 error (Request entity too large);

- client_body_timeout
    - How long to wait for a body
- client_header_timeout
    - How long to wait for a header

- keepalive_timeout
    - Defines for how long a client connection will still be valid;
    - Increase this to decrease the frequency of handshakes;
    - Decrease this to free connections from `worker_connections`;
    - `300` is a good value. It is in ms;

- send_timeout
    - Absolute time to close a connection;
    - This value is in seconds;

- add_header
    - Easy way to add custom headers

        Header | Description
        -------|------------
        `X-Frame-Options SAMEORIGIN;` | Allows browser to render a page in a frame/iframe only if same origin domain
    