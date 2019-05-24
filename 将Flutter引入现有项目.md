
## Flutter接入现有项目

### Flutter环境配置
请在以下将flutter配置在以下目录中，并作为设置到环境变量中

	/opt/flutter 

### 创建 module

在现有的iOS目录下面，执行以下命令，该模式下，创建的只是一个module形式的代码模块，可给到现有App，通过pod的模式进行接入。

~~~
flutter create -t module flutter_module 
~~~

### 修改Pod文件
在iOS目录下面的PodFile文件中，添加引入flutter 项目的代码, flutter_application_path 值为实际加载的flutter module 相对或绝对地址

~~~
flutter_application_path = './flutter_module'
eval(File.read(File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')), binding)
~~~

实际上就可以看到，在我们创建的flutter module 中 .ios/Flutter 里面的podhelper.rb 文件，就是在当前项目中引入Flutter pod依赖的关键。

### Generated.xcconfig 配置文件
我们可以在 .ios/Flutter 同级目录下看到一个 Generated.xcconfig 的配置文件。在这个文件中，定义了与 flutter 相关的环境变量，例如 flutter 的安装目录，当前Flutter项目的编译启动入口，等等内容。

#### 注意
在观察这个文件中的内容，可以看到环境变量的赋值大多为绝对地址，而不是相对相对地址，如果希望对当前项目可以存在的协同开发的目的的话，可以将其中的绝对路径都改成相对路径。

另外，在执行 pod install 命令的时候，podhelper脚本会将执行命令，将这个文件的绝对地址通过 include 的形式引入到当前项目的各个pod xccoinfig文件中。

导致当前项目在每次执行完 pod install 命令后，可能出现多个文件被修改的问题。因此，可以对Flutter的pbhelper脚本进行过部分改造。

修改部分的代码如下，主要是在给各个xcconfig文件写入 include Generated.xcconfig 内容时，修改为使用相对路径的模式写入，而不是代码check下来时的绝对路径。同时判断如果xcconfig最后一行已经引入 Generated 的话，则不再引入。

~~~
# Ensure that ENABLE_BITCODE is set to NO, add a #include to Generated.xcconfig, and
# add a run script to the Build Phases.
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['ENABLE_BITCODE'] = 'NO'
            next if  config.base_configuration_reference == nil
            xcconfig_path = config.base_configuration_reference.real_path
            arr = IO.readlines(xcconfig_path)
            lastLine = arr[arr.length-1]
            if !lastLine.include? "Generated.xcconfig"
                File.open(xcconfig_path, 'a+') do |file|
                    generatedXConfigPath = File.realpath(File.join(framework_dir, 'Generated.xcconfig'))
                    xcconfig_filepath = Pathname.new(xcconfig_path)
                    xcconfig_dirpath = Pathname.new(xcconfig_filepath.dirname)
                    relativePath = Pathname.new(generatedXConfigPath).relative_path_from(xcconfig_dirpath)

                    #puts xcconfig_dirpath
                    #puts Pathname.new(generatedXConfigPath).relative_path_from(xcconfig_filepath)
                    #puts relativePath
                    file.puts "#include \"./#{relativePath}\""
                end
            end
        end
    end
end
~~~

如果在执行完 pod install 命令后，出现了某些xcconfig文件发生改变，且修改内容存在与机器相关的绝对路径时，请注意不要提交相关内容，且检查脚本是否引入某些会写入绝对路径的地方，尝试修改下。


## 启动项目
启动iOS工程过程与原先一样，在Xcode中点击运行即可。

打开项目中，会发现在App中Tab出现了Flutter入口，点击即是Flutter View。

#### 项目中是如何初始化Flutter的
修改原来工程中AppDelegate类，从原来的UIResponse继承修改为继承自FlutterAppDelegate，并在application:didFinishLaunchingWithOptions: 方法中，加入FlutterEngine , GeneratedPluginRegistrant的创建与初始化代码，

~~~
	 _flutterEngine = [[FlutterEngine alloc] initWithName:@"io.flutter" project:nil];
    [_flutterEngine runWithEntrypoint:nil];
    [GeneratedPluginRegistrant registerWithRegistry:_flutterEngine];
~~~

在点击FlutterTab的时候，创建FlutterViewController，并添加进入当前视图树

~~~
AppDelegate *appDelegate = (AppDelegate*)[UIApplication sharedApplication].delegate;
    
    FlutterViewController *flutterViewController = [[FlutterViewController alloc] initWithEngine:appDelegate.flutterEngine nibName:nil bundle:nil];
    _flutterViewController = flutterViewController;
    [self addChildViewController:_flutterViewController];
    _flutterViewController.view.frame = self.view.bounds;
    [self.view addSubview:_flutterViewController.view];
    
~~~

### 修改初始化进入的页面

~~~
[flutterViewController setInitialRoute:@"routeIDentity"]
~~~

## 如何Hot Reload
打开运行项目后，可以在控制台看到一条打印 

    flutter: Observatory listening on http://127.0.0.1:xxxxx/xxxxxxxx/
    
说明了当前项目的Flutter模块监听了本地xxxxx端口。

如果要Hot Reload的功能，可以使用 flutter 的 attach 命令进行连接。

~~~
flutter attach --debug-uri http://127.0.0.1:xxxxx/xxxxxxxx/
~~~

连接成功后，App即可通在终端输入r进行热重启。

关于attach可接收的更多命令，可以查看 /⁨flutter/packages⁩/flutter_tools⁩/lib⁩/src⁩/command/attach.dart 文件进行查看，还有 --debug-port app-id pid-file project-root 等不同命令组合。

在flutter attach连接上后，可以尝试修改 main.dart 的入口文件里面的内容，然后查看机器上的flutter页面是否发生改变。

## Flutter初始化项目

## 如何将Flutter引入现有项目

