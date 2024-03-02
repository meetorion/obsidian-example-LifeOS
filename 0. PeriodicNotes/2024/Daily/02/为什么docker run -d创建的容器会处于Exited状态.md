## docker logs -f
使用`docker logs -f container_id`查看详细日志。
## docker run -t -i <image_id> /bin/bash
使用如下方式可以进入镜像内部：
```bash
docker run -t -i 172.16.1.99/transwarp/asciidoc-doctor:1.9 /bin/bash
```
## Exited(0)
Exited(0)是正常退出。