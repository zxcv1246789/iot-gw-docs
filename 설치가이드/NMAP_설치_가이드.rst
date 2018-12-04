NMAP 측정 프로그램 설치 가이드
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. 설치 환경
============

============ ========= ========================================================================
운영체제      CentOS    - OS 종류 및 버전이 다른 경우 패키지 의존성 문제로 설치 절차가 달라진다.
계정          root      - 모든 작업은 root 계정으로 진행해야 하며, 실행 시에도 root 계정으로 실행한다.
                        - 작업 시 디렉터리 경로를 주의해야 한다. ( /root/nmap )
============ ========= ========================================================================

2. 파일 준비
============

기존에 nmap이 설치된 호스트에 접속하여 ``/root/nmap`` 디렉터리를 압축한다.
::
  $ cd ~          // root 디렉토리로 이동한 것을 반드시 확인한다.

  $ tar zcvf nmap.tar.gz nmap/

만들어진 압축파일을 설치할 호스트에 전송한다.
::
  $ scp nmap.tar.gz root@{IP address}:/root/
  
3. nmap 설치 및 실행 등록
========================

1. nmap 설치
------------

파일을 전송 받은 호스트로 접속하고 nmap 패키지를 설치한다.
::
  $ cd ~      // root 디렉토리로 이동한 것을 반드시 확인한다.
  $ ls -l
  …
  -rw-r--r--. 1 root root     16322 Dec  3 02:31 nmap.tar.gz
  -rw-r--r--. 1 root root    822627 Dec  3 02:30 twamp.tar.gz
  …
  
  $ yum -y install nmap.x86_64

패키지 설치가 끝나면 전송 받은 ``nmap.tar.gz`` 파일의 압축을 해제하고 압축해제된 디렉터리로 이동한다.
::
  $ tar zxvf nmap.tar.gz
  $ cd nmap

2. 자동 실행 등록
-----------------

| ``crontab-nmap.sh`` 파일을 실행한다.
| 최초 실행 시 *no crontab for root* 메시지가 출력되지만 무시하고 진행한다.
실행이 완료되면 등록이 잘 되었는지 확인한다.
::
  $ ./crontab-nmap.sh
  
  $ crontab -l
  …
  */10 * * * * /bin/bash /root/nmap/nmap.sh > /dev/null 2>&1 &
  …
  
``nmap.log`` 파일을 통해 매 0분부터 10분 단위로 내용이 추가되는지 확인한다.
::
  $ tail -f nmap.log
  …
  Starting Nmap 6.40 ( http://nmap.org ) at 2018-12-03 04:50 UTC
  Nmap scan report for ec2-54-169-156-6.ap-southeast-1.compute.amazonaws.com (54.169.156.6)
  Host is up (0.22s latency).
  Not shown: 998 closed ports
  PORT    STATE SERVICE VERSION
  22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
  111/tcp open  rpcbind 2-4 (RPC #100000)

  Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 10.63 seconds
  2018-12-03 04:50:12
  …
  // 위와 비슷한 내용이 3번 반복되어 10분마다 출력된다.

3. 수동 실행
------------

``nmap.sh`` 파일을 실행한다.
::
  $ ./nmap.sh

``nmap.log`` 파일을 확인한다.
::
  $ tail -f nmap.log
  …
  Starting Nmap 6.40 ( http://nmap.org ) at 2018-12-03 04:53 UTC
  Nmap scan report for ec2-54-169-156-6.ap-southeast-1.compute.amazonaws.com (54.169.156.6)
  Host is up (0.22s latency).
  Not shown: 998 closed ports
  PORT    STATE SERVICE VERSION
  22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
  111/tcp open  rpcbind 2-4 (RPC #100000)

  Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 10.63 seconds
  2018-12-03 04:50:12
  …
  // 위와 비슷한 내용이 3번 반복되어 출력된다.
