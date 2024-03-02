## 背景
leopard-doc的cicd中使用如下命令生成pdf文件：
```bash
asciidoctor-pdf -r asciidoctor-pdf-cjk-kai_gen_gothic -a pdf-style=transwarp TimeLyre_Python_API.adoc
```
其基于的镜像为：
```
172.16.1.99/transwarp/asciidoc-doctor:1.9
```
## 查看asciidoctor-pdf版本
```bash
asciidoctor-pdf -v
```
结果如下：
```
Asciidoctor PDF 1.5.0.alpha.16 using Asciidoctor 1.5.8 [https://asciidoctor.org]
Runtime Environment (ruby 2.1.9p495 (2017-12-15 revision 54437) [x86_64-linux-gnu]) (lc:US-ASCII fs:US-ASCII in:US-ASCII ex:US-ASCII)
```
