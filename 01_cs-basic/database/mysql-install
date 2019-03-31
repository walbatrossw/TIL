# MySQL 설치 및 설정

## 1. Mac OS에서 설치

Mac OS에서 설치할 경우 설치화면에서 나타나는 임시 비밀번호 기억해둘것

## 2. Mac OS에서 구동

시스템 환경설정 > MySQL > Start MySQL Server 클릭

## 3. 비밀번호 재설정

```bash
cd /usr/local/mysql/bin
sudo ./mysql -p
SET PASSWORD = PASSWORD('root');
flush privileges;
```

## 4. UTF-8 설정

```bash
vi /etc/mysql/my.cnf
```

```
[client]
default-character-set=utf8  
[mysql]
default-character-set=utf8  
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```
