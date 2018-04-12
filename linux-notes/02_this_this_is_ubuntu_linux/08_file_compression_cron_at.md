# 파일의 압축과 묶기

## 1. 파일 압축
- 압축파일의 확장명들 : `xz`, `bz2`, `gz`, `zip`, `Z`
- `xz`, `bz2`의 압축률이 더 좋음
- 파일 압축 관련 명령어 : 기존의 파일 자체를 압축, zip만 예외
  - `xz` : 확장명 `xz` 압축/해제
    ```
    # xz [파일명]
    # xz -d [파일명.xz]
    ```
  - `bzip2` : 확장명 `bz2` 압축/해제
    ```
    # bzip2 [파일명]
    # bzip2 -d [파일명.bz2]
    ```
  - `gzip` : 확장명 `gz` 압축/해제
    ```
    # gzip [파일명]
    # gzip -d [파일명.gz]
    ```
  - `zip/unzip` : 확장명 `zip` 압축/해제
    ```
    # zip [새로생성될파일이름.zip] [압축할파일이름]
    # unzip 압축파일이름.zip
    ```

## 2. 파일 묶기
- 리눅스(unix)에서는 파일압축과 파일묶기는 원칙적으로 별개의 프로그램으로 수행
- 파일 묶기 명령어는 `tar`이며, 묶인 파일의 확장자명도 `tar`
- 파일 묶기 관련 명령어
  - `tar` : 확장명 `tar`로 묶음 파일을 만들거나 풀어준다.
    - 동작 : `c`(묶기) / `x`(풀기) / `t`(경로확인)
    - 옵션 : `f`(파일) / `v`(과정보이기) / `J`(tar+xz) / `z`(tar+gzip) / `j`(tar+bzip2)
  - 사용예
    - 묶기
      ```
      # tar cvf [묶은파일명.tar] [묶을디렉토리]
      ```
    - 묶기 + xz압축
      ```
      # tar cvfj [묶은파일명.tar.xz] [묶을디렉토리]
      ```
    - tar 풀기
      ```
      # tar xvf [묶은파일명.tar]
      ```
    - xz 압축해제 + tar 풀기
      ```
      # tar xvfj [묶은파일명.tar.xz] [압축을 해제하여 놓을 디렉토리]
      ```

## 3. 파일 위치 찾기
- 기본파일 찾기
  ```
  # find [경로] [옵션] [조건] [action]
  ```
  - 옵션 : `-name`(파일명), `-user`(소유자), `-newer`(전, 후), `-perm`(허가권), `-size`(크기)
  - action : `-print`(디폴트), `-exec`(외부명령실행)
- PATH에 설정된 디렉토리만 검색
  ```
  # which [실행파일이름]
  ```
- 실행 파일, 소스, man페이지 파일까지 검색
  ```
  # whereis [실행파일이름]
  ```
- 파일 목록 데이터베이스에서 검색
  ```
  # locate [파일이름]
  ```

## 4. 시스템 설정
- 방화벽 설정 gui 설치
  ```
  # apt-get -y install gufw python-gi
  ```
- 방화벽 설정 gui 실행
  ```
  # gufw
  ```

## 5. CRON, AT
- cron : 주기적으로 반복되는 일을 자동적으로 실행될 수 있도록 설정
  - 관련된 데몬(서비스)은 `cron`
  - 관련 파일은 `/etc/crontab`
  - `/etc/crontab`형식
    - 분 시 일 월 요일 사용자 실행명령
    - 예)
      ```
      00 05 1 * * root cp -r /home /backup
      ```
    - 예제) 매월 15일 새벽 3시 1분에 /home 디렉토리와 그 하위디렉토리를 /backup 디렉토리에 백업
      - cron 서비스 상태 확인 : active (running)
        ```
        # systemctl status cron
        ```
      - `/etc/crontab`에서 수행할 작업 작성
        ```
        01 03 15 * * root | /root/myBackUp.sh
        ```
      - myBackUp.sh 생성, 작업 작성
        ```
        # cd
        # vi myBackUp.sh
        ```
        ```
        #!/bin/sh

        set $(date)
        fname="backup-$1$2$3tar.xz"

        tar cfJ /backup/$fname /home
        ```
      - 디렉토리 생성
        ```
        # mkdir /backup
        ```
      - cron서비스 재시작
        ```
        # systemctl restart cron
        ```
      - 테스트를 위해 시간변경 후 cron 서비스 재시작
        ```
        # date 011503002018
        # systemctl restart cron
        ```
      - 1분 뒤 백업여부 확인 : 그런데 안됨... vmware라서 그런지...ㅠ
        ```
        # ls -l /backup
        ```
- at : 일회성 작업을 예약
  - 예)
    - 예약
      ```
      # at [시간]
      # at 3:00am tomorrow
      # at now + 1 hours
      ```
    - `at>` 프롬프트에 예약 명령어 입력후 `Enter`
    - 완료되면 `Ctrl + D`
    - 확인
      ```
      # at -l
      ```
    - 취소
      ```
      # atrm [작업번호]
      ```
  - 예제) 내일 새벽 4시에 시스템 전체 업데이트
    - 패키지 설치
      ```
      # apt-get -y install rdate at
      ```
    - 현재 시간 설정
      ```
      # rdate -s time.bora.net
      ```
    - 작업 시간 설정
      ```
      # at 4:00 am tomorrow
      ```
    - 작업내용 설정, 설정완료후 `Crtl + D`로 빠져나옴
      ```
      at> apt-get -y upgrade
      at> reboot
      ```
    - 작업 확인
      ```
      # at -l
      ```
    - 지금 작성한 예약 작업을 삭제할 경우?
      ```
      # atrm 1
      ```
