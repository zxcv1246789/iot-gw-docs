네트워크 품질관리 시스템 설치 가이드
---------------------------------

 시스템 구성
============

 전체 시스템 구성
-----------------

그림 전체 시스템 구성도

네트워크 품질 관리 시스템은 네트워크 품질 관리를 위한 관리 서버
및 WEB, 품질 측정을 위한 네트워크 품질 측정 SW(Light TWAMPSender/Reflector), 품질 측정 결과 분석 데이터 저장을 위한 ElasticSearch, 측정 결과 그래프 화면을 위한 Kibana로 구성된다.

 필요 프로그램 설치
-------------------

CentOS 7 64bit 1406 환경에서 테스트 하였다.

관리자 권한 획득
~~~~~~~~~~~~~~~~

일반 사용자가 관리자 권한을 획득 할 수 있게 /etc/sudoers를 수정 한다.

::

 $ su
 - root 계정의 암호를 입력한다.
 $ vi /etc/sudoers  
 - vi 편집기로 /etc/sudoers 파일을 연다.

-  root ALL=(ALL) ALL 이라는 부분을 찾아서 아래에 sudo 명령어를 사용할
   사용자 계정을 root 처럼 추가해 준다.
-  그 후, 해당 파일은 읽기전용 파일 이므로 :wq!입력하여 강제로 저장후 vi를 종료 한다.

Curl 설치
~~~~~~~~~

::

 $ sudo yum install curl 
 - curl 최신버전을 설치 한다.

JAVA JDK 다운로드 및 설치
~~~~~~~~~~~~~~~~~~~~~~~~~

네트워크 품질 관리 시스템은 JAVA 1.8 환경에서 실행할 것을 권장 한다.

JDK 설치 가능 확인
^^^^^^^^^^^^^^^^^^
::

  $ yum list java*jdk-devel  
  -  현재 시스템 에서는 1.8 버전이 설치 가능하다.  1.8 버전을 설치 한다.
  $ sudo yum install java-1.8.0-openjdk-devel.x86_64  -y
  $ rpm –qa java*jdk-devel  
  $ javac -version          
  - JDK 설치를 확인 한다.

json-c 설치
~~~~~~~~~~~

json-c 라이브러리는 runtime 패키지와 개발용 패키지가 필요하다.
centos repository에서 제공되지 않는 경우 https://rpmfind.net 사이트에서 검색/다운로드 한다.
::

  $ rpm -Uvh json-c-0.11-4.el7_0.x86_64.rpm        
  $ rpm -Uvh json-c-devel-0.11-4.el7_0.x86_64.rpm  

gRPC 설치
~~~~~~~~~

gRPC는 패키지로 제공되지 않으며, https://github.com 에서 다운로드하여 설치한다.
소스를 받기 위해서는 git 이 설치 되어 있어야 한다.
::

  $ git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc                                           
  $ cd grpc                                                              
  $ git submodule update –-init                                           
  $ make                                                                 
  $ make install                                                         
  $ cd third_party/protobuf                                              
  $ make install                                                         

Node.js 설치
~~~~~~~~~~~~

Web서버를 띄우기 위한 프로그램인 Node.js를 설치 한다. 버전은 v8 LTS를 사용 한다.
::

  $ curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -                                                         
  $ sudo yum -y install nodejs                                           
  - 바이너리 배포판 저장소를 추가 한 후, Node.js를 설치 한다.

Forever 설치
^^^^^^^^^^^^

Node.js를 설치한 후, Web서버를 background에서 실행시키기 위하여 Forever를 설치 한다.
::

  $ sudo npm install forever -g  
  - Forever를 설치 한다.

