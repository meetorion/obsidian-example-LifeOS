## 创建adoc容器
```bash
docker run -d --name adoc-test-lzy 172.16.1.99/transwarp/asciidoc-doctor:1.9
```
这将在后台创建并运行一个基于 asciidoc-doctor:1.9镜像的容器，并命名为 `adoc-test-lzy` 。 #docker/why [[为什么docker run -d创建的容器会处于Exited状态]]

如果需要在容器内执行命令，可以使用 `docker exec` 命令。例如，要进入名为 `adoc-test-lzy` 的容器：
```bash
docker exec -it adoc-test-lzy /bin/bash
```
## 相关链接
- [[如何查看后台正在运行的docker容器]] #docker 