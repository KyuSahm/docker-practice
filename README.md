# Docker
- 이성미 강사님의 [따라하며 배우는 도커 유튜브 강의](https://www.youtube.com/watch?v=NLUugLQ8unM&list=PLApuRlvrZKogb78kKq1wRvrjg1VMwYrvi)를 정리한 내용임

## 컨테이너란?
### 컨테이너를 배워야 하는 이유?
- 대규모의 시스템들을 효율적으로 운영 가능 
- 용어 정리
  - Bare Metal: OS를 포함한 소프트웨어가 전혀 설치 되지 않은 H/W 시스템
- 세대별 변환
  - 1세대: Bare Metal 시스템에 하나의 OS와 여러개의 어플리케이션을 설치
  - 2세대: Hypervisor 기술 위에 Virtual Machine을 여러 개 설치해서 머신마다 어플리케이션을 설치
  - 3세대: 하나의 OS위에 컨테이너 엔진을 올리고, Application이 설치된 여러 개의 Container들을 동작시킴
    - Scale Out을 하기 쉬움
    - Virtual Machine에 비해 가볍고, 성능이 좋음  

![소프트웨어 플랫폼의 변환](./images/Software_Platform.png)
### 컨테이너란 개념
- Application과 그것을 동작시키는데 필요한 라이브러리 또는 소프트웨어 플랫폼들을 구성한 독립된 공간
### 리눅스에서 동작시켜야 하는 이유
- 리눅스 커널 기능을 사용해야 하기 때문
  - chroot: 독립된 공간 형성
  - namespace: 독립된 공간안에 6가지 종류의 isolate 기능을 지원
    - 하나의 가상 시스템인 것처럼 동작 시킴    
  - cgroup: 필요한 만큼의 HW 지원

![리눅스에서 동작시켜야 하는 이유](./images/Docker_On_Linux.png)
- 윈도우나 Mac OS에서 동작시키는 방법
  - Hypervisor를 활성화 시킴
### 일반 프로그램과 컨테이너의 차이
- 하는 일은 동일하나, 구조가 다름
- 아래와 같이 Node.js 기반의 서버 프로그램이 있다고 가정
```bash
$cat app.js
const http = require('http');
const os = require('os');
console.log("Test server starting.....");
var handler = function(req, res) {
    res.writeHead(200);
    res.end("Container Hostname: " + os.hostname() + "\n");
};
var www = http.createServer(handler);
www.listen(8080);
```
- 일반 프로그램으로 동작 시키려면?
  - OS가 설치된 물리적인 서버에 Node.js를 설치
  - 프로그래밍(``app.js``)을 작성
  - ``node app.js``라는 명령 수행
- 컨테이너로 동작 시키려면?
  - 프로그래밍(``app.js``)을 작성
  - Docker 이미지 생성을 위한 Dockerfile 작성해서 컨테이터 빌드
    - ``node.js``가 깔린 이미지로 부터 ``app.js``를 복사
    - 실행시 동작해야 하는 Entry Point 지정    
```bash
$cat Dockerfile
FROM node:12
COPY app.js /app.js
ENTRYPOINT ["node", "app.js"]
```
- 컨테이너는 8080 포트를 열고, 서비스를 제공해 줌
### 컨테이너를 사용하는 이유?
- 개발자가 만든 설정 그대로 어디서든 돌아감 
- 개발 환경과 운영 환경이 달라도 문제가 없음 (H/W, Base Platform)
  - 개발자의 application container 내부에 해당 application을 동작시키는 소프트웨어가 모두 존재
  - 개발자가 만든 application이 운영환경에서도 똑같이 동작함

![컨테이너를 사용하는 이유](./images/Why_Container.png)
- 확장/축소(ScaleOut/ScaleIn)가 쉽고, MSA 및 DevOps에 적합
  - Virtual Machine의 기반에서는 확장 시, OS의 개수가 계속 늘어나는 문제가 존재

![컨테이너를 사용하는 이유 - ScaleOut](./images/Why_Container_ScaleOut.png)

## 도커 설치하기
### 설치 준비 사항
- 컴퓨터: BareMetal 또는 VirtualMachine
  - cpu: 2 Core, memory: 2GB 이상 
- 운영체제: 리눅스 또는 Windows 등  
- Docker 프로그램
- Docker 서비스 실행
### 도커 설치
- 옵션이 여러개
  - 옵션 1: VirtualBox 설치(HyperVisor 프로그램)-네트워크 구성-가상 머신(VM) 만들기
    - 옵션 1-1: VM에 ``Ubuntu 20.04`` 설치하고 기본 환경 구성하기
    - 옵션 1-2: VM에 ``centOS`` 설치하고 기본 환경 구성하기
    - Ubuntu/CentOS Server에 Docker 설치하기 (실제 사이트에서는 Ubuntu와 CentOS의 비율이 비슷)
  - 옵션 2: Windows 10에 DockerDesktop 설치

### VirtualBox 설치, Network 구성 그리고, 가상머신 만들기
- Step 01: VirtualBox(Hypervisor의 일종) 다운로드 후 설치
  - https://www.virtualbox.org
- Step 02: Virbual Box의 Network 구성
  - NAT 네트워크 추가: 파일-환경설정-네트워크-추가
  - 네트워크 이름: localNetwork
  - 네트워크 CIDR: 10.100.0.0/24
  - DHCP 지원
  - 포트 포워딩
```bash
이름     프로토콜  호스트IP    호스트포트   게스트IP       게스트포트
docker1  TCP      127.0.0.1   105         10.100.0.105   22
docker1  TCP      127.0.0.1   106         10.100.0.106   22
```
- Step 03: 두 대의 가상머신 만들기
  - 이름: docker-ubuntu
    - CPU(2core), Memory(2GB), network(localNetwork), disk(20GB)
  - 이름: docker-centos
    - CPU(2core), Memory(2GB), network(localNetwork), disk(20GB)
  - **실제 OS Install이 되는 것은 아니고, OS 설치를 위한 가상머신이 생성 완료**
- Step 03-1: docker-ubuntu 설정
  - VirtualBox 왼쪽 메뉴의 도구 > 새로 만들기
    - 이름: docker-ubuntu
    - 머신 폴더: D:\VirtualBox VMs
    - 종류: Linux
    - 버전: Unbuntu (64-bit)
  - 메모리(4GB)와 Disk(20GB)를 제외하고 기본 설정으로 설치
  - 설치 완료 후, ``docker-ubuntu`` 선택 후, 설정 버튼 클릭
    - 시스템 > 프로세서: CPU 개수를 2개로 조정
    - 시스템 > 마더보드: 부팅 순서에 플로피 해제
    - 네트워크 > 어댑터 1
      - 다음에 연결됨: ``NAT``를 ``NAT 네트워크``로 변경      
      - 이름: ``docker-network`` 선택
- Step 03-2: docker-centos 설정
  - VirtualBox 왼쪽 메뉴의 도구 > 새로 만들기
    - 이름: docker-ubuntu
    - 머신 폴더: D:\VirtualBox VMs
    - 종류: Linux
    - 버전: Red Hat (64-bit)
  - 메모리(2GB)와 Disk(20GB)를 제외하고 기본 설정으로 설치
  - 설치 완료 후, ``docker-centos`` 선택 후, 설정 버튼 클릭
    - 시스템 > 프로세서: CPU 개수를 2개로 조정
    - 시스템 > 마더보드: 부팅 순서에 플로피 해제
    - 네트워크 > 어댑터 1
      - 다음에 연결됨: ``NAT``를 ``NAT 네트워크``로 변경      
      - 이름: ``docker-network`` 선택

### VM에 Ubuntu 20.04 설치하고 기본 환경 구성하기
#### Ubuntu 20.04 다운로드 및 설치
- 다운로드: https://ubuntu.com/download/desktop
  - Desktop version을 다운로드: Server Version은 GUI가 없음
  - 20.04.XX LTS(Long Term Support-10년)을 다운로드
- VirtualBox에서 해당 Virtual Machine를 선택
  - 저장소 > 광학드라이브 > 디스크 파일 선택 > Ubuntu 파일 선택
  - 시작 버튼을 누르면 설치가 시작됨
- 설치 순서
  - 언어 선택(한국어) > Ubuntu 설치 > 키보드 선택 > 일반 설치 > 새로 만든 디스크 지우고 설치
  - 지금 설치 > 포맷 > 계속 설치 > 타임존 설정 > 관리자 계정 생성:gusami/******
  - 당신은 누구십니까?
    - 이름: gusami
    - 컴퓨터 이름: docker-ubuntu.example.com
    - 사용자 이름: gusami
  - 설치 진행(약 20분 소요)
- 설치 완료되면 리부팅 진행
#### 설치 후 환경 구성
- GUI로그인: gusami/******
- 화면 우측 상단에 네트워크 마크를 클릭하면 설정으로 이동
  - 디스플레이 > 해상도를 1280 * 960으로 조정
- 서버 구성: 네트워크 설정 > IPv4 > 수동
  - 주소: 10.100.0.105, Netmaks: 24, 게이트웨이: 10.100.0.1 네임서버: 10.100.0.1
    - 터미널에서 ``ip addr`` 명령어를 이용해서 바뀐 주소 확인
  - hostname 변경
    - ``$vi /etc/hostname``에서 ``docker-Ubuntu.example.com``로 수정
  - /etc/hosts 파일에 호스트 정보 추가
```bash
$sudo vi /etc/hosts
127.0.0.1      localhost
10.100.0.105   docker-ubuntu.example.com    docker-ubuntu
10.100.0.106   docker-centos.example.com    docker-centos

# The following lines are desirable for IPv6 capable hosts
....
# Google DNS(8.8.8.8)를 PING 테스트를 진행해 봄
$ping -c 3 8.8.8.8
```  
- Text 부팅으로 수정: GUI 환경의 부팅은 리소스를 많이 먹고, 속도도 늦음
  - ``$systemctl set-default multi-user.target``
  - 만약, GUI 부팅으로 되돌리고 싶으면, ``$systemctl isolate graphical.target`` 수행
- root password 설정: 기본적으로 암호가 설정되어 있지 않아서 root 계정을 사용 못함
  - ``$sudo passwd root``
  - ``$su - root``: root 계정으로 전환
- SSH 서버 설치 후 운영
  - ``$apt-get update``
  - ``$apt-get install -y openssh-server curl vim tree``
  - 설치 후, ssh daemon 상태 체크
    - ``$systemctl status sshd``
  - Xshell로 로그인 구성
    - VirtualBox에서 설정된 포트 포워딩 규칙을 이용해서 ``localhost``의 105번 포트에 대한 연결을 XShell에 생성
  - 현재 OS 정보 확인: ``$cat /etc/os-relase``
  - 현재 시스템의 메모리 사용량 확인: ``$free -h``
  - 현재 OS의 커널 정보 학인: ``$uname -r``
- 현재까지 구성된 가상머신의 OS 상태를 저장하고 싶다면?
  - VirtualBox에서 해당 가상머신의 오른쪽의 햄버거 마크를 클릭
    - 스냅샷 > 현재 정보를 기록 > 찍기

### VM에 CentOS 7 설치하고 기본 환경 구성하기
#### CentOS 7 (Redhat 기반)다운로드 및 설치
- 다운로드: https://centos.org
- 설치 후 환경구성
  - 관리자 계정: gusami/e****3
  - root password: e*****3
  - 해상도 조절
  - 네트워크 구성: 10.100.0.106/24, GW: 10.100.0.1, DNS: 10.100.0.1
    - hostname: docker-centos.example.com
    - /etc/hosts 구성
  - text-login 구성
  - sshd 서비스 동작상태 확인
  - Xshell로 로그인 가능 여부 확인
- VirtualBox에서 해당 Virtual Machine를 선택
  - 설정 > 시스템 > 마더보드 > 기본 메모리를 4096MB로 설정
  - 설정 > 저장소 > 컨트롤러 IDE > 광학드라이브 > 디스크 파일 선택 > 다운받은 이미지 선택 > 확인
  - 시작 버튼을 누르면 설치가 시작됨
  - ``Install CentOS 7`` 선택 후 > Enter 키
- 설치 순서
  - 언어 선택(English-United States)
  - Date & Time Zone > Seoul 선택
  - Software Selection > GNOME Desktop : 만약, ``Minimal Install``을 선택하면, GUI가 뜨지 않음
  - Installtion Destination > Automatic Partitioning Selected 상태
  - Network & Host Name > Ethernet On: DHCP를 이용해서 IP가 자동 할당됨
    - Host name 변경: docker-centos.example.com
    - Configure > IPv4 Settings > Method를 Manual로 선택: 네트워크를 Static IP로 변경
      - Add: Address: 10.100.0.106, Netmask: 24, Gateway: 10.100.0.1
      - DNS servers: 10.100.0.1
      - Done 선택
  - ``Begin Installation`` 선택
    - 설치 중에 ``Root Password``를 선택해서 암호 설정: ``e******3``
    - 설치 중에 ``User Creation``를 선택해서 사용자 계정 생성 및 암호 설정: ``gusami/e******3``    
  - 설치 진행(약 20분 소요)
  - 설치 완료 후, ``Reboot`` 버튼 클릭 
  - Licensing > 클릭 후, ``I accept the license agreement``선택
    - ``Finish Configuration`` 선택
  - 로그인 화면 > ``gusami`` 아래의 ``not listed``를 선택하면, ``root``로 로그인 가능
    - ``root``로 로그인
    - CentOS는 root가 시스템관리자 계정으로 등록되어 있음
      - ubuntu는 gusami가 시스템 관리자 계정임
#### 기본 환경 구성 및 VM 네트워크 설정
- 화면 상단 왼쪽에 메뉴가 두 개 위치
  - Applications
  - Places
- 화면 상단 오른쪽에 위치한 전원버튼을 마우스 클릭  > 설정 버튼 클릭
  - Devices 선택 > Resolution > 1280 * 960 선택 > Apply
  - Region & Language > input source 아래에 위치한 ``+`` 버튼을 클릭 > Korean > Korean(hangul) 선택
    - 한글 입력 가능하게 됨
  - Privacy > Automatic Screen Lock > Off
  - Power > Blank screen > Never
  - Network > 설정 버튼 클릭
    - IP, Gateway, DNS 확인
    - Connect automatically > Checked    
- 화면 상단 왼쪽 ``Applications`` > ``System Tools`` > ``Terminal`` 선택
  - centOS는 가상 머신안에 또다른 Hypervisor가 존재: 삭제 처리해야 함. ``virbr0`` 항목
```bash
# CentOS
$ip addr
$systemctl stop libvirtd
$systemctl disable libvirtd
```
![hybervisor_in_centos](./images/hypervisor_in_centos.png)
- 네트워크 파일 확인
  - ``/etc/hostname`` 파일 확인
  - ``/etc/hosts`` 파일에 docker-ubuntu와 docker-centos 등록
```bash
$cat /etc/hostname
docker-centos.example.com
# hosts 파일에 docker-ubuntu와 docker-centos 등록
$vi /etc/hosts
.....
10.100.0.105  docker-ubuntu.example.com
10.100.0.106  docker-centos.example.com
# Google DNS(8.8.8.8)를 PING 테스트를 진행해 봄
$ping 8.8.8.8 -c 3
# text login
```
- Text 부팅으로 수정: GUI 환경의 부팅은 리소스를 많이 먹고, 속도도 늦음
  - ``$systemctl set-default multi-user.target``
  - 만약, GUI 부팅으로 되돌리고 싶으면, ``$systemctl isolate graphical.target`` 수행
- 기본적으로 SSHD(SSH Demon) 가 동작
```bash
$systemctl status sshd
```
- 기본적으로 ``curl`` 설치가 되어 있음
- ``tree``이 설치되어 있지 않음
```bash
$yum install -y tree
```
- 화면 상단 오른쪽에 power button을 눌러 Power off
- Xshell로 로그인 구성
    - VirtualBox에서 설정된 포트 포워딩 규칙을 이용해서 ``localhost``의 106번 포트에 대한 연결을 XShell에 생성
  - 현재 계정 정보 확인: ``$whoami``  
  - 현재 주소 확인: ``$ip addr``  
  - hostname 확인: ``$hostname``
  - 현재 OS 정보 확인: ``$cat /etc/os-relase``
  - 현재 시스템의 메모리 사용량 확인: ``$free -h``
  - 현재 OS의 커널 정보 학인: ``$uname -r``
- 현재까지 구성된 가상머신의 OS 상태를 저장하고 싶다면?
  - VirtualBox에서 해당 가상머신의 오른쪽의 햄버거 마크를 클릭
    - 스냅샷 > 현재 정보를 기록 > 찍기
### Ubuntu/CentOS Server에 Docker 설치하기
- CentOS와 Ubuntu에 Docker 설치: https://docs.docker.com
- 설치 방법
  - Repository를 이용해서 설치
  - Download 후 직접 설치
  - Script를 이용한 설치
- 설치 후, 동작 상태 확인
- 계정 추가
#### Ubuntu Sever에 Docker 설치하기
- Install Docker Engine on Ubuntu: https://docs.docker.com/engine/install/ubuntu/
- XShell 로그인
- 기존의 설치된 docker old 버전이 있다면 삭제하기
```bash
 $sudo apt-get remove docker docker-engine docker.io containerd runc
 ```
- 설치방법 01: Repository를 이용해서 설치 (우리가 사용)
  - docker를 install할 시스템이 외부 네트워크에 접근이 가능한 경우

![docker_install_by_repository.png](./images/docker_install_by_repository.png)
- 설치방법 02: Download 후 직접 설치
  - docker 프로그램을 다른 시스템에서 다운로드 받은 후, 복사해서 직접 설치
- 설치방법 03: Script를 이용한 설치
  - ``설치 방법 01``과 ``설치 방법 02``를 할 수 있도록 스크립트를 제공
##### Repository를 이용한 설치
- docker package repository의 URL을 설치하고자 하는 로컬시스템에 등록해줘야 함 
- Step 01: HTTP를 통해 docker package repository를 사용하기 위한 패키지 설치
```bash
$sudo apt-get update
$sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```    
- Step 02: 인증서를 로컬에 저장
  - docker package들은 인증서를 가지고 디지털 서명이 되어 있음
```bash
$curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```  
- Step 03: 다운받은 인증서를 이용해서 URL 등록
```bash
$echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```  
- Step 04: install with ``apt-get command``
  - 새로 등록된 URL을 이용하여 package index를 업데이트: ``sudo apt-get update``
```bash
$sudo apt-get update
# docker-ce: docker daemon
# docker-ce-cli: client command
# containerd.io: docker engine
$sudo apt-get install docker-ce docker-ce-cli containerd.io -y
 ```
- Step 05: 설치가 잘 되었는지 확인
```bash
# 방법1: hello-world image을 다운로드 한 후, 실행
$sudo docker run hello-world
# 방법2: docker version으로 확인하는 방법. client와 server의 모든 정보가 나와야 함
$sudo $ sudo docker version
Client: Docker Engine - Community
 Version:           20.10.11
 API version:       1.41
 Go version:        go1.16.9
 Git commit:        dea9396
 Built:             Thu Nov 18 00:37:06 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.11
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.9
  Git commit:       847da18
  Built:            Thu Nov 18 00:35:15 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
#### CentOS Sever에 Docker 설치하기
- Install Docker Engine on CentOS: https://docs.docker.com/engine/install/centos/
  - CentOS 7 or 8에서 설치 가능
- XShell 로그인
- 기존의 설치된 docker old 버전이 있다면 삭제하기
```bash
 $sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
 ```
- 설치방법 01: Repository를 이용해서 설치 (우리가 사용)
  - docker를 install할 시스템이 외부 네트워크에 접근이 가능한 경우
- 설치방법 02: RPM package Download 후 직접 설치
  - docker 프로그램을 다른 시스템에서 다운로드 받은 후, 복사해서 직접 설치
- 설치방법 03: Script를 이용한 설치
  - ``설치 방법 01``과 ``설치 방법 02``를 할 수 있도록 스크립트를 제공
##### Repository를 이용한 설치
- docker package repository의 URL을 설치하고자 하는 로컬시스템에 등록해줘야 함 
- CentOS는 **root 계정**으로 설치해줘야 함
- Step 01: HTTP를 통해 docker package repository를 사용하기 위한 패키지 설치   
```bash
$yum install -y yum-utils
```    
- Step 02: Repository URL을 로컬에 등록
```bash
$yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```  
- Step 03: install with ``yum install command``
  - 설치 시, 인증서를 가져옴 (Ubuntu와 차이점)
```bash
# docker-ce: docker daemon
# docker-ce-cli: client command
# containerd.io: docker engine
$yum install docker-ce docker-ce-cli containerd.io -y
 ```
- Step 04: docker service daemon 시작 및 활성화(Ubuntu와 차이점)
```bash
$systemctl start docker
$systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
# service enable 상태 확인: 다음 부팅시에도 서비스가 동작.(centOS에만 필요)
$systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-12-11 11:32:12 KST; 41min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 7457 (dockerd)
      Tasks: 9
     Memory: 30.6M
     CGroup: /system.slice/docker.service
             └─7457 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
- Step 05: 설치가 잘 되었는지 확인
```bash
# docker version으로 확인하는 방법. client와 server의 모든 정보가 나와야 함
$docker version
Client: Docker Engine - Community
 Version:           20.10.11
 API version:       1.41
 Go version:        go1.16.9
 Git commit:        dea9396
 Built:             Thu Nov 18 00:38:53 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.11
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.9
  Git commit:       847da18
  Built:            Thu Nov 18 00:37:17 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
### 계정에 Docker 관리자 권한주기 on Ubuntu and CentOS
- 설치 후, root계정만이 ``docker`` 명령어를 수행 가능
- ``root``로 로그인 후, ``docker`` 그룹에 사용자를 추가하면 됨
```bash
gusami@docker-ubuntu:~$su -
암호: 
root@docker-ubuntu:~#usermod -a -G docker gusami
root@docker-ubuntu:~#su - gusami
gusami@docker-ubuntu:~$docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
```bash
[gusami@docker-centos ~]$su -
Password: 
Last login: Sat Dec 11 11:52:41 KST 2021 on pts/0
[root@docker-centos ~]#usermod -a -G docker gusami
[root@docker-centos ~]#su - gusami
Last login: Sat Dec 11 10:55:28 KST 2021 from 10.100.0.2 on pts/0
[gusami@docker-centos ~]$docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
### Windows에 DockerDesktop 설치하기
- hub.docker.com 계정 등록 (gusami32/e******3)
  - docker container image가 등록된 Hub 사이트
- DockerDesktop 설치하기
  - Hyper-V 가상화 기능 활성화
  - WSL2(Windows Subsystem for Linux v.2)의 리눅스 커널 설치
    - DockerDesktop 설치 시, 자동 설치  
- Docker 동작 상태 확인
#### 다운로드 및 설치
- 설치 방법: https://docs.docker.com/desktop/windows/install/
- Download Docker Desktop for windows
- ``Docker Desktop Installer.exe`` 더블 클릭해서 실행
- 선행 조건
  - Enable Hyper-V Windows Features (bios에서 hyper-v로 설정되어 있어야 함)
  - hub.docker.com 계정으로 로그인 할 수 있어야 사용 가능 
- 명령어를 이용해서 Hyper-v 기능 on
  - open a cmd with admin privilige
  - turn on
    - bcdedit /set hypervisorlaunchtype auto
  - turn off
    - bcdedit /set hypervisorlaunchtype off
  - reboot
- 설치 후, Hyper-V와 WSL2 기능 활성화를 위해 윈도우 리부팅이 필요
- 만약에 필요하다는 메시지가 뜨면, WSL2 업데이트와 리눅스를 설치하는 방법
  - https://docs.microsoft.com/en-us/windows/wsl/install-manual 참조 
- 정상 설치 후, 윈도우 오른쪽 하단의 docker system tray가 생김
- PowerShell을 관리자 버전으로 실행
  - ``docker version``을 통해 버전 확인
```bash
PS C:\Windows\system32> docker version
Client:
 Cloud integration: v1.0.22
 Version:           20.10.11
 API version:       1.41
 Go version:        go1.16.10
 Git commit:        dea9396
 Built:             Thu Nov 18 00:42:51 2021
 OS/Arch:           windows/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.11
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.9
  Git commit:       847da18
  Built:            Thu Nov 18 00:35:39 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
- docker login후 사용 가능
```bash
PS C:\Windows\system32> docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: gusami32
Password:
Login Succeeded

Logging in with your password grants your terminal complete access to your account.
For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/
```
- docker image download
```bash
PS C:\Windows\system32> docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
e5ae68f74026: Pull complete
21e0df283cd6: Pull complete
ed835de16acd: Pull complete
881ff011f1c9: Pull complete
77700c52c969: Pull complete
44be98c0fab6: Pull complete
Digest: sha256:9522864dd661dcadfd9958f9e0de192a1fdda2c162a35668ab6ac42b465f0603
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```
- docker 이미지 실행
```bash
PS C:\Windows\system32> docker run -d -p 80:80 --name web nginx
67f84a250b844ad7abd0b795c53de1312667424ec42dcf32c94b35f4acbe3d03
```
- nginx 접속
```bash
PS C:\Windows\system32> curl http://localhost:80


StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>
                    <html>
                    <head>
                    <title>Welcome to nginx!</title>
                    <style>
                    html { color-scheme: light dark; }
                    body { width: 35em; margin: 0 auto;
                    font-family: Tahoma, Verdana, Arial, sans-serif; }
                    </style...
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Accept-Ranges: bytes
                    Content-Length: 615
                    Content-Type: text/html
                    Date: Sat, 11 Dec 2021 04:46:47 GMT
                    ETag: "61814ff2-267"
                    Last-Modified: Tue, 02 Nov 2021 ...
Forms             : {}
Headers           : {[Connection, keep-alive], [Accept-Ranges, bytes], [Content-Length, 615], [Content-Type, text/html]
                    ...}
Images            : {}
InputFields       : {}
Links             : {@{innerHTML=nginx.org; innerText=nginx.org; outerHTML=<A href="http://nginx.org/">nginx.org</A>; o
                    uterText=nginx.org; tagName=A; href=http://nginx.org/}, @{innerHTML=nginx.com; innerText=nginx.com;
                     outerHTML=<A href="http://nginx.com/">nginx.com</A>; outerText=nginx.com; tagName=A; href=http://n
                    ginx.com/}}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 615
```
- docker container 중지
```bash
PS C:\Windows\system32> docker rm -f web
web
```
- docker image 삭제
```bash
PS C:\Windows\system32> docker rmi nginx
Untagged: nginx:latest
Untagged: nginx@sha256:9522864dd661dcadfd9958f9e0de192a1fdda2c162a35668ab6ac42b465f0603
Deleted: sha256:f652ca386ed135a4cbe356333e08ef0816f81b2ac8d0619af01e2b256837ed3e
Deleted: sha256:a7edf84b6db27e8ef5d7368c95159120f00a74cee57368e2bc107ee713172699
Deleted: sha256:46893639b5fbc6315531cb197fb4071750508880425c0620d88aae4e483d72c1
Deleted: sha256:afa1ff13852cf0fa5d5ff8cd21c8a21a99c139fc069926cd1316d1ad3d0c7189
Deleted: sha256:831e8983bb6a65130bec4e73bed4bc641bfd6d7c6917be32a483882f70e809b0
Deleted: sha256:d213f9a0e4eef08107f70d94b36f5a41c1437fc68af82ee82eb74de80282130a
Deleted: sha256:9321ff862abbe8e1532076e5fdc932371eff562334ac86984a836d77dfb717f5
```
- docker image 목록 확인
```bash
PS C:\Windows\system32> docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
- docker desktop 실행 종료
  - system tray에서 마우스 우클릭 후, 종료 버튼을 누름
- docker desktop 서비스 중지
  - 윈도우 서비스에서 수동으로 전환 후, 정지 시킴

## 컨테이너와 컨테이너 이미지
### 컨테이너
- 컨테이너는 하나의 Application 프로세스
- 컨테이너는 완전히 독립된 공간에 분리되어서 운영됨
  - 별도의 CPU, Memory, Host name과 Network, Disk
  - User Id
- 컨테이너 이미지가 메모리에 로딩되어 실행되는 상태

![docker container](./images/docker_container.png)
- docker host
  - docker가 설치되어 docker daemon(dockerd)이 실행되는 리눅스 커널 시스템
### 컨테이너 이미지
- 컨테이너 이미지는 하드 디스크에 파일로 존재
- 만약, node js 기반에 app.js를 코딩해서 실행하고 싶다면?
  - 컨테이너 이미지는 여러 Layer로 구성됨
    - base image Layer
    - appjs source image Layer
    - ``run node app.js`` Layer
  
![docker container image](./images/docker_container_image.png)  
### 컨테이너 동작 방식
- Docker Host와 Docker Hub가 존재
- Docker Hub: Docker Container Image가 저장된 Repository
- 사용예: Docker Hub에서 nginx container image를 찾아라!!
```bash
$docker search nginx
```
- 사용예: Docker Hub에서 nginx container image를 최신본을 가져와라!!  
```bash
$docker pull nginx:latest
```
- 사용예: 다운받은 Docker nginx container image를 실행해서 컨테이너 생성해라!!
  - ``run, create, start`` 명령어  
```bash
$docker run -d --name web -p80:80 nginx:latest
```
### 용어 정리
- Docker Host (Linux Kernel)
- Docker Daemon: systemctl start docker
- Docker Client Command: docker
- Docker Hub
- Container Images
- Container

![docker_container_glossary](./images/docker_container_glossary.png)
### 실습
- docker daemon이 동작 중인지 확인
```bash
gusami@docker-ubuntu:~$sudo systemctl status docker
[sudo] gusami의 암호: 
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-12-15 22:29:23 KST; 3min 33s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 677 (dockerd)
      Tasks: 8
     Memory: 105.3M
     CGroup: /system.slice/docker.service
             └─677 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.980384435+09:00" level=warning msg="Your kernel does not sup>
12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.980425913+09:00" level=warning msg="Your kernel does not sup>
12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.980431473+09:00" level=warning msg="Your kernel does not sup>
12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.981084107+09:00" level=info msg="Loading containers: start."
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.215296025+09:00" level=info msg="Default bridge (docker0) is>
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.250771249+09:00" level=info msg="Loading containers: done."
lines 1-17...skipping...
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-12-15 22:29:23 KST; 3min 33s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 677 (dockerd)
      Tasks: 8
     Memory: 105.3M
     CGroup: /system.slice/docker.service
             └─677 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.980384435+09:00" level=warning msg="Your kernel does not sup>
12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.980425913+09:00" level=warning msg="Your kernel does not sup>
12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.980431473+09:00" level=warning msg="Your kernel does not sup>
12월 15 22:29:22 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:22.981084107+09:00" level=info msg="Loading containers: start."
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.215296025+09:00" level=info msg="Default bridge (docker0) is>
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.250771249+09:00" level=info msg="Loading containers: done."
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.314708665+09:00" level=info msg="Docker daemon" commit=847da>
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.315513164+09:00" level=info msg="Daemon has completed initia>
12월 15 22:29:23 docker-ubuntu.example.com systemd[1]: Started Docker Application Container Engine.
12월 15 22:29:23 docker-ubuntu.example.com dockerd[677]: time="2021-12-15T22:29:23.338471841+09:00" level=info msg="API listen on /run/docker.s>
```
- docker hub에서 nginx container 찾기
```bash
gusami@docker-ubuntu:~$docker search nginx
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                             Official build of Nginx.                        15972     [OK]       
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   2103                 [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   820                  [OK]
jc21/nginx-proxy-manager          Docker container for managing Nginx proxy ho…   293                  
linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   161                  
tiangolo/nginx-rtmp               Docker image with Nginx using the nginx-rtmp…   148                  [OK]
jlesage/nginx-proxy-manager       Docker container for Nginx Proxy Manager        147                  [OK]
alfg/nginx-rtmp                   NGINX, nginx-rtmp-module and FFmpeg from sou…   112                  [OK]
nginxdemos/hello                  NGINX webserver that serves a simple page co…   79                   [OK]
privatebin/nginx-fpm-alpine       PrivateBin running on an Nginx, php-fpm & Al…   61                   [OK]
nginx/nginx-ingress               NGINX and  NGINX Plus Ingress Controllers fo…   59                   
nginxinc/nginx-unprivileged       Unprivileged NGINX Dockerfiles                  56                   
nginxproxy/nginx-proxy            Automated Nginx reverse proxy for docker con…   31                   
staticfloat/nginx-certbot         Opinionated setup for automatic TLS certs lo…   25                   [OK]
nginx/nginx-prometheus-exporter   NGINX Prometheus Exporter for NGINX and NGIN…   22                   
schmunk42/nginx-redirect          A very simple container to redirect HTTP tra…   19                   [OK]
centos/nginx-112-centos7          Platform for running nginx 1.12 or building …   16                   
centos/nginx-18-centos7           Platform for running nginx 1.8 or building n…   13                   
flashspys/nginx-static            Super Lightweight Nginx Image                   11                   [OK]
bitwarden/nginx                   The Bitwarden nginx web server acting as a r…   11                   
mailu/nginx                       Mailu nginx frontend                            10                   [OK]
webdevops/nginx                   Nginx container                                 9                    [OK]
sophos/nginx-vts-exporter         Simple server that scrapes Nginx vts stats a…   7                    [OK]
ansibleplaybookbundle/nginx-apb   An APB to deploy NGINX                          3                    [OK]
wodby/nginx                       Generic nginx                                   1                    [OK]
```
- 다운로드된 docker image 확인
```bash
root@docker-ubuntu:/var/lib/docker/overlay2$docker images ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
- docker hub에서 nginx container image 다운로드
  - ``Pull complete`` 라인의 개수가 Layer 개수를 의미
  - Layer는 ``/var/lib/docker/overlay2`` 디렉토리의 파일로 존재
```bash
root@docker-ubuntu:/var/lib/docker/overlay2$docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
e5ae68f74026: Pull complete 
21e0df283cd6: Pull complete 
ed835de16acd: Pull complete 
881ff011f1c9: Pull complete 
77700c52c969: Pull complete 
44be98c0fab6: Pull complete 
Digest: sha256:9522864dd661dcadfd9958f9e0de192a1fdda2c162a35668ab6ac42b465f0603
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```
```bash
root@docker-ubuntu:/var/lib/docker/overlay2$ls -l
drwx--x--- 4 root root 4096 12월 16 22:10 4576390776f4daafd0e07a5b3a88bee2df4ec195e3598e6376df28d87c060d41
drwx--x--- 4 root root 4096 12월 16 22:10 6d4b7d6f9be0010639e125d174f623fd9f5972aacf8f8fe8af1bacde6648d4d9
drwx--x--- 4 root root 4096 12월 16 22:10 6e95d5578380d1d3b2291d1b00c3d17a6407d78a04488a276421ce0b00c415f9
drwx--x--- 3 root root 4096 12월 16 22:10 7da02fd3d3ba3d663cea7eb47a2efe23e0a62f902cc1e9586dde6f410c955ef2
drwx--x--- 4 root root 4096 12월 16 22:10 895a8b414b39370cf72fd748998cd1d11a55b08b95aa69d82e692787debd5202
drwx--x--- 4 root root 4096 12월 16 22:10 ce1c2b29c3602d08cc0b56f1805aeb22659b64d45ce58c703d2c781541451376
```
```bash
gusami@docker-ubuntu:~$docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    f652ca386ed1   2 weeks ago    141MB
```
- 컨테이너 실행하고 확인해 보기
  - ``docker run`` 명령어를 사용
```bash
# 실행시 컨테이너 ID를 결과로 보여줌
gusami@docker-ubuntu:~$ docker run --name web -d -p 80:80 nginx
6eb251c65633b8262f63ebd05d4bb63afc2f8a94a84835086b30cbe44253a801
```
- 현재 실행중인 도커 컨테이너 프로세스 보기
```bash
gusami@docker-ubuntu:~$docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                               NAMES
6eb251c65633   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp   web
```
- nginx에 접속해 보기
```bash
gusami@docker-ubuntu:~$ curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
- docker container 중지하기
```bash
gusami@docker-ubuntu:~$docker stop web
web
gusami@docker-ubuntu:~$docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
- docker container 지우기
  - docker container 이미지는 로컬에 그대로 존재
```bash
gusami@docker-ubuntu:~$docker rm web 
web
gusami@docker-ubuntu:~$docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
- docker container image 지우기
```bash
# image 지우기
gusami@docker-ubuntu:~$docker rm image nginx
# 디렉토리에서 확인
root@docker-ubuntu:/var/lib/docker/overlay2# ls -l
```