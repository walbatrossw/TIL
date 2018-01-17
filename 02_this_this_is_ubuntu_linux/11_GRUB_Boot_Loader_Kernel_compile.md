# GRUB 부트로더

## 1. GRUB 부트로더의 특징
- 부트 정보를 사용자가 임의로 변경해 부팅할 수가 있다. 부트정보가 올바르지 않더라도 수정하여 부팅할 수 있다.
- 다른 여러가지 운영체제와 멀티부팅을 할 수 있다.
- 대화형 설정을 제공해줘 커널의 경로와 파일 이름만 알면 부팅이 가능하다.

## 2. GRUB2의 장점
- 셸 스크립트를 지원함으로써 조건식과 함수를 사용할 수 있다.
- 동적 모듈을 로드할 수 있다.
- 그래픽 부트 메뉴를 지원하며, 부트 스플래시(boot splash)성능이 개선되었다.
- ISO이미지를 이용해서 부팅할 수 있다.

## 3. GRUB2 설정방법
- `/boot/grub/grub.cfg` 설정파일(직접 변경해서는 안됨)
- `/etc/default/grub` 파일과 `/etc/grub.d/` 디렉토리의 파일을 수정한 후에 `grub-mkconfig`명령어를 실행해 설정한다.

## 4. GRUB 부트로더 변경
- 설정파일 수정
  ```
  # vi /etc/default/grub
  ```
  ```
  GRUB_DEFAULT=0
  #GRUB_HIDDEN_TIMEOUT=0 [주석처리]
  GRUB_HIDDEN_TIMEOUT_QUIET=true
  GRUB_TIMEOUT=20 [시간수정]
  GRUB_DISTRIBUTOR="THIS IS LINUX" [문자열변경]
  GRUB_CMDLINE_LINUX_DEFAULT=""
  GRUB_CMDLINE_LINUX=""
  ```
  ```
  # update-grub
  ```
- 부트로더에서 root 비밀번호를 아무나 임의로 변경할 수 없도록하기
  ```
  # vi /etc/grub.d/00_header
  ```
  ```
  cat << EOF
  set supersuers="아이디"
  password 아이디 비밀번호
  EOF
  ```
  ```
  update-grub
  ```
---

# 커널

## 1. 모듈의 개념과 커널 컴파일의 필요성
- 모듈 : 필요할 때마다 호출하여 사용되는 코드
- 커널 컴파일이 필요한 이유는?
  - 커널의 기능이 점점 많아지고 무거워짐에 따라 평소에 쓰지않는 기능의 경우 필요할 때마다 로딩할 수 있도록 모듈로 빼놓고 컴파일할 수 있게 하는 것
## 2. 커널 컴파일
- 굳이 커널 컴파일을 하지 않아도 무방하다.
- 커널 컴파일 순서
  1. 현 커널 버전 확인
      ```
      # uname -r
      ```
  2. 커널 소스 다운로드 : www.kernel.org
      ```
      # mv linux-4.7.tar.xz /usr/src
      # cd /usr/src
      ```
  3. 커널 소스 압축풀기
      ```
      # tar xfJ linux-4.7.tar.xz
      # cd linux-4.7
      ```
  4. 커널 설정 초기화
      ```
      초기화 관련된 패키지 설치
      # apt-get -y install qt5-default libssl-dev
      ```
      ```
      # make mrproper
      ```
  5. 커널 환경 설정
      ```
      # make xconfig
      ```
      - NTFS USB 사용하기 설정
        - DOS/FAT/NT Filesystems
          - NTFS file system support
            - NTFS debugging support 체크
            - NTFS write support 체크
  6. 이전 정보 삭제
      ```
      # make clean
      ```
  7. 커널 컴파일 및 설치
      ```
      # make
      # make modules_install
      # make install
      # ls -l boot
      ```
  8. 부트 로더 확인
      ```
      # cat /boot/grub/grub.cfg
      ```
