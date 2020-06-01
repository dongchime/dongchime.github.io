---
layout: post
title:  "터미널에서 고정 ip 주소 설정하기"
date:   2020-06-01 20:14:22 +0900
categories: jekyll update
---

서버실에 직접 가기 싫어서 쓰는 "터미널에서 고정 ip 설정하는 방법"

### 환경
- ubuntu 18.04 LTS

### Open netplan file
```bash
$ sudo vim /etc/netplan/01-network-manager-all.yaml
```

### 01-network-manager-all.yaml 편집
/etc/netplan/01-network-manager-all.yaml 파일을 아래와 같이 수정한다.<br />
```bash
# Let NetworkManager manage all devices on this system
network:
  version: 2
  # renderer: NetworkManager
  ethernets:
    네트워크 드라이브:
      addresses: [xxx.xxx.x.xxx/xx]
      dhcp4: false
      gateway4: xxx.xxx.x.xxx
      nameservers:
        addresses: [x.x.x.x]
    네트워크 드라이브:
      addresses: [xxx.xxx.xxx.xx/xx]
      dhcp4: false
      gateway4: xxx.xxx.xxx.x
      nameservers:
        addresses: [xxx.xxx.xx.x]
```

### netplan 적용하기
```bash
$ sudo netplan apply
```

나는 위에 명령을 입력하고 아무 반응 없어서 그냥 두고 테스트해보니 설정한 ip 주소로 잘 변경되어 있어서 빠져나왔다. (ifconfig로 확인)