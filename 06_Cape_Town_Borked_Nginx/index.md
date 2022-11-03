admin@ip-172-31-41-252:/$ lsof -i:80
admin@ip-172-31-41-252:/$ systemctl start nginx
Failed to start nginx.service: Access denied
See system logs and 'systemctl status nginx.service' for details.
admin@ip-172-31-41-252:/$ systemctl status nginx.service 
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Thu 2022-11-03 02:04:53 UTC; 1min 42s ago
    Process: 576 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
        CPU: 29ms
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Nov 03 02:04:53 ip-172-31-41-252 nginx[576]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
Nov 03 02:04:53 ip-172-31-41-252 nginx[576]: nginx: configuration file /etc/nginx/nginx.conf test failed
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: nginx.service: Failed with result 'exit-code'.
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: Failed to start The NGINX HTTP and reverse proxy server.
admin@ip-172-31-41-252:/$ systemctl status nginx.service 
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Thu 2022-11-03 02:04:53 UTC; 1min 42s ago
    Process: 576 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
        CPU: 29ms

Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Nov 03 02:04:53 ip-172-31-41-252 nginx[576]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
Nov 03 02:04:53 ip-172-31-41-252 nginx[576]: nginx: configuration file /etc/nginx/nginx.conf test failed
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: nginx.service: Failed with result 'exit-code'.
Nov 03 02:04:53 ip-172-31-41-252 systemd[1]: Failed to start The NGINX HTTP and reverse proxy server.
admin@ip-172-31-41-252:/$ sudo vi /etc/nginx/sites-enabled/default
     1  ;
     2  ##
     3  # You should look at the following URL's in order to grasp a solid understanding
     4  # of Nginx configuration files in order to fully unleash the power of Nginx.
     5  # https://www.nginx.com/resources/wiki/start/
admin@ip-172-31-41-252:/$ sudo systemctl start nginx
admin@ip-172-31-41-252:/$ curl -I 127.0.0.1:80
HTTP/1.1 500 Internal Server Error
Server: nginx/1.18.0
Date: Thu, 03 Nov 2022 02:11:23 GMT
Content-Type: text/html
Content-Length: 177
Connection: close
2022/11/03 02:10:53 [alert] 1065#1065: socketpair() failed while spawning "worker process" (24: Too many open files)
2022/11/03 02:10:53 [emerg] 1066#1066: eventfd() failed (24: Too many open files)
2022/11/03 02:10:53 [alert] 1066#1066: socketpair() failed (24: Too many open files)
2022/11/03 02:11:23 [crit] 1066#1066: *1 open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files), client: 127.0.0.1, server: _, request: "HEAD / HTTP/1.1", host: "127.0.0.1"
admin@ip-172-31-41-252:/$ ps ax | grep nginx
   1065 ?        Ss     0:00 nginx: master process /usr/sbin/nginx
   1066 ?        S      0:00 nginx: worker process
   1083 pts/0    S<+    0:00 grep nginx
admin@ip-172-31-41-252:/$ cat /proc/`pgrep nginx | head -1`/limits | grep 'open files'
Max open files            10                   10                   files 
admin@ip-172-31-41-252:/$ ulimit -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 1748
max locked memory           (kbytes, -l) 64
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1024
admin@ip-172-31-47-126:/$ sudo ls -al /proc/1066/fd/
total 0
dr-x------ 2 www-data www-data  0 Nov  3 02:32 .
dr-xr-xr-x 9 www-data www-data  0 Nov  3 02:32 ..
lrwx------ 1 www-data www-data 64 Nov  3 02:32 0 -> /dev/null
lrwx------ 1 www-data www-data 64 Nov  3 02:32 1 -> /dev/null
l-wx------ 1 www-data www-data 64 Nov  3 02:32 2 -> /var/log/nginx/error.log
l-wx------ 1 www-data www-data 64 Nov  3 02:32 4 -> /var/log/nginx/access.log
l-wx------ 1 www-data www-data 64 Nov  3 02:32 5 -> /var/log/nginx/error.log
lrwx------ 1 www-data www-data 64 Nov  3 02:32 6 -> 'socket:[12816]'
lrwx------ 1 www-data www-data 64 Nov  3 02:32 7 -> 'socket:[12817]'
lrwx------ 1 www-data www-data 64 Nov  3 02:32 8 -> 'socket:[11896]'
lrwx------ 1 www-data www-data 64 Nov  3 02:32 9 -> 'anon_inode:[eventpoll]'
admin@ip-172-31-47-126:/$ sudo vi /etc/systemd/system/nginx.service 
#LimitNOFILE=10
LimitNOFILE=100
admin@ip-172-31-47-126:/$ sudo systemctl daemon-reload
admin@ip-172-31-47-126:/$ sudo systemctl stop nginx
admin@ip-172-31-47-126:/$ sudo systemctl start nginx
admin@ip-172-31-41-252:/$ curl -I 127.0.0.1:80
