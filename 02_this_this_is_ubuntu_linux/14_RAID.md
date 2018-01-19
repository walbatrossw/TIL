# RAID

## 1. RAID 정의
- RAID(Redundant Array of Inexpensive Disks)는 여러 개의 디스크를 하나의 디스크처럼 사용
- 비용절감 + 신뢰성 향상 + 성능 향상의 효과

## 2. 하드웨어 RAID
- 하드웨어 제조업체에서 여러 개의 하드디스크를 가지고 장비를 만들어 그자체를 공급
- 좀더 안정적이나 상당히 고가

## 3. 소프트웨어 RAID
- 고가의 하드웨어 RAID의 대안
- 운영체제에서 지원하는 방식
- 저렴한 비용으로 좀더 안전한 데이터의 저장이 가능

## 4. 각 RAID방식의 비교
![RAID방식비교](http://cfile26.uf.tistory.com/image/990548465A5F2814295707)
- **단순 볼륨**
  - 사용량: 1T(N)
  - 1개 하드디스크만
- **Linear RAID**
  - 사용량: 2T(N)
  - 최소 2개 이상 하드디스크 필요
  - 1번째 디스크부터 차례로 저장
  - 디스크 추가 가능
  - 100% 공간 효율
  - 신뢰성이 낮다.
  - 디스크 크기가 다르더라도 공간효율이 좋다
- **RAID0**
  - 사용량: 2T(N)
  - 최소 2개 이상의 하드디스크 필요
  - 모든 디스크에 동시 저장(나누어서 저장)
  - 가장 빠름(신뢰성이 낮다) : 손실되도 상관없는 데이터를 저장하는데 적합
  - Stripping
  - 결함 허용(Fault-tolerance) 제공하지 않는다.
- **RAID1**
  - 사용량: 1T(N/2), 데이터의 두 배의 용량이 필요
  - 2개
  - 동시저장(동일한 데이터를 똑같이 저장)
  - 신뢰성 높음 : 중요한 데이터 저장에 적합
  - 공간효율 낮음(비용이 많이듬)
  - 결함 허용(Fault-tolerance) 제공
  - Mirroring
- **RAID5**
  - RAID1의 안전성 + RAID0의 효율성
  - 사용량: 2T(N-1)
  - 최소 3개 이상의 디스크 필요
  - 결함 허용(Fault-tolerance) 제공(2개이상이 이상 발생시 결함 허용 제공X)
  - 오류 발생시 패리티 정보 사용하여 복구(1개)
  - 공간 효츌 좋음
- **RAID6** : RAID5를 개선
  - 사용량: 2T(N-2)
  - 최소 4개 이상의 디스크 필요
  - RAID5의 개선
  - 결함 허용(Fault-tolerance) 제공
  - 중복 패리티 정보 사용(2개)
  - 저장 속도가 다소 느리다
- **RAID1+0**
  - 신뢰성(안정성)과 성능(속도)이 동시에 뛰어난 방법
  - 최소 4개 이상의 디스크가 필요(2개의 디스크에 동시에 저장하며, 동일하게 저장된 내용을 복제하여 또다시 저장하기 때문)
  - 공간효율은 50%
  - 비용이 많이드는 단점

- RAID 구성시 공간을 효율적으로 사용하기 위해서는 반드시 동일한 크기의 디스크를 사용하는 것이 좋다.
- 또한 제조사, 모델등을 동일한 것으로 사용하는 것이 좋다.
> N: 디스크의 갯수, 각 디스크당 1TB일 경우 사용량을 나타냄

## 5. RAID 구축을 위한 준비
- Linear RAID : `/dev/sdb`(2TB), `/dev/sdc`(1TB)
- RAID0 : `/dev/sdd`(1TB), `/dev/sde`(1TB)
- RAID1 : `/dev/sdf`(1TB), `/dev/sdg`(1TB)
- RAID5 : `/dev/sdh`(1TB), `/dev/sdi`(1TB), `/dev/sdj`(1TB)
- RAID 구현을 위한 준비
  - VMware로 하드디스크 9개 생성
  - 생성된 하드디스크 파티션 생성
    - 디스크 확인
      ```
      # ls -l /dev/sd*
      ```
    - 파티션 설정
      ```
      # fdisk [각각의 디스크]
      ```
      - 파티션 생성 : `n`
      - 파티션 타입 선택  : `p`(주디스크)
      - 파티션 시작점, 끝점설정 : `Enter`(default설정)
      - 파티션 타입 변경 : `t` -> `fd`(linux riad auto)
      - 파티션 등록 : `w`

## 6. Linear LAID 구축
- 구축 흐름
  1. 선처리 작업 : 파티션 작업까지 처리
      - SCSI 0:1(`/dev/sdb`) -> `/dev/sdb1`
      - SCSI 0:2(`/dev/sdc`) -> `/dev/sdb2`
  2. `mdadm`명령어를 통해 하나의 불륨 그룹(`/dev/md9`)으로 만들기 : md9의 의미는 Linear LAID의 경우 번호가 따로 붙지 않기 때문에 9번으로 지정
      - `mdadm` 설치
        ```
        # apt-get -y install mdadm
        ```
      - 하나의 RAID 볼륨 그룹으로 만들기 명령어
        ```
        # mdadm --create /dev/md9 --level=linear --raid-device=2 /dev/sdb1 /dev/sdc1
        ```
      - RAID 상세보기 명령어
        ```
        # mdadm --detail --scan
        ```
  3. 포맷, 파일 시스템 생성
      ```
      # mkfs.ext4 /dev/md9
      ```
  4. Mount를 위한 디렉토리 생성
      ```
      # mkdir /raidLinear
      ```
  5. Mount
      ```
      # mount /dev/md9 /raidLinear
      ```
  6. 생성된 LAID 크기 확인
      ```
      # df
      ```
  7. 리부팅 시 Mount 해제방지 설정
      ```
      # gedit /etc/fstab
      ```
      ```
      /dev/md9 /raidLinear ext4 defaults 0 0
      ```
  8. 생성한 LAID 작동확인
      ```
      # mdadm --detail /dev/md9
      ```
## 7. RAID0 구축
- 구축 흐름
  1. 선처리 작업 : 파티션 작업 처리
      - SCSI 0:3(`/dev/sdd`) -> `/dev/sdd1`
      - SCSI 0:4(`/dev/sde`) -> `/dev/sde1`
  2. `mdadm`명령어로 하나의 볼륨그룹(`/dev/md0`) 생성 : md0은 raid0과 번호를 맞춰줌(혼동하지 않기 위해)
      - 하나의 볼륨 만들기
        ```
        # mdadm --crate /dev/md0 --level=0 --raid-device=2 /dev/sdd1 /dev/sde1
        ```
      - RAID 상세보기 명령어
        ```
        # mdadm --detail --scan
        ```
  3. 포맷, 파일 시스템 설정
      ```
      # mkfs.ext4 /dev/md0
      ```
  4. Mount를 위한 디렉토리 설정 및 Mount
      ```
      # mkdir /raid0
      # mount /dev/md0 /raid0
      ```
  5. 생성된 LAID 크기 확인
      ```
      # df
      ```
  6. 리부팅 시 Mount 해제방지 설정
      ```
      # gedit /etc/fstab
      ```
      ```
      /dev/md0 /raid0 ext4 defaults 0 0
      ```
  7. 생성한 LAID 작동확인
      ```
      # mdadm --detail /dev/md0
      ```

## 8. RAID1 구축
- 구축 흐름
  1. 선처리 작업 : 파티션 작업 처리
      - SCSI 0:5(`/dev/sdf`) -> `/dev/sdf1`
      - SCSI 0:6(`/dev/sdg`) -> `/dev/sdg1`
  2. 하나의 볼륨생성 : md1 -> raid1
      - 볼륨 생성하기
        ```
        # mdadm --create /dev/md1 --level=1 --raid-device=2 /dev/sdf1 /dev/sdg1
        ```
        ```
        y
        ```
      - RAID 상세보기
        ```
        # mdadm --detail --scan
        ```
  3. 포맷, 파일 시스템 설정
      ```
      # mkfs.ext4 /dev/md1
      ```
  4. Mount
      ```
      # mkdir /raid1
      # mount /dev/md1 /raid1
      ```
  5. LAID 크기확인
      ```
      # df
      ```
  6. 리부팅 시 Mount 해제 방지 설정
      ```
      # gedit /etc/fstab
      ```
      ```
      /dev/md1 /raid1 ext4 defaults 0 0
      ```
  7. LAID 작동확인
      ```
      # mdadm --detail /dev/md1
      ```

## 9. RAID5 구축
- 구축 흐름
  1. 선처리 작업 : 파티션 작업처리
      - SCSI 0:8(`/dev/sdh`) - `/dev/sdh1`
      - SCSI 0:9(`/dev/sdi`) - `/dev/sdi1`
      - SCSI 0:10(`/dev/sdj`) - `/dev/sdj1`
  2. 하나의 볼륨 생성 : md5 -> raid5
      - 볼룸 생성하기
        ```
        # mdadm --create /dev/md1 --level=5 --raid-device=3 /dev/sdh1 /dev/sdi1 /dev/sdj1
        ```
      - RAID 상세보기
        ```
        # mdadm --detail --scan
        ```
  3. 포맷, 파일 시스템 설정
      ```
      # mkfs.ext4 /dev/md5
      ```
  4. Mount
      ```
      # mkdir /raid5
      # mount /dev/md5 /raid5
      ```
  5. LAID 크기확인
      ```
      # df
      ```
  6. 리부팅 시 Mount 해제 방지 설정
      ```
      # gedit /etc/fstab
      ```
      ```
      /dev/md5 /raid5 ext4 defaults 0 0
      ```
  7. LAID 작동확인
      ```
      # mdadm --detail /dev/md5
      ```

## 10. mdadm 버그 수정을 위해 `mdadm.conf` 수정
- 내용 복사
  ```
  # mdadm --detail --scan
  ```
- `mdadm.conf`에 복사한 내용 붙여넣기
  ```
  # gedit /etc/mdadm/mdadm.conf
  ```
  - 아래의 코드만 제외하고 복사한 내용 붙여넣기
    ```
    name=server:9
    name=server:0
    name=server:1
    name=server:5
    ```
- 변경한 내용 바로 적용하기
  ```
  # update-initramfs -u
  ```

---

# Linear RAID, RAID 0, 1, 5 문제발생

## 1. RAID 하드디스크 고장, 부팅이 가능하도록 응급복구
- Linear RAID, RAID 0, 1, 5의 하드디스크 고장상황을 부팅이 가능하도록 처리해보자
- VMware에서 각각의 RAID디스크 중에 1개씩 remove
  - Linear RAID : `/dev/sdc`(1TB) 삭제
  - RAID0 : `/dev/sde`(1TB) 삭제
  - RAID1 : `/dev/sdg`(1TB) 삭제
  - RAID5 : `/dev/sdi`(1TB) 삭제

- 우분투 부팅시 제대로 부팅이 되지않는다.
- root계정으로 로그인, 후 디스크 리스트 확인 : `ls -l /dev/sd*`
- 제거된 디스크를 제외하고 나머지만 출력됨
- `df`명령어를 통해 raid된 디스크 확인해보면 raid된 디스크가 존재하지 않음
- Linear RAID, RAID0은 결함허용을 제공하지 않기 때문에 복구 불가
- RAID1, RAID5는 결함허용을 제공하기 때문에 복구가 가능
  - RAID1 복구 및 확인
  ```
  # mdadm --run /dev/md1
  # df
  # ls -l /raid1
  # mdadm -detail /dev/md1
  ```
  ```
  # mdadm --run /dev/md5
  # df
  # ls -l /raid5
  # mdadm -detail /dev/md5
  ```
- RAID1, 5는 복구되었지만 Linear RAID와 RAID0은 복구가 되지 않아 부팅이 되지 않기 때문에 RAID 상태를 변경하고, Mount 해제 방지를 위한 설정을 삭제하거나 주석처리
  ```
  # mdadm --stop /dev/md9
  # mdadm --stop /dev/md0
  ```
  ```
  # vi /etc/fstab
  ```

## 2. RAID 원상복구, 새디스크로 교체
- VMware에서 제거된 디스크 다시 추가
- 새로 추가한 하드디스크에 각각 파티션 적용하기
- RAID를 정지시키고 삭제한 디스크를 생성 또는 추가
  - Linear RAID
    ```
    # mdadm --stop /dev/md9
    # mdadm --create /dev/md9 --level=linear -- raid-device=2 /dev/sdb1 /dev/sdc1
    y
    ```
  - RAID0
    ```
    # mdadm --stop /dev/md0
    # mdadm --create /dev/md0 --level=linear -- raid-device=2 /dev/sdd11 /dev/sde1
    y
    ```
  - RAID1
    ```
    # mdadm /dev/md1 --add /dev/sdg1
    ```
  - RAID5
    ```
    # mdadm /dev/md5 --add /dev/sdi1
    ```
- `/etc/fstab` 수정, 리부팅시 Mount 해제방지를 위해 주석처리한 부분을 원복
  ```
  # vi /etc/fstab
  ```
- `mdadm`버그수정을 위해 `/etc/mdadm/mdadm.conf` 수정
  - 명령어 입력후 내용을 복사
    ```
    # mdadm --detail --scan
    ```
  - 복사한 내용을 붙여넣기
    ```
    # gedit /etc/mdadm/mdadm.conf
    ```
