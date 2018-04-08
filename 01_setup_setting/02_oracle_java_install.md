# 우분투에서 Oracle Java 설치하기

## 자바설치
```
$ sudo apt-add-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```

## 환경변수 설정
```
$ sudo nano /etc/profile
```

```
# 환경변수
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib:$CLASS_PATH
```
```
Ctrl+X → Y 입력 → 엔터 키
```

```
# profile reload
source /etc/profile
```
