### squid类似于nginx , haproxy这种代理软件

#### 正向代理(生产环境上不能访问互联网的服务器-->通过代理服务器-->访问互联网,下载安装软件)
* nginx自带的功能只支持 http 的正向代理, 不支持https的正向代理,要非常麻烦地下载nginx插件,然后加入该模块,重新编译nginx
* squid自带的功能就支持 http和http的正向代理
* yum可以在 /etc/yum.conf配置文件里面设置 proxy=http://ip:port 通过代理下载安装软件

```
mkdir -p squid
cat > squid/squid.conf <<EOF
acl SSL_ports port 443
acl Safe_ports port 80      # http
acl Safe_ports port 21      # ftp
acl Safe_ports port 443    # https
acl Safe_ports port 70       # gopher
acl Safe_ports port 210	    # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280	    # http-mgmt
acl Safe_ports port 488     # gss-http
acl Safe_ports port 591     # filemaker
acl Safe_ports port 777     # multiling http
acl CONNECT method CONNECT
http_access allow !Safe_ports
http_access allow CONNECT !SSL_ports
http_access allow localhost manager
http_access allow manager
http_access allow localhost
http_port 3128
cache_dir ufs /var/spool/squid 100 16 256
coredump_dir /var/spool/squid
refresh_pattern ^ftp:     1440  20%  10080
refresh_pattern ^gopher:     1440  0%  1440
refresh_pattern -i (/cgi-bin/|\?)  0  0%  0
refresh_pattern (Release|Packages(.gz)*)$     0  20%  2880
refresh_pattern .     0  20%  4320


cache_mem 64 MB 
maximum_object_size 4 MB 
access_log /var/log/squid/access.log 
http_access allow all
EOF


docker run --name squid -itd --restart=always -p 3128:3128 -v `pwd`/squid/squid.conf:/etc/squid/squid.conf sameersbn/squid:3.5.27-2
# 查看日志 
curl -x localhost:3128 http://www.baidu.com
curl -x localhost:3128 https://www.baidu.com
curl -x localhost:3128 https://www.taobao.com
docker exec -it squid cat /var/log/squid/access.log
```