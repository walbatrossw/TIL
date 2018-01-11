# Windows10 & Ubuntu 16.04 LTS Dual Booting Setting & Dev

## 1\. Ubuntu 설치 공간 준비

- 디스크 관리에서 Ubuntu 설치를 위한 파티션 준비
- UEFI 펌웨어일 경우 빠른 시작 해제

## 2\. Ubuntu 설치

- 설치 진행
- 파티션 할당

  - swap : 크기는 램 용량에 따라 유연하게
  - home : 적당한 크기 할당
  - root : 나머지 용량 할당

# 3\. Ubuntu 16.04 LTS Setup After

- 설치 후 세팅을 위한 참고 블로그

  - http://programmingsummaries.tistory.com/389
  - http://all-record.tistory.com/category/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C%2C%20%EC%84%9C%EB%B2%84/%EB%A6%AC%EB%88%85%EC%8A%A4
  - http://jimnong.tistory.com/694?category=575588
- 우분투 wiki : <https://wiki.ubuntu-kr.org/index.php/Getting_Started>

# 4\. 개발 환경 세팅

- font : hack (http://sourcefoundry.org/hack/)

- java : 1.8 설치
  - java 버전 확인
    ```
    java -version
    javac -version
    ```
  - java 설치
    ```
    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo atp-get update
    $ sudo apt-get install oracle-java8-installer
    ```
  - java 환경변수 설정
    ```
    $ sudo vi /etc/profile
    ```
    ```
    export JAVA_HOME=/usr/lib/jvm/java-8-oracle
    exprot PATH=$JAVA_HOME/bin:$PATH
    export CLASS_PATH=$JAVA_HOME/lib:$CLASS_PATH
    ```

- dbms
  - mysql : 5.7
    - 설치
    ```
    $ sudo apt-get install mysql-server-5.7
    ```
    - workbench 설치
    ```
    $ sudo apt-get install mysql-workbench
    ```
  - h2

- apache-tomcat : tomcat 7 ~ 9

- IDE

  - IntelliJ : 2017.3
  - STS
  - Eclpse

- Script Editor

  - Atom

- git gui

  - gitKraken
