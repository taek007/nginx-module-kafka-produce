##声明

此模块是在 https://github.com/brg-liuwei/ngx_kafka_module 基础上改写，
增加librdkafka中的属性 Batch.num.messages 
queue.buffering.max.ms
queue.buffering.max.messages
后续topic会由用户请求url获取, kafka的实例放在共享内存中
在此对brg_liuwei表示由衷的感谢。

##Introduction

This module is used to send post data from nginx to kafka.

##Installation

###1 install librdkafka

    git clone https://github.com/edenhill/librdkafka
    cd librdkafka
    ./configure
    make
    sudo make install

###2 compile nginx with nginx kafka module

    git clone https://github.com/taek007/nginx-module-kafka-produce.git
    cd /path/to/nginx
    ./configure --add-module=/path/to/nginx-module-kafka-produce
    make && make install

###3 edit nginx.conf file
```
	#user  nobody;
	worker_processes  4;
	worker_cpu_affinity 0001 0010 0100 1000;
	error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;

	#pid        logs/nginx.pid;

	worker_rlimit_nofile 655350;
	events {
	use epoll;
	    worker_connections  102400;
	    accept_mutex off;
	    multi_accept off;
	}


	http {

		keepalive_timeout  120;
		keepalive_requests 8192; 

		include       mime.types;
		default_type  application/octet-stream;
		kafka.broker.list  1.2.3.4:9092 5.6.7.8:9092;

		# log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
		#               '$status $body_bytes_sent "$http_referer" '
		#                  '"$http_user_agent" "$http_x_forwarded_for"';

		# access_log  logs/access.log  main;
		access_log off;
		sendfile        on;
		#tcp_nopush     on;
  

		#gzip  on;

		server {
			client_header_buffer_size 4k;
			open_file_cache max=65535 inactive=60s;
			open_file_cache_min_uses 1;

			listen  8093 reuseport;

			server_name  localhost;

			#charset koi8-r;

			#access_log  logs/host.access.log  main;

			location / {
				root   html;
				index  index.html index.htm;
			}

			location = /test {
				kafka.topic  test;
			}


			error_page   500 502 503 504  /50x.html;
				location = /50x.html {
				root   html;
			}
		}
	}
```

###4 start nginx

###5 test

    curl "http://1.2.3.4:8093" -d "hello kafka"

