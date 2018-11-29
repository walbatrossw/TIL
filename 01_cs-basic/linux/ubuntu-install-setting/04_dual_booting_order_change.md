# 우분투 & 윈도우 듀얼부팅 순서 변경하기

## grub 설정파일 편집

```command-line
$ sudo gedit /etc/default/grub
```

## 수정
`GRUB_DEFAULT=0` 값을 변경, 보통 4번의 윈도우가 위치하기 때문에 4번으로 변경

## 부팅 진입시간 변경
`GRUB_TIMEOUT=10` 변경

## 변경 사항 갱신

```command-line
$ sudo update-grub
```
