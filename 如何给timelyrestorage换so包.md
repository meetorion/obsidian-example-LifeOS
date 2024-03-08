给x86集群的timelyrestorage换so包的话，找一台节点（如crux-49），执行如下动作：
## 准备动作
将最新代码（要build成so包的代码）上传到服务器某个目录上
## 进入docker容器
挂载含代码的目录到docker容器里面
- 参考
```bash
docker stop timelyre-builder
docker rm timelyre-builder
docker run -dit --name timelyre-builder --net host --cpus=60 -v /mnt/disk1/myf/:/myf 172.16.1.99/timelyre/timelyre/build/x86_64/centos/builder:go1.21_rust1.74_java1.8
docker exec -it timelyre-builder bash
```
- 替换为我的
```bash
docker stop timelyre-builder
docker rm timelyre-builder
docker run -dit --name timelyre-builder --net host --cpus=60 -v /mnt/disk1/lzy/workspace/timelyre:/lzy/timelyre 172.16.1.99/timelyre/timelyre/build/x86_64/centos/builder:go1.21_rust1.74_java1.8
docker exec -it timelyre-builder bash
```
## build so包
在docker容器里面执行build命令
- 进入binding目录
```bash
cd binding
```
- 编译
```bash
bash build.sh
```
- 编译完成后在../outputlib/timelyre.so
## 修改Dockerfile
在Dockerfile中添加如下内容替换原so包：
```Dockerfile
COPY timelyre.so /usr/local/lib/libtimelyre.so
```
## 如果报错
如果报错如下：
```
error: could not compile `flux-core` (lib) due to previous error
warning: build failed, waiting for other jobs to finish...
error: could not compile `flux-core` (lib) due to previous error
Error installing library	{"name": "flux", "error": "exit status 101"}
/root/go/bin/pkg-config: exit status 1
github.com/apache/arrow/go/v12/arrow/compute
github.com/apache/arrow/go/v12/parquet/pqarrow
github.com/influxdata/influxdb/v2/common
github.com/influxdata/influxdb/v2/sqlite
+ patchelf --set-soname libtimelyre.so ../outputlib/timelyre.so
patchelf: getting info about '../outputlib/timelyre.so': No such file or directory
+ readelf -d ../outputlib/timelyre.so
readelf: ../outputlib/timelyre.so: Error: No such file
```