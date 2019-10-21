# 1. Hadoop 설치
* 본 내용은 Virtualbox를 이용하여 구성한 환경에서 진행
  * Virtualbox 6.0.14
  * CentOS 6.1.0
    * 사양 : 2GB RAM / 20GB HDD
  * 교재: 실무로 배우는 빅데이터 기술(김강원 지음)


### 1.1 사전 환경 구축
#### 하둡 설치하기위한 환경 구성
- #### Virtualbox 에서 Centos 6 가상 머신 생성

- #### 그 외 주의사항
  #### 1. 고정 IPv4 주소를 가지고 있어야 한다
  #### 2. FQDN으로 서로를 인식해야 한다.
  #### 3. 방화벽이 꺼져 있어야 한다.
  #### 4. SELINUX 기능이 꺼져 있어야 한다.
  #### 5. vm.swappiness=0 , 가상메모리(페이징) 기능을 끈다.
  #### 6. OS의 limits를 최대로 한다(nproc, nofile)
  #### 7. THP 기능 끄기
  #### 8. root 또는 root 권한을 가져올 수 있는 계정으로 모든 장비 접근 가능.
- #### A. 첫 번째 머신 생성
  - #### 기본 정보
    - ##### id : root
    - ##### pw : adminuser
    - ##### nano /etc/inittab -> id:3 으로 변경
  - #### 고정 IP 부여
    - ##### 고정 IP 부여하는 파일
      ```
      $ nano /etc/sysconfig/network-scripts/ifgcfg-eth1
      > DEVICE=eth1
      > HWADDR=맥 주소
      > TYPE=Ethernet
      > ONBOOT=yes
      > BOOTPROTO=static
      > IPADDR=192.168.56.101
      > NETMAST=255.255.255.0
      > GATEWAY=192.168.56.1
      >NETWORK=192.168.56.0
      ```
    - ##### 이후 /etc/udev/rules.d/70-persistent-net.rules 의 네트워크 룰 삭제(전부)

  - #### SSH 설치
  ```
  $ yum install openssh*
  $ service sshd restart
  $ chkconfig sshd on
  ```
  - #### Putty 원격 접속 확인

  - #### Server 호스트 정보 수정
    ```
    $ nano /etc/hosts
    > 127.0.0.1 localhost server01
    > 192.168.56.101 server01.hadoop.com server01
    > 192.168.56.102 server02.hadoop.com server02
    > 192.168.56.103 server03.hadoop.com server03

  - #### 방화벽 및 커널 매개변수 설정 - 다음 명령어 순차 실행함
  ```
  $ nano /etc/selinux/config
  $ service iptables stop
  $ chkconfig iptables off
  $ chkconfig ip6tables off
  $ sysctl -w vm.swappiness=100
  $ nano /etc/sysctl.conf - vm.swappiness=100 설정 추가
  $ nano /etc/rc.local - 다음 설정 추가
  > echo never > /sys/kernel/mm/transparent_hugepage/enabled
  > echo never > /sys/kernel/mm/transparent_hugepage/defrag
  $ nano /etc/security/limits.conf - 다음 설정 추가
  > root soft nofile 65536
  > root hard nofile 65536
  > * soft nofile 65536
  > * hard nofile 65536
  > root soft nproc 32768
  > root hard nproc 32768
  > * soft nproc 3268
  > * hard nproc 3268
  $ reboot
  ```

- #### B. 하둡 실습을 위한 가상머신 복제(총 3개 사용)
  - ####  VM VirtualBox에서 서버 선택하여 복제
    - ##### 이 때, 네트워크 카드 MAC 주소 초기화 필요
    - ##### /etc/sysconfig/network-scripts/ifcfg-eth1의 IPADDR 수정
    - ##### /etc/sysconfig/network-scripts/ifcfg-eth1의 MAC 주소 수정
    - ##### /etc/udev/rules.d/70-persistent-net.rules 의 네트워크 룰 삭제(전부)
    - ##### /etc/hosts Alias serverXX 로 수정
    - ##### /etc/sysconfig/network HOSTNAME 을 serverXX.hadoop.com으로 수정


### 1.2 Hadoop 배포본 설치
#### 배포본에 포함되는 소프트웨어 대충 소개
- #### Zookeeper : 분산환경 관리 프로그램
- #### Sqoop : SQL 사용 DB와 하둡과의 데이터 연동
- #### Flume : Log Control
- #### Pig : 맵 리듀스 작업을 명령하는 Script *단점: Pig용 Script 언어 학습 필요*
- #### Hive : SQL로 작업
- #### Mahout : ML
- #### Oozie : 하둡 환경 Workflow 관리
- #### Hbase : Column 기반 DBMS / HDFS 상에서 돌아감, 맵 리듀스 처리 x(내부 엔진)
- #### Implala : Hbase와 유사함, 자체 엔진을 이용해 처리
