# 우분투 & 윈도우 듀얼 부팅 시간 설정

우분투 윈도우 듀얼부팅 환경으로 설정하고, 윈도우로 부팅하면 시간이 맞지 않는 경우가 발생

## 해결 방법

```bash
$ timedatectl set-local-rtc 1
```

```bash
$ /etc/default/rcS
```

```
UTC=no
```

- 출처 : https://tuwlab.com/ece/28472
