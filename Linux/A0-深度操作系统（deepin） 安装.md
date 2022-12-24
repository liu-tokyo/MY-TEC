# 深度操作系统（deepin） 安装

> 个人安装系统的时候，遇到了如下问题，很辛苦找到了解决办法，如果您在安装的时候遇到类似问题，也可以作为参照。  
> 比较起来，还是不如Ubuntu等Linux，人家毕竟是用户面太广，用起来自然问题很少。如果用 Linux 作为开发，还是选择国际版本的类似于 Ubuntu 更为合理。

## 升级时出现

- 升级指令

  ```
  sudo apt update
  ```

- 问题：deepin20执行`sudo apt update` 后出现如下错误提示：

  ```vbnet
  E: 无法获得锁 /var/lib/apt/lists/lock - open (11: 资源暂时不可用) 
  E: 无法对目录 /var/lib/apt/lists/ 加锁
  ```

- 无法升级。经查解决办法如下：

  ```shell
  sudo rm /var/cache/apt/archives/lock
  sudo rm /var/lib/dpkg/lock
  ```
  
  真实原因是，系统自身启动之后再进行更新，所以你无法用指令进行更新。到设置里面把 **更新** 设置为不要自动更新，省了和我们抢资源，自己有对更新的安排不好，导致出现错误。



## 更换软件源

- 修改文件

  ```
  sudo nano /etc/apt/sources.list
  ```

  在日可以用的软件源：

  ```
  deb http://ftp.tsukuba.wide.ad.jp/Linux/deepin/ apricot main contrib non-free
  deb ftp://ftp.riken.jp/Linux/deepin/ apricot main contrib non-free
  ```

  举例：
  阿里源：

  deb [by-hash=force] https://mirrors.aliyun.com/deepin/ apricot main contrib non-free

  网易源：

  deb [by-hash=force] https://mirrors.163.com/deepin/ apricot main contrib non-free

  华为源：

  deb [by-hash=force] https://mirrors.huaweicloud.com/deepin/ apricot main contrib non-free

  清华源：

  deb [by-hash=force] https://mirrors.tuna.tsinghua.edu.cn/deepin/ apricot main contrib non-free

1. 官方软件源商店部分：

   文件路径：`/etc/apt/sources.list.d/appstore.list`
   主要包括深度自己开发的deepinwine应用，部分商业合作应用，部分重分发授权应用

   ```
   deb https://com-store-packages.uniontech.com/appstore deepin appstore
   ```

2. 官方软件源打印机驱动部分：

   文件路径：`/etc/apt/sources.list.d/printer.list`
   主要包括深度适配的打印机驱动等

   ```
   deb https://community-packages.deepin.com/printer eagle non-free
   ```

3. 官方软件源仓库部分：

   文件路径：`/etc/apt/sources.list`
   主要是不受商业协议影响的开源部分及上游更新的仓库



## 软件源错误

- 错误内容

  ```
  命中:1 http://mirrors.aliyun.com/deepin apricot InRelease                                                                                                                 
  忽略:2 https://pro-driver-packages.uniontech.com eagle InRelease                                                                                                          
  错误:3 https://pro-driver-packages.uniontech.com eagle Release                                                                                       
    404  Not Found [IP: 61.54.25.98 443]
  命中:5 https://community-packages.deepin.com/driver driver InRelease                                     
  命中:4 https://home-store-img.uniontech.com/221221173616551/appstore deepin InRelease
  命中:6 https://community-packages.deepin.com/printer eagle InRelease
  正在读取软件包列表... 完成
  E: 仓库 “https://pro-driver-packages.uniontech.com eagle Release” 没有 Release 文件。
  N: 无法安全地用该源进行更新，所以默认禁用该源。
  N: 参见 apt-secure(8) 手册以了解仓库创建和用户配置方面的细节。
  ```

  原因：有些软件源（devicemanager）的URL不完善。

- 修改方法

  ```
  cd /etc/apt/sources.list.d
  sudo mv devicemanager.list ~/.
  ```

  把 `devicemanager.list` 文件，转移到其它位置，相当于删除吧！



## 解锁密钥环

密钥环是linux系统用于安全保存程序私密数据的模块，可以用于加密保存密码、证书、密钥等安全数据。chrome的密钥环用于保存本地访问站点密码或缓存从google服务器同步下来的访问站点的密码。  
Deepin系统的chrome会默认会把密码放在登录密钥环里，之所以会提示解锁登录密钥环是因为你的登陆密钥环被锁定了，只要把你的登陆密钥环解锁就可以了。  
之所以出现这个问题，是我个人把开机设置为自动登录，没有了输入口令这一环节，导致每次开机后最先出现的都是 `解锁密钥环` 的画面。

- 安装seahorse

  ```
  sudo apt-get install seahorse
  ```

- 运行seahorse

  ```
  seahorse
  ```

- 解锁密码或修改密码环密码
  密码->登录(Login)->右键【**删除**】。（因为我希望系统自动登录，而不需要输入口令）

- 另外一种情况
  如果是chrome的安全数据没有存放于登录密钥环，那么有可能是创建了一个新叫“默认密钥环”的密钥环来存储。如想不每次打开电脑都输入的话就应该把它的密码设为空或者在登录密钥环上创建一个项目指向“默认密钥环”。

  密码->默认密钥环->修改密码



## 修改键盘布局

本人的电脑是日文的106键盘，因为键盘不对照的问题，输入的时候非常难受。所以需要把键盘修改为日文键盘方式，国内普通用户没有这个要求。

- 修改键盘配置文件：

  ```bash
  sudo nano /etc/default/keyboard
  ```

  把文件中的内容中的 `cn` 修改为 `jp` ；

  ```
  XKBMODEL="pc105"
  XKBLAYOUT="jp"
  XKBVARIANT=""
  XKBOPTIONS=""
  
  BACKSPACE="guess"
  ```

  其它的无需修改，虽然有人建议用如下操作修改键盘布局，但是虽然是看起来修改了，依然是没有正确修改，都需要后续的步骤进行补充。

  ```bash
  sudo dpkg-reconfigure keyboard-configuration
  ```

  这个命令可以修改键盘布局，当然也依然需要后续的步骤补充，其实他也是修改 `/etc/default/keyboard` 这个文件，只是给了一个伪界面而已。

- 控制中心 → 键盘和语言；

- 在 **输入法** 页面，点击**（+）**追加输入法；  
  在 `添加输入法` 页面里面，输入 `j` 字母，就能直接跳转到日语的选择部分。  
  选择 `键盘 - 日语`，点击 `添加` 按钮。

- 点击  `输入法管理` 右侧的  `编辑`按钮， 删除列表里面的`键盘 - 英语（美国）` 项目后，点击 `完成` 按钮；

- 重新启动电脑，可以确认键盘的布局变为了日语的 106 键盘。

如上操作后，键盘输入变为日语 106 键盘，虽然偶尔还不是那么舒服，但是已经很不错了。