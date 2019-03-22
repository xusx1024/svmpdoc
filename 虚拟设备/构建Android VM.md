# 构建Android VM

[TOC]

## 设置构建环境

对于VM映像本身，构建过程与上游AOSP大致相同，并且适用于该主题的大部分AOSP文档。我们强烈建议您在继续之前阅读以下页面：

- [此处](http://source.android.com/source/initializing.html)描述了构建环境的先决条件。
- [这里](http://source.android.com/source/building.html)有构建命令的详细信息。

所述AOSP文档引用上游主干，这是目前开发的高于4.4版本的Android。在SVMP代码方面存在一些差异。例如，OpenJDK **无法**与SVMP中的 Android 4.4,4.2或4.0一起可靠地工作，而**必须使用 **[Oracle JDK 6](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html#jdk-6u45-oth-JPR)。

如果您无权访问本机运行正确版本的计算机，则可以在[chroot](http://slightlymorethanworthless.blogspot.com/2011/08/easy-chroot-build-environment-for.html)，LXC或[docker容器](https://github.com/dkeppler/docker-aosp-build)或VM中创建工作构建环境。网上有很多很棒的指南，所以我们把这作为读者练习。如果您发现任何特别好的方法值得分享，请考虑为[维基](https://github.com/SVMP/svmp.github.io/wiki#howto-guides)做贡献。

### SVMP 1.1及更新版本

所述SVMP v1.1-1.5系列基于停用Android 4.2.2和SVMP V2.X基于停用Android 4.4。两者的推荐构建环境是Ubuntu 12.04。对于这些版本的SVMP，Ubuntu 12.04 的AOSP构建环境指令和所需的软件包列表已足够。Ubuntu 14.04也可以工作，但需要对这个[容器](https://github.com/dkeppler/docker-aosp-build/blob/master/Dockerfile)中看到的必备软件包进行一些细微的更改。

### SVMP 1.0及更早版本

SVMP 1.0和更早的代码是基于停用Android 4.0.4的。对于此版本，您将需要一台64位Ubuntu 10.04的计算机，并安装上游文档中列出的Ubuntu 10.04所需软件包。

### 建立在MacOS上

根据作者的经验，这从未成功过。最近的[上游文档](http://source.android.com/source/initializing.html#setting-up-a-mac-os-x-build-environment)已经有了很大的改进，但它仍然需要处理操作系统和Xcode的过时版本。我们建议使用Linux VM。

## 开始构建

摘要：

1. cd $ {SVMP_AOSP_ROOT} 
2. ./build-svmp.sh 
3. 根据您的偏好，根据提示回答问题。如果最后保存设置，下次将跳过菜单。
4. [编译](https://xkcd.com/303/)

从干净的树上完整构建Android可能需要花费大量时间，具体取决于您的硬件。完整构建超过一个小时并不罕见。

*注意：可以使用环境变量覆盖构建过程要求的所有问题。查看device/miter/svmp/build-svmp2.sh以查看可以设置的变量。和/或，在AOSP根目录的.svmp.conf文件中设置相同的值。*

### 关于自定义签名密钥的注意事项

如果您选择使用自定义签名密钥，这是强烈建议的，请注意用户数据卷将绑定到这些签名。如果您尝试将首先使用一个系统映像初始化的用户数据卷与一组签名连接，然后将其与另一组由另一组密钥签名的系统映像配对，则该实例将无法启动。

如果确实需要将/ data分区移动到不同签名的VM，则可以通过删除以下两个文件来清除缓存的签名：

- /data/system/packages.list
- /data/system/packages.xml

### 一体化图像构建

从SVMP 1.5.0 开始，引入了一个新的“一体化”构建目标。此选项将在单个磁盘映像中创建包含系统和数据分区的单个统一磁盘映像。这在不需要连接长期持久数据卷并从短期系统实例中删除的情况下非常有用。

要选择此选项，请选择“All-in-one”作为构建脚本的“All-in-one image”页面。

