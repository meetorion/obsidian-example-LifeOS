使用kubectl exec进入worker所在pod后，对其中的Worker进程打jstack，报错：
![[Pasted image 20240307115505.png]]
## 原因
需要指定hive用户来打jstack，因为hive用户才有对worker操作的权限。
```bash
sudo -u hive jstack 43 > dump.txt
```
https://community.boomi.com/s/article/Java-Error-Unable-to-open-socket-file-target-process-doesn-t-respond-or-HotSpot-VM-not-loaded
