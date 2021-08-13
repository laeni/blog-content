---
title: '修改系统和常用应用的缓存路径到非系统分区'
author: 'Laeni'
tags: Linux, Windows, 默认路径, 缓存路径
date: '2021-08-12'
updated: '2021-08-13'
---

大部分应用程序都有缓存，而且一般都会将缓存数据放在当前用户的用户目录下，默认配置情况下将会导致系统分区数据越来越多，从而经常出现空间不够的情况。数据全部放在系统分区，在重装系统时也会越到数据备份的问题。所以一般情况下需要更改大部分软件的安装目录和缓存目录到非系统分区。

**修改注意事项：**

1. 一台电脑不管是Windows还是Linux，一般情况下都只是一个人在使用，而即使有多人使用，那很多缓存是可以公用的，所以在修改缓存目录的时候最好考虑一下能否多人共享，对于不能共享的缓存建议还是以用户名隔离。
2. 有时候并不需要每个软件都修改，拿`Maven`和`Gradle`为例，只要进行全局修改之后，相关软件会自动识别。

# 系统

## Linux

对于Linux系统而言是最方便的，直接将其他分区挂载到`/home`目录即可，这样既可以解决数据不放系统分区的问题，也能让配置处于默认状态。

## Windows

对于Windows来说就不那么简单了，几乎需要一个一个的更改。

### 桌面/下载/文档/图片

以更改桌面位置为例，在资源管理器的桌面上"右键" -> "属性"  -> "位置" 处即可更改。

同样的方式可以更改下载、文档、图片的位置。

# 软件

## Gradle

设置系统环境变量`GRADLE_USER_HOME`即可全局修改。如：![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/sys/modify-default-path/1.png)

## Maven

对于Maven，一般修改本地仓库路径。修改方式最好是在当前用户目录的`.m2`目录下新建一个`settings.xml`配置文件，在配置文件中更改本地仓库目录即可。因为Maven会默认读取该配置文件，这就实现全局修改的效果。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

    <localRepository>F:\User\.m2\repository</localRepository>

    ...
</settings>
```

## JetBrains全家桶（IDEA、WebStorm、GoLand、Android Studio、...... ）

1. 设置`user.home`
   在`bin/idea[64].exe.vmoptions`配置文件中增加一行配置即可：
   `-Duser.home=F:\User`
2. 取消`bin/idea.properties`的路径配置注释。

## Android studio 缓存路径

Android studio除了是IDEA外，还需要修改Android相关缓存配置。

### .android

.android文件夹是存放android虚拟机相关的文件夹。

**修改**

1. 添加`ANDROID_SDK_HOME`环境变量，设置自己自定义目录（如`F:\User`）。
2. 修改新路径下的xxx.ini文件内的路径信息，修改完成需要重启生效。
    ![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/sys/modify-default-path/2.jpg)

### Gradle

对于Gradle，建议不要在IDE中直接修改，而是全局修改，参见[Gradle配置](#Gradle)。

### 参考

<https://blog.csdn.net/qq282330332/article/details/89244986>

## Yarn

类似于和Maven，直接在当前用户目录下的Yarn配置文件`.yarnrc`中增加缓存和全局依赖安装目录配置即可。

```
cache-folder "F:\App\NodeJs\Yarn\Cache"
global-folder "F:\App\NodeJs\Yarn\Data\global"
prefer-offline true
preferred-cache-folder "F:\App\NodeJs\Yarn\Cache"
# (可选)设置离线镜像缓存路径
yarn-offline-mirror "F:\App\NodeJs\Yarn\npm-packages-offline-cache"
# (可选)使缓存文件夹保持最新
yarn-offline-mirror-pruning true
```

## Npm

1. 设置环境变量(参考)

   ```bash
   NODEJS_HOME=/opt/apps/nodejs/node-v16.5.0
   NODE_PATH=/opt/apps/nodejs/global/node_global/node_modules
   PATH=/opt/apps/nodejs/global/node_global/bin:$NODEJS_HOME/bin:$PATH
   export NODEJS_HOME NODE_PATH PATH
   ```

2. 设置缓存和前缀

   ```
   prefix=F:\App\NodeJs\Npm\node_global
   cache=F:\App\NodeJs\Npm\node_cache
   ```


# 其他

对于Windows，可以参看官方的[释放 Windows 中的驱动器空间](https://support.microsoft.com/zh-cn/windows/%E9%87%8A%E6%94%BE-windows-%E4%B8%AD%E7%9A%84%E9%A9%B1%E5%8A%A8%E5%99%A8%E7%A9%BA%E9%97%B4-85529ccb-c365-490d-b548-831022bc9b32)教程，或许会有帮助。

