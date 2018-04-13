# 우분투 설치 및 설정

## 1. 설치 준비

- 디스크 관리에서 Ubuntu 설치를 위한 파티션 준비
- UEFI 펌웨어일 경우 빠른 시작 해제

## 2. 우분투 설치

#### # 윈도우 듀얼 부팅으로 설치
- 기타 설치 선택

#### # 파티션 설정
- SWAP 영역 : 램크기 만큼
- /(루트) 영역 : 원하는 크기 만큼
- /home 영역 : 원하는 크기 만큼

#### # 지역 설정
- 서울 선택

#### # 언어 선택
- 한국어 / 104키보드 설정

## 3. 설치 완료 후 우분투 설정 및 개발 환경 세팅

#### # 터미널 설정하기
- 터미널 크기 조정
- 색 : 투명도 조정
- 스크롤 막대 표시 제거

#### # Unity Tweak 설치 및 설정
##### 설치
```bash
$ sudo apt-get install unity-tweak-tool
```

##### 설정
- search 옵션 설정 해제
- panel 옵션 설정 : 날짜 시간, 사용자 이름

##### 테마, 아이콘 추가 설치
```bash
$ sudo add-apt-repository ppa:numix/ppa
$ sudo apt-get update
$ sudo apt-get install numix-*
```

```bash
$ sudo add-apt-repository ppa:moka/daily
$ sudo apt-get update
$ sudo apt-get install moka-icon-theme
```
#### # Plank 설치 및 설정
```
$ sudo add-apt-repository ppa:ritcotz/docky
$ sudo apt-get update
$ sudo apt-get install plank
$ sudo plank --preference
```
- Plank 설치 설정
```
$ sudo apt-get install gnome-tweak-tool
```
- 시작프로그램에 plank 등록하기 위해 gnome-tweak-tool 설치
- 시작프로그램에 등록

#### # 우분투 소프트웨어를 통해 유틸리티 및 개발도구 설치
- 클래식메뉴 인디케이터 설치 : 설치프로그램 메뉴
- Bluefish 설치 : 텍스트 에디터
- GIMP 설치 : 이미지 편집기
- Pinta 설치 : 윈도우 그림판과 유사한 기능
- FileZilla 설치
- Putty 설치
- IntelliJ 설치

#### # IntelliJ 설정
- ignore 플러그인 설치
- lombok 플러그인 설치
- material theme ui 플러그인 설치

#### # GDebi 설치 : deb설치 파일 인스톨을 위해
```bash
$ sudo apt-get install gdebi
```

#### # Chrome, Atom, GitKraken 설치
- Chrome(브라우저) : https://www.google.co.kr/chrome/index.html
- Atom(에디터) : https://atom.io/
- GitKraken(Git GUI) : https://www.gitkraken.com/

#### # Tomcat 설치
Tomcat을 다운받아 workspace에 버전별로 세팅
http://tomcat.apache.org/

#### # Java 설치 및 환경변수 설정
##### Java 설치
```bash
$ sudo apt-add-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```

##### 환경변수 설정
```bash
$ sudo gedit /etc/profile
```

```
# 환경변수
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib:$CLASS_PATH
```

```
$ source /etc/profile
```

#### # Mysql 5.7 설치 및 한글 인코딩 설정

##### Mysql, Workbench 설치
```bash
$ sudo apt-get install mysql-server-5.7
$ sudo apt-get install mysql-workbench
```

##### 한글 인코딩 설정
```bash
$ sudo gedit /etc/mysql/my.cnf
```
```
[client]
default-character-set = utf8

[mysqld]
character-set-client-handshake=FALSE
init_connect="SET collation_connection = utf8_general_ci"
init_connect="SET NAMES utf8"
character-set-server = utf8
collation-server = utf8_general_ci

[mysqldump]
default-character-set = utf8

[mysql]
default-character-set = utf8
```
```bash
$ sudo service mysql restart
```


#### # 개발용 폰트 설치
- Menlo, Monaco : https://github.com/ueaner/fonts
- Hack : https://fonts2u.com/hack-bold.font


## 참조 블로그
- http://programmingsummaries.tistory.com/389
- http://all-record.tistory.com/category/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C%2C%20%EC%84%9C%EB%B2%84/%EB%A6%AC%EB%88%85%EC%8A%A4
- http://jimnong.tistory.com/694?category=575588
- http://webdir.tistory.com/217
