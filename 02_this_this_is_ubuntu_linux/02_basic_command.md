# 필수 개념 및 명령어

## 프롬프트
- `#` : root 사용자
- `$` : 일반사용자

## 시작과 종료
- 종료
  ```
  $ power off
  $ shutdown -P now
  $ halt -p
  $ init 0
  ```
- 시스템 재시작
  ```
  $ shutdown -r now
  $ reboot
  $ init 6
  ```
- 로그아웃
  ```
  $ logout
  $ exit
  ```

## 가상콘솔
- 일종의 가상의 모니터, 우분투는 총 7개의 모니터를 제공

## Runlevel
- 'init' 명령어 뒤에 붙는 숫자를 런레벨(Runlevel)이라 함.
    - `$ init 0` : 종료
    - `$ init 1` : 시스템 복구모드
    - `$ init 2` : 사용X
    - `$ init 3` : 텍스트 모드의 다중 사용자 모드
    - `$ init 4` : 사용X
    - `$ init 5` : 그래픽 모드의 다중 사용자 모드
    - `$ init 6` : 재시작

- Runlevel 변경해보기 : 그랙픽모드 -> 텍스트 모드
  ```
  ls -l /lib/systemd/system/default.target
  ```
  ```
  ln -sf /lib/systemd/system/multu-user.target /lib/systemd/system/default.target
  ```
  ```
  ls -l /lib/systemd/system/default.target
  ```

## 자동 완성, 히스토리, 디렉토리 이동
- 파일명이나 디렉토리의 일부만 작성하고 `tab`키를 이용하여 자동완성할 수 있음.
- 빠른입력뿐만아니라 정확한 타이핑이 가능하기 때문에 매우 유용
- 이전에 작성한 명령키는 위아래 화살표키를 이용하여 다시 사용할수 있음.
- `history`명령어를 입력하면 그동안 작성한 명령어들이 나온다.
- `history -c` : 그동안 작성한 명령어를 삭제
- `cd [해당 디렉토리 경로]` : 해당 디렉토리로 이동
- `pwd` : 현재 디렉토리 경로를 알려줌
- `cat [파일명]` : 해당 파일의 내용 출력
