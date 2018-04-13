# inotify limit error 해결 방법

```bash
$ cat /proc/sys/fs/inotify/max_user_watches
```

```bash
$ echo fs.inotify.max_user_watches=16384 | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p
```

## 출처 : https://askubuntu.com/questions/770374/user-limit-of-inotify-watches-reached-on-ubuntu-16-04?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa
