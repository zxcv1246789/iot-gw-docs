TWAMP 설치 가이드
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. 설치 환경
============

============ ========= ========================================================================
운영체제      CentOS    - OS 종류 및 버전이 다른 경우 패키지 의존성 문제로 설치 절차가 달라진다.
계정          root      - 모든 작업은 root 계정으로 진행해야 하며, 실행 시에도 root 계정으로 실행한다.
                        - 작업 시 디렉터리 경로를 주의해야 한다. ( /root/twamp )
============ ========= ========================================================================

2. 파일 준비
============

기존에 twamp이 설치된 호스트에 접속하여 ``/root/twamp`` 디렉터리를 압축한다.
::
  $ cd ~          // root 디렉토리로 이동한 것을 반드시 확인한다.
  
  $ tar zcvf twamp.tar.gz twamp/

만들어진 압축파일을 설치할 호스트에 전송한다.
::
  $ scp twamp.tar.gz root@{IP address}:/root/  

3. twamp 설치 및 실행 등록
=========================

1. twamp 설치
-------------
디렉터리를 root로 이동하고 libjson-c를 설치한다.
::
  $ cd ~    // root 디렉터리로 이동한 것을 반드시 확인한다.
  
  $ wget https://github.com/json-c/json-c/archive/json-c-0.13.1-20180305.tar.gz
  $ tar zxvf json-c-0.13.1-20180305.tar.gz
  $ cd json-c-json-c-0.13.1-20180305
  
  $ ./autogen.sh
  $ ./configure
  $ make
  $ make install
  $ cd ~
  
  $ vi /etc/ld.so.conf
  …
  /usr/local/lib       // 하단에 내용 추가
  
  $ ldconfig
  
현재 디렉터리가 root인 것을 확인하고 전송받은 twamp 압축파일을 해제한다.
::
  $ cd ~      // root 디렉터리로 이동한 것을 반드시 확인한다.
  
  $ tar zxvf twamp.tar.gz
  $ cd twamp
 
``targets.dat`` 파일을 수정하고 저장한다.
::
  $ vi targets.dat
  …
  xxx.xxx.xxx.xxx       // 측정할 타겟 호스트 IP 목록을 작성한다.
  yyy.yyy.yyy.yyy       
  …

**target 파일 수정 할 때 유의해야 할 점은 다음과 같다.**

#. IP 주소는 한 줄에 하나씩 입력한다.
#. ``sender.sh`` 파일을 인자없이 실행하기 위해서는 반드시 3개 이상의 호스트를 등록해야 한다.
#. 등록한 호스트에서 ``~/twamp/reflector.sh`` 를 *start* 해야한다.
  
2. 자동 실행 등록
-----------------

| ``crontab-twamp.sh`` 파일을 실행한다.
실행이 완료되면 등록이 잘 되었는지 확인한다.
::
  $ ./crontab-twamp.sh
  
  $ crontab -l
  …
  */5 * * * * /bin/bash /root/twamp/sender.sh > /dev/null 2>&1 &
  
``twamp.log`` 파일을 통해 매 0분부터 5분 단위로 내용이 추가되는지 확인한다.
::
  …
  { "session_id": "", "start_time": "2018-12-03T05:10:04.910+0000", "timestamp": 1543813804, "end_time": "2018-12-03T05:10:06.030+0000", "elapsed_time": 1.120285, "src_host": "1.2.3.4", "dst_host": "13.250.34.92", "measurement_count": 1, "measurement_index": 0, "try_count": 10, "up_lost_packets": 0, "down_lost_packets": 0, "up_duplicate_packets": 0, "down_duplicate_packets": 0, "up_outoforder_packets": 0, "down_outoforder_packets": 0, "ttl": 228, "inter_delay": { "min": 0.0, "max": 0.001, "avg": 0.00090000000000000008 }, "up_delay": { "min": 105.98200000000001, "max": 106.042, "avg": 106.02009999999999 }, "down_delay": { "min": 113.959, "max": 114.05, "avg": 114.00709999999999 }, "rtt": { "min": 219.98100000000002, "max": 220.07499999999999, "avg": 220.02809999999999 }, "ipdv": { "ipdv": 0.14299999999997648, "max": 0.066999999999983739, "min": -0.07599999999999274 }, "up_ipdv": { "ipdv": 0.056999999999987616, "max": 0.024999999999997247, "min": -0.031999999999990369 }, "down_ipdv": { "ipdv": 0.10599999999999499, "max": 0.04300000000000137, "min": -0.062999999999993617 }, "pdv": { "pdv": 0.093999999999982986, "max": 0.093999999999982986, "min": 0.0 }, "up_pdv": { "pdv": 0.059999999999990616, "max": 0.059999999999990616, "min": 0.0 }, "down_pdv": { "pdv": 0.090999999999993864, "max": 0.090999999999993864, "min": 0.0 } }
  …
  // 위와 비슷한 내용이 3번 반복되어 5분마다 출력된다.
  
3. 수동 실행
------------
``sender.sh`` 파일을 실행한다.
::
  $ ./sender.sh
  // sender.sh 실행 시 뒤에 측정할 호스트의 숫자를 써서 특정 개수만 측정할 수 있다. ex) ./sender.sh 2 <- 2개만 측정
  [OK] Start
  [OK] Finished!!!
  …

``twamp.log`` 파일을 확인한다.
::
  $ tail -f twamp.log
  …
  { "session_id": "", "start_time": "2018-12-03T05:32:25.744+0000", "timestamp": 1543815145, "end_time": "2018-12-03T05:32:26.865+0000", "elapsed_time": 1.120957, "src_host": "1.2.3.4", "dst_host": "13.250.34.92", "measurement_count": 1, "measurement_index": 0, "try_count": 10, "up_lost_packets": 0, "down_lost_packets": 0, "up_duplicate_packets": 0, "down_duplicate_packets": 0, "up_outoforder_packets": 0, "down_outoforder_packets": 0, "ttl": 228, "inter_delay": { "min": 0.001, "max": 0.002, "avg": 0.0011000000000000001 }, "up_delay": { "min": 
  …
  0.18600000000000561, "min": -0.13000000000000511 }, "down_ipdv": { "ipdv": 0.11299999999998811, "max": 0.042999999999987493, "min": -0.070000000000000617 }, "pdv": { "pdv": 0.23499999999998522, "max": 0.23499999999998522, "min": 0.0 }, "up_pdv": { "pdv": 0.2360000000000001, "max": 0.2360000000000001, "min": 0.0 }, "down_pdv": { "pdv": 0.070000000000000617, "max": 0.070000000000000617, "min": 0.0 } }
  …
  // 위와 비슷한 내용이 3번 반복되어 출력된다.

