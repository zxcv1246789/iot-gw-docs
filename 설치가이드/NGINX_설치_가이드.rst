NGINX 설치 가이드
=================
이 설치 가이드는 Windows와 Ubuntu 16.04 LTS 버전을 기준으로 작성되었습니다.

|

.. contents::

1. 개요
-------

1.1 전체 개요
^^^^^^^^^^^^^^
Nginx는 오픈 소스기반의 리버스 프록시 서버로 HTTP뿐만 아니라, HTTPS, SMTP, POP3, IMAP 프로토콜을 지원한다. 
웹 서버 소프트웨어로, 가벼움과 높은 성능을 목표로 하고, 적은 수의 쓰레드로도 많은 클라이언트를 처리할 수 있다. 
아파치와 대표적으로 다른 점은 쓰레드 기반이 아닌 비 동기 이벤트 기반이라는 것이다. 

Nginx 의 HTTP 요청(request)는 Nginx의 설정 파일인 nginx.conf 파일에서 지정된 핸들러 모듈로 전달된다. 
해당 핸들러는 HTTP요청에 대한 HTTP 응답을 생성하고 이를 필터 모듈(체인 형태)로 다시 전달하여 가공된 후 클라이언트에게 전달되는 형태이다.

.. image:: /설치가이드/images/nginx_01.png

**그림 - nginx HTTP 요청 처리 (출처 : naver d2)**

|

2. 설치 및 실행
------------------
2.1 우분투 환경
^^^^^^^^^^^^^^^^
ubuntu 16.04 LTS 버전을 기준으로 다음 아래와 같이 설치할 수 있다.
::

  $ sudo apt-get update
  $ sudo apt-get install nginx
  
설치가 끝난 후 방화벽에서 nginx를 허용하도록 설정을 변경해야 한다. 다음 명령어를 통해 nginx가 있는지 확인한다.
::

  $ sudo ufw app list
  
  Available applications:
    Nginx Full
    Nginx HTTP
    Nginx HTTPS

각 프로필에 대한 설명은 아래와 같다.

============  ============================
name          Description
============  ============================
Nginx Full    포트 80(암호화 x 트래픽), 포트 443(SSL, TSL 암호화 트래픽)을 열 수 있는 프로파일.
Nginx HTTP    포트 80을 열 수 있는 프로파일.
Nginx HTTPS   포트 443을 열 수 있는 프로파일.
============  ============================

다음 명령어를 이용해 nginx가 사용할 포트를 열어준다.
::

  $ sudo ufw allow 'Nginx HTTP'

다음 명령어를 이용해 nginx 포트가 열렸는지 확인할 수 있다.
::

  $ sudo netstat -napt | grep nginx

설치가 끝나면 nginx는 자동으로 실행된다. 다음 명령어를 통해 확인할 수 있다.
::

  $ systemctl status nginx
  
  ● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since 금 2018-06-08 10:50:22 KST; 8min ago
    Process: 23472 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid (code=exited, status=2)
    Process: 23966 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, startup=0/SUCCESS)
    Process: 23982 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, startup=0/SUCCESS)
   Main PID: 23987 (nginx)
     CGROUP: /system.slice/nginx.service
             ─23987 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
             ─23988 nginx: worker process
             
2.2 윈도우 환경
^^^^^^^^^^^^^^^
윈도우의 경우 아래 사이트에서 다운받아 사용할 수 있다.

다운로드 사이트: https://nginx.org/en/download.html

.. image:: /설치가이드/images/nginx_02.png

윈도우 OS는 Ubuntu와 달리 별다른 설정 없이 바로 사용할 수 있다.

2.3 설정 파일 위치
^^^^^^^^^^^^^^^^^^^^^^^^^^^
우분투 환경에서는 설정파일을 다음 경로에서 확인할 수 있다.
::

  $ cd /etc/nginx

윈도우 환경에서는 압축해제한 디렉터리 내부에 존재한다.
::

  {nginx 설치 디렉토리}/conf

2.4 조작 방법
^^^^^^^^^^^^^^^^^^^^
**우분투 환경** 에서는 아래 3개의 명령어중 하나를 택하여 실행할 수 있다.
::

  // 실행 시 
  $ sudo service nginx start

  // 재실행 시
  $ sudo service nginx restart

  // 중지 시
  $ sudo service nginx stop

|

**윈도우 환경** 에서는 설치한 디렉토리에서 실행파일을 바로 실행하면 된다.

.. image:: /설치가이드/images/nginx_03.png

중지 시에는 커맨드창에 다음 명령어를 입력하면된다.
::

  > taskkill /f /IM nginx.exe

