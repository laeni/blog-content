# 通过 GRUB 命令行启动

1. 进入 GRUB 命令行

2. 列出可用的启动选项

   ``````sh
   ls
   ``````

3. 设置 root 设备

   ```sh
   set root=(hd0,msdos1)
   ```

   > 1. 上述示例中的`hd0,msdos1`为`ls`命令输出的结果之一。
   > 2. 可以通过`echo $root`或`gettext $root`查看当前 root 设备。

4. 选择需要使用的 EFI 文件并加载

   ```sh
   chainloader xxx.efi
   ```

   > 设置好 root 设备后，可以通过`ls /xxx`查看文件。

5. 启动系统

   ```sh
   boot
   ```

   