Maria DB 설치, 설정
~~~~~~~~~~~~~~~~~~~
::

  $ sudo vi /etc/yum.repos.d/MariaDB.repo  
  - Repo 설정을 위해 해당 커맨드을 입력하여 vi 편집기를 실행 한다.

  [mariadb]                                            
  name = MariaDB                                       
  baseurl = http://yum.mariadb.org/10.1/centos7-amd64  
  gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB   
  gpgcheck=1                                           

  - 그 후, 해당 문자열을 입력한 후, 읽기전용 파일 이므로 :wq!로 저장한다.

  $ sudo yum install MariaDB-server  
  - MariaDB 설치를 시작한다.

  $ systemctl start mariadb
  -  mariadb 서비스를 시작하여 정상적으로 구동하는지 확인한다.

  $ systemctl stop mariadb
  - mariadb 서비스를 정지한다.
  
  - 그후, mariadb의 root계정의 패스워드를 변경한다.
  
  $ sudo /usr/bin/mysqld_safe --skip-grant &
  - mariadb 서비스를 안전모드로 실행 한다.
  
  $ mysql -uroot mysql
  - root권한으로 mariadb에 접속한다.
  
  Mariadb[mysql]> update user set password=password('twamp') where user='root';
  - root계정의 비밀번호를 변경한다.(여기서는 twamp로 변경한다.)
  Mariadb[mysql]> flush privileges;
  - flush 한다.
  Mariadb[mysql]> exit;
  - 종료한다.
  
  $ mysql –uroot -p  
  - 해당 커맨드를 입력 한 후, 변경한 비밀번호를 입력하여 mariadb에 접속한다.

  create database if not exists twamp_portal;                              
  create user ‘twampuser’@’%’ identified by ‘twamppass’;     
  grant all privileges on twamp_portal.\* to twampuser@’%’;  
  flush privileges;                                          
  quit;                                                      


-  Database를 생성한 후, 사용자 계정을 생성, DB 권한 부여 한다.

-  DB 이름은 twamp_portal과 동일하게 해준다.(sql파일 – Table setting
   위해)
   
-  그 후, quit;를 입력하여 mysql을 빠져나온다.

::

  $ mysql –u twampuser –p twamppass twamp_portal < twamp_portal.sql  
  -  Mysql에 접속할때와 동일하게 사용자 계정 이름과 password를 입력해준 후, 뒤에 생성한 DB이름, 제공한 Table 생성 sql파일을 입력해준다.