|

2.5 실행 확인
^^^^^^^^^^^^^^
nginx가 정상적으로 실행되면 브라우저 상에 
http://{자신의 ip} 혹은 http://localhost 경로로 들어가서 다음 페이지를 확인할 수 있다.

.. image:: /설치가이드/images/nginx_04.png

nginx는 설치를 하면 별도의 log를 보관하는 폴더를 생성하고 그 내부에 로그를 관리한다.

- 우분투

::

  $ cd /var/log/nginx

- 윈도우

::

  {nginx 설치 디렉토리}\logs

해당 디렉토리 안에는 다음과 같은 로그 파일이 있다.

==========  ========================================
name        Description
==========  ========================================
access.log  nginx에 들어온 요청에 관한 로그 (접근 로그)
error.log   에러 로그
==========  ========================================

*※ nginx.conf 파일을 수정한 후 nginx를 동작할 때 켜지지 않는 경우가 있다. 
이는 설정의 잘못으로 에러 로그를 보면 해당 설정 라인, 어떤 부분이 잘못되었는지 볼 수있다.*

|

3. 설정 파일
--------------
nginx에서 기본으로 제공되는 설정파일은 아래와 같다.

==================  ==============================================================
name                Description
==================  ==============================================================
nginx.conf          nginx의 메인 설정파일. server, http의 기초적인 설정을 할 수 있음.
fastcgi.conf        FastCGI(웹서버, 프로그램간 바이너리 통신 프로토콜) 변수 설정파일
fastcgi.params      FastCGI(웹서버, 프로그램간 바이너리 통신 프로토콜) 변수 설정파일
koi-utf             koi8-r utf-8 매핑 파일
koi-Win             window koi 매핑 파일
mine.types          타입 지정 파일
scgi_params         scgi 프로토콜 변수 설정 파일
uwsgi_params        uwsgi server 변수 설정 파일
win-utf             window utf-8 매핑 파일
==================  ==============================================================

| *(※ scgi : simple common gateway interface alternative)*
| *(※ uwsgi : wsgi 웹 서버 중의 한 종류)* 

3.1 nginx.conf
^^^^^^^^^^^^^^^^^
nginx의 메인 설정을 변경할 수 있는 파일이다. 내부는 총 4가지의 설정 영역으로 나뉘어져있다.

