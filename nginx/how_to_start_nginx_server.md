20230202

root 계정으로 접속한다
nginx 설치 장소는 /etc/nginx

nginx.conf 파일을 열고
#virtual host 에서 nginx 하위 경로를 허용한다

conf.d 로 들어가서 도메인 파일을 만든다.
ex) vi dahee.cubox.com
```
server {
        listen      {포트번호};

        server_name  {서버 명};

        client_max_body_size 4G;

        #access_log  /home/ubuntu/nginx_log/access.log  main;


        location / {

                 proxy_pass  http://{호스트:포트};

        }


        location /{라우트} {

                rewrite ^/div(.*)$ $1 break; # url에서 other 뒤에 있는 URL을 전부 그대로 사용.
                proxy_pass  http://{호스트}:{포트};

        }

        location /{라우트} {

                rewrite ^/total(.*)$ $1 break; # url에서 other 뒤에 있는 URL을 전부 그대로 사용.
                proxy_pass  http://{호스트}:{포트};
        }


         #error_page  403 404 405 406 411 497 500 501 502 503 504 505 /home/opc/engine/nginx-1.18.0/html/error.html;


}
```

저장하고 나가서 엔진엑스를 재실행한다.
```
systemctl restart nginx
systemctl status nginx
```

status가 enabled로 되어있으면 호스트:포트로 들어가 확인한다.

