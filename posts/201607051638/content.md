title: php-fpm配置的一个坑
date: 2016/07/05 16:38
---

调了一个下午，发现 php-fpm 有个怪坑

我在 docker 下，基于 `php:7.0-fpm` 镜像，构建了一个容器

并且 `ADD ./app.pool.conf /usr/local/etc/php-fpm.d/` ，放了个 pool 进去

具体内容为

```ini
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = www-data
group = www-data

; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses on a
;                            specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
listen = 0.0.0.0:9000

; Choose how the process manager will control the number of child processes.
; Possible Values:
;   static  - a fixed number (pm.max_children) of child processes;
;   dynamic - the number of child processes are set dynamically based on the
;             following directives. With this process management, there will be
;             always at least 1 children.
;             pm.max_children      - the maximum number of children that can
;                                    be alive at the same time.
;             pm.start_servers     - the number of children created on startup.
;             pm.min_spare_servers - the minimum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is less than this
;                                    number then some children will be created.
;             pm.max_spare_servers - the maximum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is greater than this
;                                    number then some children will be killed.
;  ondemand - no children are created at startup. Children will be forked when
;             new requests will connect. The following parameter are used:
;             pm.max_children           - the maximum number of children that
;                                         can be alive at the same time.
;             pm.process_idle_timeout   - The number of seconds after which
;                                         an idle process will be killed.
; Note: This value is mandatory.
pm = dynamic

; The number of child processes to be created when pm is set to 'static' and the
; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
; This value sets the limit on the number of simultaneous requests that will be
; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
; CGI. The below defaults are based on a server without much resources. Don't
; forget to tweak pm.* to fit your needs.
; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
; Note: This value is mandatory.
pm.max_children = 20

; The number of child processes created on startup.
; Note: Used only when pm is set to 'dynamic'
; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
pm.start_servers = 2

; The desired minimum number of idle server processes.
; Note: Used only when pm is set to 'dynamic'
; Note: Mandatory when pm is set to 'dynamic'
pm.min_spare_servers = 1

; The desired maximum number of idle server processes.
; Note: Used only when pm is set to 'dynamic'
; Note: Mandatory when pm is set to 'dynamic'
pm.max_spare_servers = 3

;---------------------

; Make specific Docker environment variables available to PHP
env[DB_1_ENV_MYSQL_DATABASE] = $DB_1_ENV_MYSQL_DATABASE
env[DB_1_ENV_MYSQL_USER] = $DB_1_ENV_MYSQL_USER
env[DB_1_ENV_MYSQL_PASSWORD] = $DB_1_ENV_MYSQL_PASSWORD

catch_workers_output = yes
```

然而...

php-fpm 一启动就报错

```
root@8ddf4ad1d3b6:/usr/local/etc/php-fpm.d# php-fpm
[05-Jul-2016 08:36:34] ERROR: [/usr/local/etc/php-fpm.d/applll.pool.conf:4] unknown entry 'user'
[05-Jul-2016 08:36:34] ERROR: Unable to include /usr/local/etc/php-fpm.d/applll.pool.conf from /usr/local/etc/php-fpm.conf at line 4
[05-Jul-2016 08:36:34] ERROR: failed to load configuration file '/usr/local/etc/php-fpm.conf'
[05-Jul-2016 08:36:34] ERROR: FPM initialization failed
```

研究了好几个钟头无果

瞎弄的时候把 `app.pool.conf` 重命名成了 `aaa.pool.conf`

神奇的发现， php-fpm 正常了

```
root@8ddf4ad1d3b6:/usr/local/etc/php-fpm.d# php-fpm
[05-Jul-2016 08:34:07] NOTICE: fpm is running, pid 188
[05-Jul-2016 08:34:07] NOTICE: ready to handle connections
```

然后测试了几个文件名，发现只要是 `app` 打头，都会造成 php-fpm 抽筋...目测是撞上了什么奇怪的规则

然而我 Google 搜了好久都没发现有人报告这个问题...\_(:3」∠)_

嘛，自己记录一下吧.
