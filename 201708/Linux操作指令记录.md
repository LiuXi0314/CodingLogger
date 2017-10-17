# Linux 操作指令

#### 查看所有进程
`ps -a`

#### 杀死指定进程
`kill -9 [进程号]`

#### 查看adb设备
`adb devices`

#### 杀死adb server
`adb kill-server`

#### 启动 adb server
`adb start-server`

#### adb 安装apk常用的两种方式
`adb install [apkname]  直接安装`
`adb install -r [apk name]  覆盖安装`

#### Terminal 启动切换快捷按键
1. 启动Terminal
`ctrl + alt + T`
2. 启动Terminal 子Tab
`ctrl + shift + t`
3. 切换Terminal Tab
`alt + [tab no]`

#### 远程访问主机
`ssh [主机name]/[主机IP]`
`输入密码`

#### 拷贝远程主机制定内容
`scp [主机name]/[主机IP]:~/[dir] 复制文件`
`scp -r [主机name]/[主机IP]:~/[dir] 复制文件夹带内容`

#### 卸载程序
`sudo apt-get remove --purge [程序名称]`

