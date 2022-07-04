# 通过Rancher Desktop在桌面上运行K8s

Rancher 发行的操作系统新选择：Rancher Desktop for Windows，它可以帮助你在Windows桌面上管理Kubernetes和容器。当然他当然会支持Linux，Mac的。

[![clip_image002](https://img2020.cnblogs.com/blog/510/202201/510-20220103101625573-1228544541.jpg)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101625157-1198363946.jpg)

准备工作

在我们探索全新的Rancher Desktop之前，我们需要准备以下内容：

1、Windows 10，版本是21H1

2、安装WSL2特性：https://docs.microsoft.com/en-us/windows/wsl/install-win10#step-1---enable-the-windows-subsystem-for-linux

3、安装Winget：Windows的包管理器，https://github.com/microsoft/winget-cli

4、安装Windows Terminal：winget install Microsoft.WindowsTerminal

安装Rancher Desktop

前期准备的内容都安装完毕之后，我们可以开始安装Rancher Desktop。

第一步，从Github repo（https://github.com/rancher-sandbox/rancher-desktop ）中获取最新版本。下载完成安装程序之后，我们可以安装Rancher Desktop：

[![clip_image004](https://img2020.cnblogs.com/blog/510/202201/510-20220103101542393-2143467192.jpg)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101541938-592822917.jpg)

安装完毕之后，我们就可以启动Rancher Desktop。选择k8s版本和 容器运行时：

[![clip_image005](https://img2020.cnblogs.com/blog/510/202201/510-20220103101543220-1020823421.png)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101542766-403392436.png)

接收后进入Rancher Desktop：

[![clip_image007](https://img2020.cnblogs.com/blog/510/202201/510-20220103101544073-2119325057.jpg)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101543654-1904157683.jpg)

启动K8s的过程中需要从github拉取镜像，因此需要开启fastgithub 这个工具。

[![clip_image009](https://img2020.cnblogs.com/blog/510/202201/510-20220103101544882-1427312410.jpg)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101544451-1178268043.jpg)

Kubernetes发展飞速，有时我们的应用程序可能还没有适配好最新版本，特别是有重大变更的情况下。Rancher深知这一点，因此为了让其操作更轻松，Rancher Desktop能一键切换版本：

[![clip_image011](https://img2020.cnblogs.com/blog/510/202201/510-20220103101545703-757280280.jpg)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101545296-1443966804.jpg)

连接到Rancher Desktop

既然已经完成了Rancher Desktop的安装，我们可以通过Kubectl命令来连接它，就像我们在任何其他Kubernetes集群或其他操作系统（如Linux）中所做的那样：

[![clip_image013](https://img2020.cnblogs.com/blog/510/202201/510-20220103101546501-507194015.jpg)](https://img2020.cnblogs.com/blog/510/202201/510-20220103101546110-880833735.jpg)

总结

在本文中，我们了解了一种在Windows上使用K8s的新方式。Windows与任何其他操作系统在Kubernetes方面不相上下。

Rancher Desktop主页已经上线，您可以访问官网主页了解更多信息：

https://rancherdesktop.io/

同时，欢迎通过GitHub下载Rancher Desktop并安装使用：

https://github.com/rancher-sandbox/rancher-desktop