# 如何本地编译Flutter Engine引擎

本文讲述在Mac上如何编译Flutter的iOS侧Engine引擎模块，并使用自编译模块进行开发调试的步骤

代码地址：[https://github.com/flutter/engine.git](https://github.com/flutter/engine.git)
### 

文章参考地址：

[https://github.com/flutter/flutter/wiki](https://github.com/flutter/flutter/wiki)
[https://github.com/flutter/flutter/wiki/Setting-up-the-Engine-development-environment](https://github.com/flutter/flutter/wiki/Setting-up-the-Engine-development-environment)


## 环境、工具配置

### depot_tools

[depot_tools](http://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)是Chromium用来开发编译的一套工具。

将[depot_tools](http://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)下载到本地，并将其配置到本地环境变量中。

具体可以参考[depot_tools](http://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)网站上的说明 

~~~
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

export PATH=$PATH:/path/to/depot_tools
~~~

配置完成后，在终端输入gclient，可以运行。

### 安装 curl, unzip, ant, cquery

请使用homebrew来安装 curl, unzip, ant, cquery  几个工具

~~~
brew install curl, unzip, ant, cquery
~~~

### fork Engine 仓库

从[Flutter](https://github.com/flutter)的[engine](https://github.com/flutter/engine)仓库，fork一份到自己的仓库中。请定期同步自己fork仓库与主仓库之间的代码，确保代码为最新的部分。

## 拉取代码

### 创建文件夹
请在本地电脑中创建一个文件夹，文件夹名字不做特殊要求，可以任意。

但是比较推荐使用 engine 。


### 配置gclient文件
该文件夹中，创建 ` .gclient ` 文件，输入内容

~~~
solutions = [
  {
    "managed": False,
    "name": "src/flutter",
    "url": "git@github.com:<your_name_here>/engine.git",
    "custom_deps": {},
    "deps_file": "DEPS",
    "safesync_url": "",
  },
]
~~~

将url的配置修改为自己fork出来的仓库地址即可。

### 同步第三方代码

通过终端进入该文件夹

~~~
cd /path/to/engine
~~~

然后执行 

~~~
gclient sync
~~~

这一步骤是将engine的代码拉取下来，并分析代码依赖于哪些第三方库，并将其依赖库也都拉取下来。

### 注意

**执行这步的过程时间会比较长。**

**非常长**

**非常长！！**

本人第一次跑这个命令的时候，执行了一个多小时，并且后续终端上没有任何输出，会给人造成一种，卡死的感觉。

实际上这时候是gclient在将下载下来的第三方库代码存在tmp文件中，并逐渐将其check out出来。

不过如果查看整个文件夹大小的话，实际上会看到大小数字是在不停增长的。

可能会需要一段比较长的时间，才能将整个工程拷贝下来，达到能运行的状态。这步请等待命令行自行结束退出。如果不小心退出了，重新执行 `gclient sync` 即可，会继续同步未同步代码。


## 编译Engine

在确保代码与依赖项都拉取完成后，即可进入编译的过程了。

使用终端进入 engine 目录下的src目录

~~~
cd engine/src
~~~

### 准备编译代码

在终端输入 

~~~
./flutter/tools/gn --ios --unoptimized
./flutter/tools/gn --ios --simulator --unoptimized
~~~

然后根据你的本地选择需要编译的平台，执行选项。具体可以看本地out目录下面生成的文件内容。

在iOS的编译模式下，生成的文件内容会出现一个`all.workspace`的文件，里面就是iOS编译需要的各个源文件。

~~~
ninja -C out/ios_debug_unopt 
ninja -C out/host_debug_unopt
~~~

在通过`ninja`编译完项目后，即可在对应的目录中看到`Flutter.framework`打包后的静态库

## 设置使用 -local-engine

### 使用Flutter工具运行项目

如果是直接使用的 `flutter run `命令运行的flutter项目，那么，在终端输入命令时，可以添加输入选项，来指定使用某个engine包。

~~~
flutter run --local-engine=XXXX
~~~

### 在现有App中指定engine引擎
可以修改App的对应的flutter模块下的`Generated.xcconfig`配置文件，里面加入一些环境变量的设置

~~~
FLUTTER_ENGINE=/path/to/engine/floder
LOCAL_ENGINE=ios_debug_sim_unopt ### and any mode else

~~~
