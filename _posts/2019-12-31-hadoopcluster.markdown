---
layout: post
title:  "하둡 클러스터 설치하기"
date:   2019-12-31 16:21:22 +0900
categories: jekyll update
---

### 사전 준비사항
- openjdk 8 
- ldap 에 hadoop 그룹 추가, hadoop 계정 추가 (hadoop 계정을 이용해서 hadoop을 설치할것임)

### 1. HDFS로 사용할 sdb 하드디스크 포맷, 마운트하기
모든 서버에 적용한다. <br />

<b>파티션 만들기</b>
```bash
$ fdisk /dev/sdb
```
현재 마운트 되어있는 것 모두 삭제 (d) <br />
새 파티션 생성 (n) <br />
변경사항 반영 (w) <br />

<b>포맷하기</b>
```bash
$ mkfs.ext4 /dev/sdb1
```

<b>마운트하기</b><br />
root 디렉토리에 data1 디렉토리를 생성해서 거기에 마운트를 해줄것이다. <br />

```bash
$ mkdir /data1
$ mount /dev/sdb1 /data1
$ df -T
```

** 부팅 시 자동으로 마운트 되도록 하기 <br />
/etc/fstab 파일에 아래 내용 추가<br />
```bash
/dev/sdb1 /data1 ext4 defaults 0 0
```

### 2. 시스템 설정하기
<b>hosts 파일 수정하기</b> <br />
/etc/hosts 파일을 아래와 같이 수정한다.<br />
```bash
192.0.2.1    node-master
192.0.2.2    node1
192.0.2.3    node2
```

<b>hadoop 계정에서 비밀번호 없이 로그인하기</b><br />
홈 디렉토리로 가서 <br />
```bash
$ ssh-keygen -t rsa
$ cd .ssh
$ cat id_rsa.pub > authorized_keys 
```

<b>hadoop binary 파일 다운받기</b><br />
/opt/hadoop 에서 설치할 것이다.
```bash
wget https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-x.x.x/hadoop-x.x.x.tar.gz
$ tar -xzf hadoop-x.x.x.tar.gz
$ mv hadoop-x.x.x hadoop
```

<b>환경변수 설정</b><br />
/opt/hadoop/.profile 파일에 아래 내용 추가<br />
```bash
PATH=/opt/hadoop/bin:/opt/hadoop/sbin:$PATH
```
.bashrc 파일에 아래 내용 추가<br />
```bash
export HADOOP_HOME=/opt/hadoop
export PATH=${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
```

### 3. Master Node 설정하기
<b>자바 환경변수 설정</b><br />
/opt/hadoop/etc/hadoop/hadoop-env.sh 파일에 아래 내용 추가<br />
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
```

<b>NameNode Location 설정</b><br />
/opt/hadoop/etc/hadoop/core-site.xml 파일에 아래 내용 추가<br />
```bash
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://<ip주소>:9000</value>
        </property>
    </configuration>
```

<b>HDFS path 설정</b><br />
/opt/hadoop/etc/hadoop/hdfs-site.xml 파일에 아래 내용 추가<br />
```bash
<configuration>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>/data1/nameNode</value>
    </property>
    <property>
            <name>dfs.datanode.data.dir</name>
            <value>/data1/dataNode</value>
    </property>
    <property>
            <name>dfs.replication</name>
            <value>2</value>
    </property>
</configuration>
```

<b>YARN 설정</b><br />
opt/hadoop/etc/hadoop/mapred-site.xml 파일에 아래 내용 추가<br />
```bash
<configuration>
    <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
    </property>
    <property>
            <name>yarn.app.mapreduce.am.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
            <name>mapreduce.map.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
            <name>mapreduce.reduce.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
</configuration>
```

opt/hadoop/etc/hadoop/yarn-site.xml 파일에 아래 내용 추가 <br />
```bash
<configuration>
    <property>
            <name>yarn.acl.enable</name>
            <value>0</value>
    </property>
    <property>
            <name>yarn.resourcemanager.hostname</name>
            <value><마스터노드 ip 주소></value>
    </property>
    <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

<b>Workers 설정</b><br />
opt/hadoop/etc/hadoop/workers 파일에 아래 내용 추가 <br />
```bash
node1
node2
...
```

### 4. Config 파일을 각 노드로 복사하기
```bash
$ cd /opt
$ rsycn -avPd opt/* node1:/opt
$ rsycn -avPd opt/* node2:/opt
```

### 5. HDFS 포맷
master node에서 <br />
```bash
$ hdfs namenode -format
```

### 6. HDFS 실행 및 모니터링
master node에서 <br />
HDFS 시작<br />
```bash
$ start-dfs.sh
```
HDFS 모니터링<br />
http://<마스터노드 ip주소>:9870 접속 <br /><br />

HDFS 끄기 <br />
```bash
$ stop-dfs.sh
```

### 7. YARN 실행 및 모니터링
master node에서 <br />
YARN 시작<br />
```bash
$ start-yarn.sh
```
YARN 모니터링<br />
http://<마스터노드 ip주소>:8088 접속 <br /><br />

YARN 끄기 <br />
```bash
$ stop-yarn.sh
```

### 8. hadoop 돌려보기
IntelliJ 에서 짰던 WordCount 코드를 돌려보려고 한다.
- 프로젝트 디렉토리에서 mvn install
- target 디렉토리에 test-1.0-SNAPSHOT.jar 파일이 생긴다.
- jar 파일을 hadoop 계정으로 가져온다.
- `hadoop jar test-1.0-SNAPSHOT.jar package1.WordCount -Dmapred.reduce.tasks=2 test.txt` 실행
- 프로세스는 8088에서 볼 수 있다.
