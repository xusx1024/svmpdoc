# 安装和运行Android VM

[TOC]

## VirtualBox

在“要构建的映像”屏幕的build-svmp.sh脚本中，选择“ovf-vbox”选项。这将启动预设构建，其中2个虚拟磁盘通过生成的VM设备中的SATA控制器连接。

通过VirtualBox文件菜单导入生成的$ {SVMP_AOSP_ROOT} /out/target/product/svmp/svmp_vbox.ova文件。

笔记本电脑用户注意事项：默认情况下，VirtualBox会将ACPI数据从主机传递到VM。如果在使用电池运行时启动VM，VBox将错误地将“睡眠”命令传递给VM，导致它在完成启动后立即关闭。要更正，请插入电源或将VBox配置为不执行ACPI直通。

注意2：您可以将一个串行端口设备添加到VM以访问dmesg和init控制台日志。如果将交互式pty附加到该串行端口，则还可以获取本地root shell。

## VMware

在“要构建的映像”屏幕的build-svmp.sh脚本中，选择“ovf-vmware”选项。这将启动预设构建，其中2个虚拟磁盘通过生成的VM设备中的SCSI控制器连接。

通过VirtualBox文件菜单导入生成的$ {SVMP_AOSP_ROOT} /out/target/product/svmp/svmp_vmware.ova文件。

## KVM

还没有针对KVM的设备构建，因此需要稍长的方法。

1. 在构建脚本的“要构建的映像”屏幕中，选择“qcow2”。
2. 在“驱动器类型”屏幕上，为您将使用的驱动器控制器类型选择合适的型号。对于virtio半虚拟化磁盘（建议获得最佳性能），请选择“vdx”选项。
3. 根据您的偏好选择“一体化”或“单独”系统和数据磁盘

此构建将在$ {SVMP_AOSP_ROOT} / out / target / product / svmp /中创建一个或多个qcow2文件

使用qemu-img将android_system_disk.img和android_data_disk.img转换为QCOW2格式。

## OpenStack（带有KVM计算节点）

1. 根据上面的KVM指令构建QCOW2 virtio映像
2. 上传生成的图像一目了然。
3. 根据android_data_disk扫视图像创建一个cinder卷。这将成为克隆或创建每个新用户的持久性数据磁盘的主副本。
4. 从系统磁盘映像启动新VM实例。使用至少1GB的根磁盘和1GB的RAM。短暂的磁盘可以保留为零。
5. 根据需要配置安全组。见下文。
6. [可选]将VM连接到步骤3中创建的“黄金映像”卷并安装任何应用程序并配置您希望所有用户拥有的任何设置（请记住将克隆整个环境）。完成后分离VM。
7. 使用步骤3中的“黄金”快照作为基础，为每个用户帐户创建一个cinder卷。将此新卷附加到正在运行的VM。
8. 硬重新启动VM实例，以便它可以找到现在连接的数据卷。

### 安全组

Openstack安全组可用于为SVMP实例有效创建基于主机的防火墙。如果使用此功能，则必须允许访问这些端口。

SVMP服务器

- 来自svmp-server的入站TCP 8001

WebRTC

- 出站TCP和UDP 3478到您的STUN / TURN服务器
- 如果使用STUN进行NAT遍历，则在所有端口上使用STUN，入站和出站UDP
- 如果在TURN服务器上使用TURN中继，端口49125-65535上的入站和出站UDP（或者如果不是默认值，则匹配TURN服务器中配置的范围）

调试（ADB和SSH shell）

- 入站TCP 5555
- 入站TCP 22（仅在使用嵌入式SSH密钥对构建VM映像时才可访问）

## 数据磁盘

### SVMP 1.5.0+

在SVMP 1.5.0及更高版本中，构建系统根据设备/ miter / svmp /中的BoardConfig.mk和image_build / svmp _ * _ image_layout.conf文件中的设置从头开始自动创建数据磁盘。

同样在1.5.0中，缓存分区已从数据磁盘移动到系统磁盘。为了向后兼容，创建的数据卷仍将包含先前由缓存占用的插槽中的分区，因此旧数据卷仍将适用于较新的VM版本。

要自定义数据分区大小，只需将BoardConfig.mk中BOARD_USERDATAIMAGE_PARTITION_SIZE的值（以字节为单位）更改为您选择的值。如有必要，还可以调整layout.conf文件中的“num_lba”值，以使所有包含的分区具有足够的大小。

### SVMP 1.4.1及以下

构建系统创建的默认数据磁盘和输出为android_data_disk.img的是5GB磁盘，其中4GB分配给/ data，1GB分配给/ cache。

可以在以/ data分区的大小命名的'device / miter / svmp / data-disk-blanks'中找到多种大小的替代图像。全部包含1GB缓存分区。

如果预构建的图像无法满足您的需求，您可以使用以下步骤创建自定义空白数据磁盘。

1. 创建所需大小的稀疏文件

   ```
       $ dd if = /dev/zero of = android_data_disk.img bs = 1 count = 0 seek =  
   ```

2. 在映像中为/ cache和/ data创建分区。/ cache分区大约应为1GB，而/ data是可配置的，但本身至少应为1GB。

   ```
       $ fdisk android_data_disk.img 
       命令：n 
       选择：p 
       分区号：1 
       第一扇区：
       最后一个扇区...或+大小{K，M，G}：+ < SIZE / data> 
       命令：n 
       选择：p 
       分区号：2 
       第一扇区：
       最后部门： 
       命令：w  
   ```

3. 扫描图像并将新分区添加到/ dev。请注意，loopXpY为下一步命名第一个命令。

   ```
       $ sudo kpartx -l android_data_disk.img 
       $ sudo kpartx -a android_data_disk.img  
   ```

4. 这两个分区应该出现在/ dev / mapper中，类似于loop0p1和loop0p2，应该由`kpartx -l`命令打印出来。

5. 格式化两个分区。将示例中的设备名称替换为步骤3中的实名。

   ```
       $ sudo mkfs.ext4/dev/mapper/loopXp1 
       $ sudo mkfs.ext4/dev/mapper/loopXp2  
   ```

6. 卸载环回设备

   ```
       $ sudo kpartx -d android_data_disk.img  
   ```

为了将来的考虑，所有这些都可以用libguestfs很好地编写脚本。