# 开发Flutter Channel

对于一个现有已上线的项目来说，必定存在了大量自己代码中封装的一些模块，无论是纯技术层面的，还是一些与上层业务层面相关的模块。

接入一个跨平台框架时，无论选择`RN`，`Weex`，`Flutter`中的哪个，都需要搭建一套通信的机制，从而实现原生模块与跨平台框架的数据传递与方法调用。

在RN与Weex中，这个叫做Module，而在Flutter中，这个叫`Platform Channel`。

参考文章 [《如何创建开发Flutter包与插件》](https://flutter.dev/docs/development/packages-and-plugins/developing-packages)

## 实现Channel

我们可以看下官方给出的Demo


创建一个FlutterMessageChannel，赋值name，需要调用的实例，并且确定与某一个FlutterEngine变量进行绑定。
FlutterEngine会将这个Channel实例保存在engine.platformview.messagerouter的一个std::unordered_map 哈希表里面。

在channel创建的时候，还会需要传入一个setMethodCallHandler block，作为接收到方法调用时的回调，接收一个FlutterMethodCall, 和FlutterResult, 方法调用的方法名和传入的参数，都保存在call对象的属性中，而需要回传信息的时候，将回传的信息通过result的调用，把信息回调回去。

在Channel创建的时候，还可以传入一个codec的变量，这个是用来解析flutter传递过来的data对象的。
实际上，Flutter与OC之间的交互都是通过Data对象进行的解析。

那么，需要一个解码器，在方法传递过来的时候，在接收到Data对象的时候，根据Data对象的各个字节，将其中的数据解析出来，并转换成对应的OC对象。

例如在方法调用解析的时候，就是由FlutterStandardMethodCodec进行的解析。
我们可以看到，一个标准的方法调用，在dart端调用时，只接受了两个参数，一个是方法名，一个是一个任意类型的对象。
而在iOS端接收到的FlutterMethodCall 对象，可以看到，也只是定义了两个属性，就是方法名，和一个id类型的参数。


关键是，需要先创建好engine后，在

官方给出了一个使用Plugin的例子出来。
其实本质也是，在GeneratedPluginRegistrant里面，将原来定义在Plugin里面的
