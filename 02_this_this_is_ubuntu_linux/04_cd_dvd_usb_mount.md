# DVD/USE 마운트

## 마운트 된 장치 목록
  ```
  $ mount
  ```
## 마운트 된 장치 제거
  ```
  $ umount [장치디렉토리]
  ```
## CD/DVD 마운트 제거
  ```
  $ umount /dev/cdrom
  ```

## USB 장치 인식
  - NTFS포맷일 경우 기본적으로 인식하지 못함. 하지만 인식할수 있도록 설정할 수 있음
  - FAT32포맷이어야 인식가능

## 텍스트모드 장치 mount
  - 그래픽모드는 자동적으로 장치인식이 되지만 텍스트모드의 경우 자동인식이 되지않음
  - 직접 인식시켜주어야함

## ISO 이미지 생성하고 마운트하기
  - genisoimage 설치유무 확인
    ```
    $ dpkg --get-selections genisoimage
    ```
  - iso 이미지 생성
    ```
    $ genisoimage -r -J -o 파일명.iso [iso이미지에 저장할 파일의 디렉토리]
    ```
  - iso 이미지 마운트
    ```
    $ mkdir /media/iso
    $ mount -o loop 파일명.iso /media/iso
    ```
