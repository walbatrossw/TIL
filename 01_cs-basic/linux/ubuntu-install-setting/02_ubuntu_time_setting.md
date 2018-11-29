# 우분투 & 윈도우 듀얼 부팅 시간 설정

우분투 윈도우 듀얼부팅 환경으로 설정하고, 윈도우로 부팅하면 시간이 맞지 않는 경우가
발생할 경우 아래와 같이 설정해준다.


## 1.해결 방법

```command-line
$ timedatectl set-local-rtc 1
```

```command-line
$ sudo gedit /etc/default/rcS
```

```command-line
UTC=no
```

- 출처 : https://tuwlab.com/ece/28472