시간 동기화 (NTP) 설치
~~~~~~~~~~~~~~~~~~~~~~
::

  $ sudo yum install ntp  
  - 관리자 계정의 패스워드를 입력한 후 설치를 진행 한다.

  $ sudo vi /etc/ntp.conf  

  - /etc/ntp.conf 파일을 편집하기 위해 vim 실행 한다.
  - 이 부분을 찾아서 주석 처리(#) 한 후, 해당 문자열을 입력하고 저장한다.

  server 1.kr.pool.ntp.org    
  server 3.asia.pool.ntp.org  
  server 1.asia.pool.ntp.org  

::

  $ firewall-cmd --add-service=ntp –permanent  
  $ firewall-cmd --reload                      
  - ntp 방화벽 설정을 추가한 후 Reload 한다.

::

  $ sudo systemctl start ntpd   
  $ sudo systemctl enable ntpd  
  - ntp 서비스를 시작하고, 시스템 재부팅 후에도 자동으로 시작할 수 있도록 한다.

Elasticsearch & Kibana
----------------------

elasticsearch 설치 및 구동
~~~~~~~~~~~~~~~~~~~~~~~~~~

다운로드
^^^^^^^^

elasticsearch는 다음 URL에서 다운로드 할 수 있다.
::

  홈페이지                                                               
  https://www.elastic.co/kr/products/elasticsearch                       
                                                                         
  다운로드 URL                                                           
  https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz                                                             


압축 해제
^^^^^^^^^

Elasticsearch는 압축을 해제하고, 몇 가지 설정만 수정하여 실행하기 때문에
운영할 디렉터리를 생성하고 해당 디렉터리에서 압축을 해제한다.
::

  $ cd /home/twamp/twamp                             
  $ tar zxvf <저장 경로>/elasticsearch-6.4.0.tar.gz  
  $ cd elasticsearch/conf                            

설정
^^^^

외부 서비스에서 검색/저장을 수행할 수 있도록 하기 위해서는 vi 등의
편집기를 이용하여 IP를 설정하여야 한다.
::

  $ vi elasticsearch.yml        
  #network.host: 192.168.0.1    
  network.host: 210.120.248.53  

시스템 설정(vm.max_map_count)을 항목을 최소 262144 이상 설정해야 한다.
설정 방법은 다음과 같다.
임시 설정은 현재 시스템이 부팅되어 있는 상태에만 유효하며 재부팅시 설정은 사라진다.
::

  $ sysctl -w vm.max_map_count=262144  

영구적으로 설정하여 시스템이 재부팅 되어도 유지하기 위해서는 시스템 설정
파일(sysctl.conf)에 vm.max_map_count를 추가한다.
::

  /etc/sysctl.conf
  vm.max_map_count=262144  

Elasticsearch의 실행
^^^^^^^^^^^^^^^^^^^^

Elasticsearch는 일반 계정으로 실행하여야 하며, 실행은 다음과 같이 elasticsearch 실행한다.
::

  $ cd ../bin                                                    
  $ elasticsearch -d -p /home/twamp/twamp/run/elasticsearch.pid  

Kibana 설치 및 구동
~~~~~~~~~~~~~~~~~~~

다운로드
^^^^^^^^

kibana는 다음 URL에서 다운로드 할 수 있다.
::

  홈페이지                                                               
  https://www.elastic.co/kr/products/elasticsearch                       
                                                                         
  다운로드 URL                                                           
  https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz                                                              

압축 해제
^^^^^^^^^

kibana는 압축을 해제하고, 몇 가지 설정만 수정하여 실행하기 때문에 운영할
디렉터리를 생성하고 해당 디렉터리에서 압축을 해제한다.
::

  $ cd /home/twamp/twamp                                    
  $ tar zxvf <저장 경로>/ kibana-6.4.0-linux-x86_64.tar.gz  
  $ cd kibana-6.4.0-linux-x86_64/config                     


설정
^^^^

외부 서비스에서 검색을 수행할 수 있도록 하기 위해서는 vi 등의 편집기를
이용하여 IP를 설정하여야 한다.
::
  $ vi kibana.yml                                  
  …                                                
  #server.host: “localhost”                        
  server.host: “0.0.0.0”                           
  …                                                
                                                   
  #elasticsearch.url: "http://localhost:9200"      
  elasticsearch.url: "http://210.120.248.53:9200"  

Kibana의 실행
^^^^^^^^^^^^^

Kibana는 일반 계정으로 실행하여야 하며, 실행은 다음과 같이
::

  $ cd ../bin                                                            
  $ nohup kibana serve -l                                                
  /home/twamp/twamp/kibana-6.4.0-linux-x86_64/log/kibana.log 1> /dev/null 2>&1 &                                                       

Kibana의 경우 pid를 저장하는 옵션이 존재하지 않는다. Kibana를 쉽게
관리하기 위하여 제공되는 스크립트를 이용한다.
::

  $ cd /home/twamp/twamp  
  # 실행                  
  $ ./ctl-k.sh start      
  # 종료                  
  $ ./ctl-k.sh stop       
  # 실행 확인             
  $ ./ctl-k.sh status     
+------------------------+

웹 & API 서버 설치
------------------

API 서버와 WEB 서버를 설치하고 구동에 필요한 목록들을 설정 한다.

압축 해제
~~~~~~~~~
::

  $ ls                 
  $ tar xvf twamp.tar  
  - 제공한 twamp.tar 파일을 푼다.

디렉토리 구조
~~~~~~~~~~~~~

API 서버 디렉토리 구조
^^^^^^^^^^^^^^^^^^^^^^
::

  - twampAPI : twamp API jar 파일과 config 파일(application.properties), 실행 스크립트 파일이 들어있다.
  - log : API 서버 log 파일이 저장된다.
  - run : API 서버 구동시 PID가 저장 된다.
  - src : Kibana – visualization, dashboard 생성 관련 json 파일이 저장되어있다.
  - twamp-api-ctrl.sh : API 서버 jar 파일 실행 스크립트 파일이다.

WEB 서버 디렉토리 구조
^^^^^^^^^^^^^^^^^^^^^^
::

  - twampd.web : 서버 구동시 필요한 JS 파일과 Angular 설정 파일들이 저장되어있다.
  - src : html 파일과 ts 파일들이 저장되어있다.
  - assets : Web서버 구동시 변경 가능한 설정이 저장되어있다
  - dist : Web서버 build시 파일이 저장되는 위치이다.
  - Server.js : Web서버 구동시 실행하는 파일이다.

API 서버 설정 & 구동
~~~~~~~~~~~~~~~~~~~~

설정
^^^^
::

  $ vi application.properties  
  - application.properties 파일을 vi 편집기로 실행 한다.
  - url=jdbc:mariadb://{DB서버주소}:{DB포트}/{DB이름}
  - username={DB사용자계정이름}
  - password={DB사용자계정비밀번호}
  - config.twamp.visualization.host={ElasticSearch서버주소} <- 주소는 localhost가 아닌 ip 주소입력
  - config.twamp.visualizaiton.port={ElasticSearch포트}
  - server.port={API서버설정할포트} <- 가급적 변경 금지


구동
^^^^
::

  ./twamp-api-ctrl.sh run  
  - 쉘 스크립트 파일을 이용하여 API서버 jar파일을 foreground 상태로 실행한다.

  ./twamp-api-ctrl.sh start  
  - 쉘 스크립트 파일을 이용하여 API서버 jar파일을 background 상태로 실행한다.

  ./twamp-api-ctrl.sh stop  
  - 쉘 스크립트 파일을 이용하여 실행중인 서버 프로세스를 종료 한다.

WEB 서버 설정 & 구동
~~~~~~~~~~~~~~~~~~~~

설정
^^^^
::

  $ npm install  
  - 해당 디렉토리로 이동하여, package.json에 등록되어있는 의존성
    패키지들을 설치 한다.

  $ vi dist/assets/config.dev.json  

  - 해당 디렉토리로 이동하여, package.json에 등록되어있는 의존성
    패키지들을 설치 한다.
  - name : build 옵션 종류 (변경 금지)
  - url :
  - pagination : 조회 테이블 옵션(변경 금지)
  - kibana.version : Kibana 버전 확인하여 변경
  - login : 웹 로그인시 필요한 id, password 지정

구동
^^^^
::

  $ node server.js  
  - server.js 파일을 foreground 상태로 실행 한다.(중지: Ctrl + C)

  $ forever start server.js  
  - server.js 파일을 forever 패키지로 background 상태로 실행 한다.

  $ forever stop server.js  
  - background 상태로 실행중인 server.js 프로세스를 중지 한다.

측정 도구 설치
--------------

측정 도구는 제공되는 소스를 컴파일하여 설치한다.

디렉터리 구성
~~~~~~~~~~~~~
::

  $ cd /home/twamp          
  $ mkdir twamp             
  $ cd twamp                
  $ mkdir bin run src conf  

fping
~~~~~
::

  $ cd /home/twamp/twamp/src                
  $ tar zxvf fping-4.0.tar.gz               
  $ cd fping                                
  $ make                                    
  $ cp src/fping /home/twamp/twamp/bin      
  $ chmod 4755 /home/twamp/twamp/bin/fping  

gRPC 데몬
~~~~~~~~~

빌드 및 설치
^^^^^^^^^^^^
::

  $ cd /home/twamp/twamp/src                                
  $ tar zxvf twampd.rpc.tar.gz                
  $ cd twampd.rpc                             
  $ make                                                    
  $ cp twampd twamp-client /home/twamp/twamp/bin            
  $ cp twampd.json measurement.json /home/twamp/twamp/conf  

설정
^^^^

다음은 측정 데몬의 설정 파일이다. 디폴트 설정 값으로 데몬 실행 시 주어지는 인자는 디폴트 값을 override 한다.

::

  /home/twamp/twamp/conf/twampd.json
  {                                                 
  "comment": "TWAMP Service configuration",         
  "daemon" : true,                                  
  "pid": "twampd.pid",                              
  "log": null,                                      
  "port": 2000,                                     
  "verbose": false,                                 
  "twamp": {                                        
  "command" : "/home/twamp/twamp/bin/twampSender",  
  "arguments" : [                                   
  "-f",                                             
  "/home/twamp/twamp/conf/twampSender.json"         
  ]                                                 
  },                                                
  "icmp": {                                         
  "command" : "/home/twamp/twamp/bin/fpingSender",  
  "arguments" : [                                   
  "-f",                                             
  "/home/twamp/twamp/conf/fpingSender.json"         
  ]                                                 
  }                                                 
  }                                                 

실행
^^^^
::

  $ cd /home/twamp/twamp/src                           
  $ ./twampd -D -f /home/twamp/twamp/conf/twampd.json  
  $                                                    

사용법
^^^^^^

::

  $ ./twampd -h                                
  usage : ./twampd [options]                    
                                                
  -D --daemon run as daemon                     
  -p --port <port> service port(default: 2000)  
  -f --config <path> configuration file path    
  -i --pid <path> pid file path                 
  -l --log <path> log file path                 
  -v --verbose verbose                          
  -h --help this message                        

TWAMP 측정 도구
~~~~~~~~~~~~~~~

빌드 및 설치
^^^^^^^^^^^^

::

  $ cd /home/twamp/twamp/src                                      
  $ tar zxvf twamp.tar.gz                           
  $ cd twamp                                        
  $ make                                                          
  $ cp twampCommander twampReflector /home/twamp/twamp/bin        
  $ cp sender.json /home/twamp/twamp/conf/twampSender.json        
  $ cp reflector.json /home/twamp/twamp/conf/twampReflector.json  

설정
^^^^

다음은 TWAMP 측정 도구의 설정 파일이다. 디폴트 설정 값으로 데몬 실행 시
주어지는 인자는 디폴트 값을 override 한다.

::

  /home/twamp/twamp/conf/twampSender.json
  {                                                                       
  "comment": "TWAMP Sender configuration",                                
  "daemon" : false,                                                       
  "pid": null,                                                            
  "log": null,                                                            
  "host": null,                                                           
  "port": 20000,                                                          
  "duration": 1000,                                                       
  "measurement": 1,                                                       
  "count": 10,                                                            
  "session_id": null,                                                     
  "test_comment": "0: owamp, 1: twamp(default), 2: icmp, 3: twamp+icmp",  
  "test": 1,                                                              
  "timeout":1000,                                                         
  "debug_level": "1: TRACE, 2: DEBUG, 4: INFO, 8: WARNING, 16: ERROR",    
  "debug": 16,                                                            
  "result" : {                                                            
  "enable": true,                                                         
  "url":"http://xx.xx.xx.xx:9200/twamp/measurement",                   
  "method-example": "POST"                                                
  },                                                                      
  "test_start" : {                                                        
  "enable": true,                                                         
  "url":"http://127.0.0.1:8090/current-status/${ session_id }",           
  "method-example": "PUT"                                                 
  },                                                                      
  "test_end" : {                                                          
  "enable": true,                                                         
  "url":"http://127.0.0.1:8090/quality-history",                          
  "method-example": "POST"                                                
  }                                                                       
  }                                                                       

다음은 twamp reflector의 설정 파일이다. 디폴트 설정 값으로 데몬 실행 시
주어지는 인자는 디폴트 값을 override 한다.

::

  /home/twamp/twamp/conf/twampReflector.json
  {                                                                     
  "comment": "TWAMP Reflector configuration",                           
  "daemon" : true,                                                      
  "pid": "reflector.pid",                                               
  "log": null,                                                          
  "port": 20000,                                                        
  "debug_level": "1: TRACE, 2: DEBUG, 4: INFO, 8: WARNING, 16: ERROR",  
  "debug": 16                                                           
  }                                                                     

실행
^^^^

Twamp 측정 도구는 측정 데몬(twampd) 가 측정 요청 수신 시에 실행한다.
다음은 수동 실행 명령의 예이다.

::

  $ cd /home/twamp/twamp/bin                                             
  $ ./twampSender -f twampSender-no-report.json -s 11036 -H xx.xx.xx.xx -p 862 -o 3000 -m 10 -c 1000 -t 1 --debug=30              

사용법
^^^^^^

::

  $ ./twampSender -h                                                     
  usage : ./twampSender [options]                                        
                                                                         
  measurement:                                                           
  -H --host <target ip> target ip                                        
  -p --port <target port> target port(default: 20000)                    
  -c --count <count> send count(default: 10)                             
  -d --duration <milliseconds> test duration (default: 1000 ms)          
  -m --measurement <count> measurement count(default: 3)                 
  -o --timeout <millisecond> default: 1000 ms                            
  -s --session <session id> session id                                   
  -t --test <mode> 0: owamp, 1: twamp(default), 2: icmp, 3: twamp+icmp   
                                                                         
  etc:                                                                   
  -D --daemon run as daemon                                              
  -f --config <path> configuration file path                             
  -i --pid <path> pid file path                                          
  -l --log <path> log file path                                          
  --debug <number> debug level(1: TRACE, 2: DEBUG, 4: INFO, 8: WARNING,  16: ERROR)
  -h --help this message                                                 


ICMP 측정 도구
~~~~~~~~~~~~~~

빌드 및 설치
^^^^^^^^^^^^

:: 

  $ cd /home/twamp/twamp/src                               
  $ tar zxvf fping.tar.gz                    
  $ cd fping                                 
  $ make                                                   
  $ cp fpingSender /home/twamp/twamp/bin                   
  $ cp fping.json /home/twamp/twamp/conf/fpingSender.json  


설정
^^^^

다음은 fpingSender의 설정 파일이다. 디폴트 설정 값으로 데몬 실행 시
주어지는 인자는 디폴트 값을 override 한다.

::

  /home/twamp/twamp/conf/fpingSender.json

  {                                                                       
  "comment": "ICMP Sender configuration",                                 
  "command": "/home/twamp/twamp/bin/fping",                               
  "arguments" : [                                                         
  "-e",                                                                   
  "-s",                                                                   
  "-i", 0                                                                 
  ],                                                                      
  "daemon" : false,                                                       
  "pid": null,                                                            
  "log": null,                                                            
  "host": null,                                                           
  "duration": 1000,                                                       
  "measurement": 1,                                                       
  "count": 100,                                                           
  "session_id": null,                                                     
  "test_comment": "0: owamp, 1: twamp(default), 2: icmp, 3: twamp+icmp",  
  "test": 1,                                                              
  "timeout":1000,                                                         
  "debug_comment": "1: TRACE, 2: DEBUG, 4: INFO, 8: WARNING, 16: ERROR",  
  "debug": 31,                                                            
  "result" : {                                                            
  "enable": true,                                                         
  "url":"http://xx.xx.xx.xx:9200/twamp/measurement",                   
  "method-example": "POST"                                                
  },                                                                      
  "test_start" : {                                                        
  "enable": true,                                                         
  "url":"http://127.0.0.1:8090/current-status/${ session_id }",           
  "method-example": "PUT"                                                 
  },                                                                      
  "test_end" : {                                                          
  "enable": true,                                                         
  "url":"http://127.0.0.1:8090/quality-history",                          
  "method-example": "POST"                                                
  }                                                                       
  }                                                                       

실행
^^^^

ICMP 측정 도구는 측정 데몬(twampd) 가 측정 요청 수신 시에 실행한다.
다음은 수동 실행 명령의 예이다.

::

  $ cd /home/twamp/twamp/bin                                             
  $ ./fpingSender -f fpingSender-no-report.json -s 11053 -H xx.xx.xx.xx -o 3000 -m -1 -c 100 -t 2 --debug=30                    

사용법
^^^^^^

::

  $ ./fpingSender -h                                                     
  usage : ./fpingSender [options]                                        
                                                                         
  measurement:                                                           
  -H --host <target ip> target ip                                        
  -c --count <count> send count(default: 10)                             
  -d --duration <milliseconds> test duration (default: 1000 ms)          
  -m --measurement <count> measurement count(default: 3)                 
  -o --timeout <millisecond> default: 1000 ms                            
  -s --session <session id> session id                                   
  -t --test <mode> 0: owamp, 1: twamp(default), 2: icmp, 3: twamp+icmp   
                                                                         
  etc:                                                                   
  -D --daemon run as daemon                                              
  -f --config <path> configuration file path                             
  -i --pid <path> pid file path                                          
  -l --log <path> log file path                                          
  --debug <number> debug level(1: TRACE, 2: DEBUG, 4: INFO, 8: WARNING, 16: ERROR)
  -h --help this message                                                 
