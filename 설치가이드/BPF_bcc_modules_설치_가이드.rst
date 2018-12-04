BPF, bcc modules 설치 가이드
============================

1 개요
------

Trust AP용 BPF 컴파일 메뉴얼을 제공한다.

2 장비 구성
-----------

ATOM 보드 (4 core, 8RAM)

VM ware (4 core, 8RAM)

3 bpf kernel compile
--------------------

VM ubuntu 16.04 LTS 버전을 설치하며, kernel은 4.4.0-116을 사용하여 아래 목록을 설치한다.

3.1 Package
~~~~~~~~~~~

3.1.1 apt-get
^^^^^^^^^^^^^

컴파일을 위한 패키지를 설치한다.

.. code:: shell

    $ sudo apt-get install -y make gcc 

커널 소스의 .config 파일을 만들기 위한 패키지를 설치한다.

.. code:: shell

    $ sudo apt-get install -y libncurses-dev libncurses5-dev bison flex

커널 모듈 빌드를 위한 패키지를 설치한다.

.. code:: shell

    $ sudo apt-get install -y libelf-dev libcap-dev util-linux binutils-dev
    $ sudo apt-get install -y libssl-dev bc gcc-multilib pkg-config libmnl-dev graphviz libdb-dev

3.2 kernel Source
~~~~~~~~~~~~~~~~~

3.2.1 .config 설정
^^^^^^^^^^^^^^^^^^

커널 소스 컴파일을 위해 .config파일을 생성한다.

.. code:: shell

    $ git clone https://github.com/torvalds/linux.git
    $ cd ~/linux
    $ make menuconfig (바로 저장)

.config 파일의 BPF 설정을 변경한다 (~/linux/.config)

.. code:: shell

    CONFIG_CGROUP_BPF=y
    CONFIG_BPF=y
    CONFIG_BPF_SYSCALL=y
    CONFIG_NET_SCH_INGRESS=m
    CONFIG_NET_CLS_BPF=m
    CONFIG_NET_CLS_ACT=y
    CONFIG_BPF_JIT=y
    CONFIG_LWTUNNEL_BPF=y
    CONFIG_HAVE_EBPF_JIT=y
    CONFIG_BPF_EVENTS=y
    CONFIG_TEST_BPF=m

3.2.2 kernel build
^^^^^^^^^^^^^^^^^^

커널 소스를 빌드한다.

.. code:: shell

    $ make –j (number of cores)

커널 모듈, 헤더을 빌드한다.

.. code:: shell

    $ make modules_install
    $ make headers_install

현재 커널 버전을 빌드왼 커널 소스(4.17.0+)로 변경한다.

.. code:: shell

    $ make install
    $ sudo reboot
    $ uname –a
    $ Linux ymtech-virtual-machine 4.17.0+ #1 SMP Tue Jun 12 11:09:53 KST 2018 x86_64 x86_64 x86_64 GNU/Linux

4 bcc 모듈 설치
---------------

기존에 사용한 llvm 이 있다면 삭제하고 3.7 버전을 설치한다.

.. code:: shell

    $ dpkg –l | grep llvm 
    $ apt-get purge llvm...

다음 bcc 모듈을 위한 패키지를 설치한다.

.. code:: shell

    $ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys D4284CDD
    $ echo "deb https://repo.iovisor.org/apt/xenial xenial main" | sudo tee /etc/apt/sources.list.d/iovisor.list
    $ sudo apt-get update
    $ sudo apt-get install bcc-tools libbcc-examples linux-headers generic
    $ VER=trusty
    $ echo "deb http://llvm.org/apt/$VER/ llvm-toolchain-$VER-3.7 main
    $ deb-src http://llvm.org/apt/$VER/ llvm-toolchain-$VER-3.7 main" | \
      sudo tee /etc/apt/sources.list.d/llvm.list
    $ wget -O - http://llvm.org/apt/llvm-snapshot.gpg.key | sudo apt-key add -
    $ sudo apt-get update
    $ sudo apt-get -y install bison build-essential cmake flex git libedit-dev \
      libllvm3.7 llvm-3.7-dev libclang-3.7-dev python zlib1g-dev libelf-dev
    $ sudo apt-get -y install luajit luajit-5.1-dev

bcc 모듈을 설치하여 build한다.

.. code:: shell

    $ git clone https://github.com/iovisor/bcc.git
    $ mkdir bcc/build; cd bcc/build
    $ cmake .. -DCMAKE_INSTALL_PREFIX=/usr
    $ make
    $ sudo make install

