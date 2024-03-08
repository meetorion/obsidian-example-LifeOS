目前微信出来原生linux版本，但是deb格式的，如何在archlinux中安装deb应用呢？

## debtap打包
- 安装debtap
```bash
yay -S debtap
```
- 更新debtap数据库
```bash
sudo debtap -u
```
- 使用debtap转换deb包
```bash
debtap xxx.deb
```
- 安装
```bash
sudo pacman -U xxx.pkg
```
