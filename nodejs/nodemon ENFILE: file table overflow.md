# nodemon 因为 Mac 上打开文件数的限制而运行失败的问题

> [nodemon] Internal watch failed: ENFILE: file table overflow, watch 'xxxxx'

**解决方法**：

```shell
echo kern.maxfiles=65536 | sudo tee -a /etc/sysctl.conf
echo kern.maxfilesperproc=65536 | sudo tee -a /etc/sysctl.conf
sudo sysctl -w kern.maxfiles=65536
sudo sysctl -w kern.maxfilesperproc=65536
ulimit -n 65536
```

