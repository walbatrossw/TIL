# 사용자별 공간 할당 - 쿼터(Quota)

## 쿼터란?
- 파일시스템마다 사용자나 그룹이 생성할 수 있는 파일의 용량 및 갯수를 제한하는 것
- 파일시스템을 `/`로 지정하는 것보다는, 별도의 파일시스템을 지정해서 해당부분을 쓰도록 하는 것이 좋다.
- `/`파일시스템을 많은 사용자가 동시에 사용하게 되면, 우분투 서버를 운영하기 위해서 디스크를 읽고 쓰는 작업이 동시에 발생하므로 전반적인 시스템 성능이 저하된다.

## 쿼터실습
- 사용자를 만들고, 해당 사용자에게 공간을 할당
- 쿼터의 설정 및 작동에 대해 알아보기
- 실습 진행 흐름
  1. `/etc.fstab` 수정
  2. reboot or remounting
  3. 쿼터 DB생성
  4. 개인별 쿼터 설정
- 실습
  - VMware에서 Ubuntu 가상머신에 하드디스크(1GB) 1개 추가
  - 부팅후 새로 추가한 하드디스크 파티션 및 파일시스템 설정, 마운트까지 진행
    ```
    파티션 설정
    # fdisk /dev/sdb
    n
    p
    Enter
    Enter
    Enter
    p
    w
    ```
    ```
    파일시스템 설정
    # mkfs.ext4 /dev/sdb1
    ```
    ```
    마운트를 위한 디렉토리 생성
    # mkdir /userHome
    ```
    ```
    마운트
    # mount /dev/sdb1 /userHome
    ```
    ```
    리부팅시 마운트 해제방지 설정
    # vi /etc/fstab
    /dev/sdb1 /userHome ext4  defaults  0 0
    ```
  - 새로운 사용자 2명 추가, 새로 추가한 하드디스크에 홈디렉토리 설정
    ```
    # adduser --home [추가한 디스크의 사용자경로] [사용자명]
    # adduser --home /userHome/john john
    # adduser --home /userHome/dann dann
    ```
    ```
    추가한 사용자 디렉토리 확인
    # ls -l /userHome
    ```
  - 추가한 하드디스크를 쿼터형이라는 것을 지정하기 위한 설정
    ```
    # vi /etc/fstab
    /dev/sdb1 /userHome ext4  defaults,usrjquota=aquota.user, jqfmt=vfsv0   0 0
    ```
  - 쿼터 설치
    ```
    # apt-get -y install quota
    ```
  - 쿼터DB 생성
    ```
    디렉토리 이동
    # cd /userHome
    ```
    ```
    quota 종료
    # quotaoff -avug
    ```
    ```
    quota 체크 : quota.user 또는 quota.group를 최근상태로 업데이트
    # quotacheck -augmn
    ```
    ```
    quota 파일 삭제
    # rm -f aquota.*
    ```
    ```
    quota 체크 : quota.user 또는 quota.group를 최근상태로 업데이트
    # quotacheck -augmn
    ```
    ```
    quota 파일 생성
    # touch aquota.user aquota.group
    ```
    ```
    quota 파일 권한 변경
    # chmod 600 aquota.*
    ```
    ```
    quota 체크 : quota.user 또는 quota.group를 최근상태로 업데이트
    # quotacheck -augmn
    ```
    ```
    quota 시작
    # quotaon -avug
    ```
    ```
    사용자 쿼터용량 제한 설정
    # edquota -u [사용자]
    # edquota -u john
    ```
    - 내부 속성
      ```
      bolck만 수정
      /dev/sdb1 28 10240 15360 5 0 0
      ```
      - Filesystem(디렉토리)
      - block(크기)
        - soft(제한량) : 크기가 넘어가더라도 경고만
        - hard(제한량) : 제한
      - inode(파일갯수)
        - soft(제한량) : 갯수가 넘어가더라도 경고만
        - hard(제한량) : 제한
  - 쿼터 적용 확인해보기
    ```
    사용자로 접속
    # su - john
    ```
    ```
    사용자 확인
    $ whoami
    ```
    ```
    현재 디렉토리 확인
    $ pwd
    ```
    ```
    파일 복사해보기 : 허용된 용량이 넘어갈 경우, 초과되었다면 메시지와 허용된 용량만큼 파일이 잘려서 복사된다.
    $ cp [파일] test1
    $ cp [파일] test2
    $ cp [파일] test3
    ```
    ```
    quota 확인 명령어 : 자신에게 할당된 quota 확인
    $ quota
    ```
  - 전체 사용자의 quota 확인 : root사용자에서 확인가능
    ```
    # repquota [디렉토리]
    # repquota /userHome
    ```
  - 특정 사용자의 quota를 동일하게 다른 사용자에게 적용하기
    ```
    # edquota -p [특정사용자] [다른사용자]
    # edquota -p john bann
    ```
