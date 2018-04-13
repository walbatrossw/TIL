# RAID에 우분투 설치하기

## 1. VMware에서 새로운 가상머신 추가
- Ubuntu 64bit 운영체제 선택
- 80GB 하드디스크 2개로 설정
- Ubuntu-16.04-server-amd64 설치

## 2. 설치 시 파티션 디스크 설정
- `Partition disks`에서 `Munual`선택
  - `sda` 첫번째 디스크 선택
    - `FREE SPACE` 선택
    - `Create a new Partition` 선택
    - 용량은 4GB로 설정하고 디스크 타입은 `Primary`로 선택
    - `Beginning` 선택
    - `Use as`를 `do not use the partition`선택하고 `Go Back`
    - `FREE SPACE`를 선택해 나머지 공간도 `do not use the partition`으로 동일하게 설정
  - `sdb` 두번째 디스크도 동일하게 설정
- SWAP영역 RAID 설정
  - `Partition disks`에서 `Configure software RAID` 선택 후 `Yes`
  - `Create MD device`를 선택한 뒤 `RAID1`선택 장치의 갯수는 2, 예비 갯수는 0
  - SWAP영역을 설정하기 위해 4GB로 나눈 `/dev/sda1`, `/dev/sdb1`를 선택
- /영역 RAID 설정
  - 위와 같이 동일하게 설정
- 이전에 `do not use the partition`선택했던 것들을 각각 `swap area`와 `Ext4 journaling file system`(Mount point: `/`)으로 선택하고 설치 진행
- 설치가 완료되고 부팅

## 3. 설치 완료 후 확인
- 로그인하고 mdadm명령어로 파티션 확인
  ```
  $ sudo mdadm --detail --scan
  $ sudo mdadm --detail /dev/md0
  $ sudo mdadm --detail /dev/md1
  ```

## 4. 하드디스크 손상시 복구를 위한 예비 하드디스크 설정
- VMware에서 하드디스크(80GB)를 추가
- 부팅 후 하드디스크 추가 확인
  ```
  $ ls -l /dev/sd*
  ```
- 새로 추가한 하드디스크에 동일한 파티션을 복제
  ```
  $ sudo sfdisk -d /dev/sda | sudo sfdisk /dev/sdc
  ```
- 복제된 하드디스크 파티션 확인
  ```
  $ sudo fdisk -l /dev/sda /dev/sdc
  ```

## 5. 복구
- VMware에서 SCSI 0:0인 하드디스크를 제거하고 부팅
- 정상적인 부팅이 진행되지않고 `(initramfs)`라는 프롬프트가 뜸
  - `ls /dev/sd*`명령어를 치면 `/dev/sda`, `/dev/sdb`가 나오는데 제거된 디스크가 나오는것이 아니라 남아있는 디스크들의 이름들이 당겨져서 나오는 것
  - `mdadm` 명령어로 확인해보면 상태가 `INACTIVE-ARRAY`상태라는 것을 확인할 수 있음
    ```
    mdadm --detail --scan
    ```
  - `mdadm` 명령어로 강제로 작동할 수 있도록 설정
    ```
    mdadm --run /dev/md0
    mdadm --run /dev/md1
    ```
  - 예비 하드디스크에 복구작업 진행
    ```
    mdadm /dev/md0 --add /dev/sdb1
    ```
    ```
    mdadm /dev/md0 --add /dev/sdb2
    ```
    > recovery done 메시지가 뜰때까지는 사용하거나 전원을 끄지않고 기다려야한다.
