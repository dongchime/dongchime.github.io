---
layout: post
title:  "DMLab 클러스터 서버 구축해보기"
date:   2019-12-24 17:26:22 +0900
categories: jekyll update
---

### 1. 노동의 시작 (서버 10대를 7호관 서버실로 옮기기)
김인규 교수님께서 7년된 중고 서버 10대를 우리 DMLab에 주시기로 했다. 
2019년 12월 19일 목요일에 교수님, 연구실 후배와 서버 10대를 경영관에서 7호관 (약 3분거리)으로 옮겼다. 서버 10대, 모니터, 랙을 한번에 옮기려 하니 국민대에는 경사도 많고 계단도 많아 쉽지 않았다. 결국 서버 10대를 랙에서 빼내고 끌차에 올려 옮겼다. 경영관에서 7호관까지는 3분거리였는데 끌차를 이용해 엘리베이터가 있는 길을 찾아 옮기니 10분이 걸렸다. 그리고 다시 경영관으로 돌아와 모니터와 랙을 다시 끌차에 올려 7호관으로 가져오게 되었다. 굉장한 노동이였다! 손이 까맣게 되고 추워서 동상에 걸릴뻔 했지만 마지막에 맛있는 닭도리탕을 먹으니 다시 괜찮아졌다.

이 녀석이 DMLab 서버.<br />
![server-front](https://github.com/dongchime/dongchime.github.io/blob/master/assets/img/front.jpg?raw=true)   

### 2. 운영체제 설치
서버를 전부 7호관 서버실로 옮겨 설치한 후, 10대에 모두 Ubuntu 18.04.3 LTS를 설치해주었다.

### 3. 네트워크 설정
10대의 서버에 모두 우분투를 설치하고 네트워크를 설정해주었다.
![server-network](https://github.com/dongchime/dongchime.github.io/blob/master/assets/img/network.jpg?raw=true) <br />
과사무실에서 받은 ip 주소를 master server에 외부 인터넷 고정 ip 주소로 설정하고, 위 사진처럼 나머지 9대의 서버와의 통신을 위해 swich ip 주소를 입력해 주었다. 그리고 나머지 9대의 서버도 swich ip 주소를 입력하여 서로 통신이 되도록 하였다. 

사진 맨 아랫부분 랜선이 많이 꽂혀있는 것이 스위치이다.
![server-back](https://github.com/dongchime/dongchime.github.io/blob/master/assets/img/back.jpg?raw=true)

### 4. OpenSSH 설치
ip 주소를 알맞게 10대의 서버에 입력을 해주고, 다음에 해야할 것은 OpenSSH를 설치하여 원격으로 서버에 접속할 수 있도록 하는 것이다. master server는 외부 인터넷에 연결되어 있어서 OpenSSH를 간단하게 설치할 수 있었다.
- master server 의 터미널에서 openssh-server 설치
```bash
$ sudo apt update
$ sudo apt install openssh-server
```

나머지 서버들에는 usb에 OpenSSH 설치 파일을 미리 다운받아 놓고 각 서버에 하나씩 설치해주었다. 9대의 서버에 설치할 때에 dependency 문제때문에 까다로울 줄 알았는데 openssh-sftp-server, openssh-client만 따로 설치해주니까 금방 설치가 되었다.
- 나머지 9대의 서버에도 설치하기
usb에 openssh-server.deb, openssh-client.deb, openssh-sftp-server.deb 을 다운받아 9개의 서버에 하나씩 설치.
```bash
$ sudo dpkg -i openssh-sftp-server_7.9p1-10+deb10u1_amd64.deb
$ sudo dpkg -i openssh-client_7.9p1-10+deb10u1_amd64.deb
$ sudo dpkg -i openssh-server_7.6p1-4_amd64.deb
```

### 5. NFS 설치
10대의 서버가 서로 홈 디렉토리를 공유하게 하기 위해 NFS를 설치하게 되었다. 
- 홈 디렉토리를 실제로 소유하고 있는 master 서버에 NFS 서버를 설치한다. 
```bash
$ sudo apt install nfs-common nfs-kernel-server portmap
```

- 설치 후, /etc/exports 파일에 아래 내용 추가.
home 디렉토리에 ip주소의 최상위비트부터 24비트까지의 io주소를 가지면 접근할 수 있도록 한다는 뜻이다.
```
/home <swich ip주소>/24(rw,no_all_squash,sync,no_root_squash,no_subtree_check)
```

- 설정이 끝나면 NFS 서버를 재실행한다.
```bash
$ sudo systemctl restart nfs-server
```

클라이언트 서버 9대에도 NFS를 설치한다. 이 9대의 서버는 외부 인터넷 연결이 안되어있기 때문에 마스터 서버에서 미리 다운받아서 클라이언트 서버 9대에 파일을 보내서 설치한다.

- 마스터 서버에서 폴더를 하나 만들어서 거기에 필요한 설치파일을 모두 다운받아 놓는다.
필요한 것은 nfs-common.deb, libnfsidmap2.deb, libtirpc1.deb, rpcbind.deb, keyutils.deb
```bash
$ wget http://archive.ubuntu.com/ubuntu/pool/main/n/nfs-utils/nfs-common_1.3.4-2.1ubuntu5_amd64.deb
$ wget http://archive.ubuntu.com/ubuntu/pool/main/libn/libnfsidmap/libnfsidmap2_0.25-5.1_amd64.deb
$ wget http://archive.ubuntu.com/ubuntu/pool/main/libt/libtirpc/libtirpc1_0.2.5-1.2_amd64.deb
$ wget http://archive.ubuntu.com/ubuntu/pool/main/r/rpcbind/rpcbind_0.2.3-0.6_amd64.deb
$ wget http://archive.ubuntu.com/ubuntu/pool/main/k/keyutils/keyutils_1.5.9-9.2ubuntu2_amd64.deb
```

---------------------------------------------------
반복하기 

- 이제 이 디렉토리를 9대의 클라이언트 서버의 홈 디렉토리에 보낸다. 
```bash
$ rsync -avPd <디렉토리> <client서버>:~
```

- 디렉토리를 받은 클라이언트 서버로 접속해서 위에서 다운받은 것들을 설치해준다.
```bash
$ ssh <client서버>
$ cd <디렉토리>
$ sudo dpkg -i keyutils_1.5.9-9.2ubuntu2_amd64.deb
$ sudo dpkg -i rpcbind_0.2.3-0.6_amd64.deb
$ sudo dpkg -i libtirpc1_0.2.5-1.2_amd64.deb
$ sudo dpkg -i libnfsidmap2_0.25-5.1_amd64.deb
$ sudo dpkg -i nfs-common_1.3.4-2.1ubuntu5_amd64.deb
```

- /etc/fstab 파일에 다음을 추가해준다.
```bash 
마스터서버주소:/home /home nfs defaults 0 0
```

- 서버를 재부팅한다.
```bash
$ sudo reboot
```

- 01~09 서버까지 전부 위 작업을 반복한다.

------------------------------------------------------

### *비밀번호 없이 로그인하기
위와 같은 작업을 하다보니 다른 서버 접속시 계속 암호를 입력하기가 너무 귀찮았다. 그래서 암호를 입력하지 않고 로그인 할 수 있도록 설정을 해보았다. 

- 마스터 서버에서 rsa 암호화 방식으로 키를 생성한다.
```bash
$ ssh-keygen -t rsa
```

- authorized_keys(리모트 머신의 .ssh 디렉토리 아래에 위치하면서 id_rsa.pub 키의 값을 저장.)을 id_rsa.pub 파일에 덮어쓴다.
```bash
$ cat id_rsa.pub > authorized_keys 
```

이제 서버 접속을 했고, 다른 서버로 로그인 할 때 암호를 요구하지 않는다!

### 6. LDAP 설치 
<b>먼저 LDAP 서버를 0번 서버에 설치해준다.</b>

- 서버의 hostname 변경하기
```bash
$ sudo hostnamectl set-hostname <hostname>
$ echo "<ip address> <hostname>" | sudo tee -a /etc/hosts
```

- ldap 설치하기
```bash
sudo apt update
sudo apt -y install slapd ldap-utils
```
설치할 때 admin의 비밀번호 설정을 할 수 있다.

설치가 완료되면 `$ sudo slapcat` 으로 SLAPD의 데이터베이스 내용 확인을 할 수 있다.

- admin 계정 추가하기
```bash
$ sudo vim basedn.ldif
```
basedn.ldif 파일에 아래 내용 추가
```bash
dn: ou=Users,dc=<domain>,dc=co,dc=kr
objectClass: organizationalUnit
ou: Users
dn: ou=Groups,dc=<domain>,dc=co,dc=kr
objectClass: organizationalUnit
ou: Groups
```

아래 명령 실행해서 admin 계정 추가
```bash
$ sudo ldapadd -x -D cn=admin,dc=<domain>,dc=co,dc=kr -W -f basedn.ldif
```

<i>ldap 서버를 편리하게 관리하기 위해 phpLDAPAdmin을 설치한다.</i>

먼저 phpLDAPAdmin 설치를 위한 패키지들을 설치해준다.
```bash
$ sudo apt -y install apache2 php php-cgi libapache2-mod-php php-mbstring php-common php-pear
$ sudo a2enconf php7.2-cgi
$ sudo systemctl reload apache2
```

이제 phpLDAPAdmin을 다운받아 설치한다.
```bash
$ cd ~/Downloads
$ git clone https://github.com/breisig/phpLDAPadmin.git
$ sudo mv ~/Downloads/phpLDAPadmin /var/www/html/phpldapadmin
$ cd /var/www/html/phpldapadmin/config
$ sudo cp config.php.example config.php
```

config.php 파일을 열어 아래 내용을 수정한다. <br />
293번째 줄 : servers->setValue('server','host','<ip주소>'); <br />
300번째 줄 : servers->setValue('server','base',array('dc=<도메인>,dc=co,dc=kr')); <br />
335번째 줄 : servers->setValue('server','tls',false); <br />
453번째 줄 : servers->setValue('login','anon_bind',false); <br />

** 서버에 방화벽이 실향되고 있으면 80번, 443번 포트를 허용한다.
```bash
$ sudo ufw allow proto tcp from any to any port 80,443
```

이제 브라우저에서 phpLDAPAdmin에 접속 가능하다. <br />
http://<아이피주소>/phpldapadmin

admin으로 로그인 <br />
Login DN: cn=admin,dc=<도메인>,dc=co,dc=kr <br />
password: **********

로그인 후 dc에 password를 설정해준다.
dc트리에서 dc 선택한 후 Add new attribute에서 password 선택 후 Password를 설정한다.

-------------------------------------------------

<b>이제 0번부터 9번 서버에 LDAP Client를 설치해 준다.</b>
```bash
$ sudo apt-get update
$ sudo apt-get -y install libnss-ldap libpam-ldap ldap-utils nscd
```

첫번째 창에서 ldap://<1번 서버 ip주소> <br />
두번째 창에서 dc=<도메인>,dc=co,dc=kr <br />
세번째 창에서 LDAP version 선택 <br />
네번째 창에서 YES <br />
다섯번째 창에서 NO <br />
여섯번째 창에서 cn=admin,dc=<도메인>,dc=co,dc=kr <br />
일곱번째 창에서 비밀번호 설정 <br />

<i>configure authentication</i>
```bash
$ sudo vim /etc/nsswitch.conf
```

nssitch.conf 파일에 아래 내용 수정
```bash
passwd:         compat ldap
group:          compat ldap
shadow:         compat ldap
```

user의 홈 디렉토리 자동생성을 원하면 
```bash
$ sudo vim /etc/pam.d/common-session
```

common-session 파일에 맨아래에 아래 내용 추가
```bash
session required        pam_mkhomedir.so skel=/etc/skel umask=077
```

nscd 다시 시작
```bash
$ sudo service nscd restart
```