- | **http 블록**
  | nginx의 server를 설정하고 각 설정된 요청을 핸들링 할 수 있는 블록이다. 
    http, server, location 구조를 가지고 있으며 서로 계층 구조를 가진다. 
    (http > server > location)

  ::

    http {
      include mine.types;
      default_type application/octet-stream;
      server {
        listen 80;
        server_name localhost;
        location / {
            proxy_pass http://39.119.118.155:8090
        }
      }
    }

  계층 구조를 가지고 있기에 상위 단계에서 선언된 변수들은 하위 단계에서 
  기본값으로 설정이 된다. 위의 예시에서 default_type 이 application/octet-stream 
  으로 설정된 값이 아래 server, location에 전부 적용된다.

  ==================  ===============================================
  name                Description
  ==================  ===============================================
  keepalive_timeout   접속 커넥션 유지 설정 값 (초 단위)
  include             옵션을 불러오는 명령어. (conf 디렉토리 내부 파일을 지정하여 읽음
  ==================  ===============================================

|

- | **server 블록**
  | http 블록의 하위 단계로 하나의 웹 사이트(가상 호스트)를 선언하는데 사용된다.

  ==================  ===============================================
  name                Description
  ==================  ===============================================
  listen	            포트 설정 값
  server_name	        서버 이름
  charset	            문자 캐릭터셋
  access_log	        access를 기록할 로그 디렉토리 값
  ==================  ===============================================

|

- | **location 블록**
  | 지정한 server의 하위 블록으로 상위 servername 을 기초로 하위 url을 지정하고 
    해당 url에 대한 처리를 기술하는 블록. proxy에 관한 설정을 할 수 있다.

  ::

    location /test/ {
      proxy_pass http://00.00.00.00:00;
    }

  위의 예시는 location/test/ url 형태로 접근이 가능하다.

  - location 변경자

    ===========================  =======================================
    변경자                        설명
    ===========================  =======================================
    /site                        /site, /site/page1, /site/page1/index.html 이 매칭이 됨
    = /site	                     /site 만 매칭이 됨
    ~* .(png|jpg|gif)$	         앞의 url 상관 없이 .png, .jpg, .gif 가 매칭이 됨
    ^~ 	                         지정된 패턴으로 시작되는 url 만 매칭이 됨
    ===========================  =======================================

    ::

      location /test  { ... }
      location = /test { ... }
      location ~* .(png|jpg|gif)$ { ... }
      location ^~ /img/ { ... }

  - location 핸들링 순위

    | 1.  = 변경자를 가진 location 블럭의 문자열과 일치
    | 2.  변경자가 없는 location 블럭의 문자열과 일치
    | 3.  ^~ 변경자를 가진 location의 시작 부분과 일치
    | 4.  ~ 또는 ~* 변경자를 가진 location블럭의 패턴과 일치
    | 5.  변경자가 없는 location 블럭의 시작부분과 일치


    ::

      - 예시 1
      url -> /testlocation

      -- nginx - server 블록 내부
      location /test { 가 }
      location ~* ^/testlocation$ { 나 }

      - 우선순위: 나 > 가
      - (가)는 들어온 url의 시작 부분이 일치하지만 전체가 일치하지 않기 때문에 위의 우선순위의 5번에 해당된다.
      - (나)는 들어온 url의 문자열이 패턴과 일치하므로 우선운위의 4에 해당된다. 
    
    ::  
    
      - 예시 2
      url -> /testlocation

      -- nginx - server 블록 내부
      location /testlocation { 가 }
      location ~* ^/testlocation$ { 나 }

      - 우선순위 : 가 > 나
      - (가)는 들어온 url의 전체와 일치하기 때문에 위의 우선순위의 2번에 해당된다.
      - (나)는 들어온 url의 문자열이 패턴과 일치하므로 우선운위의 4에 해당된다.
    
    ::  
    
      - 예시 3
      url -> /testlocation

      -- nginx - server 블록 내부
      location ^~ /test { 가 }
      location ~* ^/testlocation$ { 나 }

      - 우선순위 : 가 > 나
      - (가)는 들어온 url의 시작부분이 일치하기 때문에 위의 우선순위의 3번에 해당된다.
      - (나)는 들어온 url의 문자열이 패턴과 일치하므로 우선운위의 4에 해당된다.

|

- | **upstream 블록**
  | 위의 server, location 블록과 다르게 상관관계를 가지지 않고 로드 벨런싱을 해주는 
    블록 영역이다. 총 4개의 로드 밸런싱 메소드를 제공하며 사용방법은 다음과 같다.

  ::

    updtream backend {
      [method]
      server backend1.example.com;
      server backend2.example.com;
    }

  =======================  ======================================================
  name                     Description
  =======================  ======================================================
  default                  라운드 로빈 방식
  least_conn	             연결이 가장 작은 서버로 요청을 보냄
  ip_hash	                 클라이언트 IP기준으로 요청을 분배함
  least_time (NginX plus)	 평균 레이턴시, 연결 기준으로 로드가 적은 서버로 요청을 보냄.
  =======================  ======================================================

|

- | **event 블록**
  | 네트워크의 동작 방법과 관련된 설정 값을 설정한다.

  =======================  ======================================================
  name                     Description
  =======================  ======================================================
  worker_connections	     하나의 워커가 동시에 처리하는 접속 수
  multi_accept	(on|off)   복수의 접속을 받아들이는 여부
  use	                     리눅스 커널 2.6이상일 경우 epoll / BSD의 경우 kqueue 로 지정
  =======================  ======================================================

|

nginx 설정은 include 기능을 지원한다. 선언위치는 상관 없으며, 값으로 디렉토리
위치를 지정하며 와일드카드 문자 또한 지원한다.

- http 블록

::

  http {
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*; 
  }

- sites-available/defualt

::

  server {
    location ....
  }

| *우분투의 경우 기본으로 nginx 설정파일이 sites-available 디렉토리의 파일을 전부 포함하고 있다.* 
| *윈도우의 경우는 include 설정이 지원되나, 기본적으로 사용되고 있지는 않다.*

|

4. 변수관리 및 URL 핸들링
-------------------------
nginx 에선 메인 설정파일 내부(nginx.conf)에 관리자가 직접 변수를 설정할 수 있고 
이를 이용할 수 있으며 혹은 nginx 에서 제공된 변수를 핸들링 하여 쓸 수 있다. 
여기에선 대표적인 것만 작성한다.

4.1 Set
^^^^^^^^^^^^
set은 일방적으로 문자 혹은 숫자 형태의 값을 받는다. 지정과 사용 방식은 다음과 같다.

::

  location /test/ {
    set $a 10;
    proxy_pass http://00.00.00.00:00/$a;
    # 결과 http://00.00.00.00:00/10
  }
  // 지정된 변수 a는 location /test/ 블록 내부에서만 사용가능하다.

4.2 Rewrite
^^^^^^^^^^^^^^
location에서 지정된 url을 재 작성 혹은 redirection을 하기 위하여 쓰는 모듈이다.
사용 방법은 주로 정규식을 이용해 url을 변경한다.

- 재작성

::

  location /lunch/diner/chicken {
    rewrite ^\/lunch(.*)$ $1; 
  }

  // 이 경우 기존 url이 rewrite를 통해 /diner/chicken으로 변경된다. 구조는 다음과 같다
  
  rewrite     {치환 영역 지정}    {대체할 값 지정};

- 리다이렉션

::

  location /lunch/diner/chicken {
    rewrite ^ http://00.00.00.00:00; #외부 redirection
    rewrite ^/lunch/(.*)$ /diner/pizza #내부 redirection
  }
  
  // 외부 리다이렉션의 경우 htp://00.00.00.00:00 으로 리다이렉션 해준다.
     내부 리다이렉션의 경우 /diner/pizza로 리다이렉션 해준다.

  // 특이하게 rewrite 구문에 http:// 가 존재한다면 자동으로 외부 리다이렉션을 사용한다.

4.3 Query String
^^^^^^^^^^^^^^^^^^^
nginx에서는 $query_string 변수를 제공한다. 아래는 간단한 예시이다.



::

  - url
  /ymtech?idx=1&service=control

  - nginx.conf
  location ^~ /ymtech{
    proxy_pass http://00.00.00.00:00/api?$query_string;
    # 결과 : http://00.00.00.00:00/api?$idx=1&service=control
  }

  // $query_string은 위 예시의 service, idx 이름과 파라메터 전체를 가져오지만 
     url 앞의 '?'는 가져오지 않는다. 그러기에 직접 '?'를 써줘야 한다.

4.4 정규식 사용 치환
^^^^^^^^^^^^^^^^^^^^^
query string 대신 정규식으로 사용하여 치환 후 
직접 proxy_pass 혹은 rewrite를 하여 진행하는 방법이 있다.

rewrite 정규식 치환
  location 구문에 직접 하는 방법과 다르게 rewrite를 이용해 url을 치환하여 proxy_pass 에 보내주는 방법이 있다.

  ::

    - url
    url -> lunch/finder/chicken

    - nginx.conf
    location /lunch/finder/ {
      rewrite /lunch/finder/([^/]+) /backend/server/$1 break;
      proxy_method GET;
      proxy_pass_request_headers on;
      proxy_pass http://00.00.00.00:00;
      # 결과 : http://00.00.00.00:00/backend/server/chicken
    }
    // 이 경우 query string과 다르게 proxy_pass를 수정할 필요 없이 rewrite 단계에서 url을 수정할 수 있다. 
      (참고 : http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)

|

5. Proxy
-----------
nginx는 proxy 기능으로 사용하기 위해 제공되는 메소드, 변수들이 있다. 
그 중 reqeust, response 관련 변수를 설정하려면 fastcgi.conf 파일을 수정하여 지정할 수 있다.

.. image:: /설치가이드/images/nginx_05.png

** 그림 - nginx fastcgi.conf 파일 내부 통신관련 변수 지정 일부 **

5.1 http method 설정
^^^^^^^^^^^^^^^^^^^^^^
::

  location /login {
    proxy_method POST; #혹은 $reqeust_method 사용 가능
    proxy_pass_request_header on;
    proxy_pass_reqeust_body on;
    proxy_pass http://00.00.00.00:00;
  }

- proxy_method는 http Method를 지정해준다. POST, PUT, GET, DELETE
- proxy_pass_reqeust_header는 받아온 해더값을 그대로 proxy서버에 전송되는지의 여부를 나타낸다.
- proxy_pass_reqeust_body는 받아온 바디값을 그대로 proxy서버에 전송되는지의 여부를 나타낸다.

5.2 변경자를 이용한 단순 프록시 구축
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
변경자는 받아온 url의 매칭을 해주는 역할을 하고 있다. 이를 이용해 단순 프록시 서버를 
구축할 수 있다.

::

  - API 컨텍스트가 /ymtech 일 때
  location ^~ /ymtech {
    proxy_pass_request_header on;
    proxy_pass_reqeust_body on;
    proxy_pass http://00.00.00.00:00;
  }

- 단순 서버에 들어오는 /ymtech로 시작하는 모든 url에 대해 
  해더와 바디를 그대로 http://00.00.00.00:00 으로 전달해주는 프록시 구축 예시이다.
