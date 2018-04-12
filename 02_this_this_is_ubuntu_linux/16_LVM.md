# LVM(Logical Volume Manage)

## 1. LVM이란?
- 주요기능
  - 여러 개의 하드디스크를 합쳐서 한 개의 파일시스템으로 사용하는 것으로 필요에 따라 다시 나눌 수 있다.
  - 예를 들어 2TB의 하드디스크를 2개로 합쳐 4TB로 만든 뒤 다시 3TB와 1TB로 나눌 수가 있다.
- 용어
  - Phsical Volume(물리볼륨) : /dev/sda1, /dev/sdb1 등의 파티션을 의미
  - Volume Group(볼륨그룹) : 물리볼륨을 합쳐 1개의 물리그룹으로 만든 것
  - Logical Volume(논리볼륨) : 볼륨 그룹을 1개 이상으로 나누어 논리 그룹으로 나눈 것

- LVM 구현
  - LVM 구현 예시
  ![LVM](http://cfile28.uf.tistory.com/image/991F893E5A61A8A3075812)
  - 구현 흐름
    1. 선처리작업 : VMware에 하드디스크 추가(3TB, 2TB)
        - 파티션 작업
          ```
          # fdisk /dev/sdb
          n : 파티션생성
          p : 주디스크 설정
          Enter : 파티션 넘버 설정(default)
          Enter : 파티션 시작점 설정(default)
          Enter : 파티션 끝점 설정(default)
          t : 파티션 타입 설정
          8e : Linux LVM으로 설정
          w : 쓰기 설정
          ```
        - 물리볼륨 생성
          ```
          관련패키지 설치
          apt-get -y install lvm2
          ```
          ```
          pvcreate /dev/sdb1
          pvcreate /dev/sdc1
          ```
    2. 볼륨그룹 생성
      ```
      # vgcreate myVG /dev/sdb1 /dev/sdc1
      ```
    3. 볼륨그룹 보기
      ```
      # vgdisplay
      ```
    4. 볼륨그룹을 논리불륨으로 분할 : 1TB/3TB/1TB
      ```
      설정한 크기에 따라 분할
      # lvcreate --size [분할할크기] --name [분할할 논리볼륨명] [볼륨그룹명]
      # lvcreate --size 1G --name myLG1 myVG
      # lvcreate --size 3G --name myLG2 myVG

      남은 용량을 다 사용
      # lvcreate --extents 100%FREE --name [분할할 논리볼륨명] [볼륨그룹명]
      # lvcreate --extents 100%FREE --name myLG3 myVG
      ```
    5. 각각의 논리볼륨을 파일시스템 생성(포맷)
      ```
      # mkfs.ext4 /dev/myVG/myLG1
      # mkfs.ext4 /dev/myVG/myLG2
      # mkfs.ext4 /dev/myVG/myLG3
      ```
    6. Mount
      ```
      # mkdir /lvm1 /lvm2 /lvm3
      # mount /dev/myVG/myLG1 /lvm1
      # mount /dev/myVG/myLG2 /lvm2
      # mount /dev/myVG/myLG3 /lvm3
      ```
    7. 리부팅시 Mount해제방지를 위한 설정 : `/etc/fstab`수정
      ```
      # gedit /etc/fstab
      ```
