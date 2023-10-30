# issueWhenCode_flutter
issues when coding in flutter

## 开发问题汇总

### 回到上一页并刷新
   A->B
   
   B: context.pop(true)
   
   A: 
      final bool? result = await context.push<bool>(A);
      if(result!=null&&result){
         //callback
      }

### 回到指定页
   Navigator.popUntil(context, ModalRoute.withName('/index'));

### Wrap宽度是由子元素决定。
   可将Wrap放置在Container中，实现居左定位。

### SizedBox可以解决icon外边缘不统一问题，同理，可解决checkbox外边缘问题。

### 从页面A push 到页面B，页面B中调用context.pop  回到页面A，页面A如何知道重新展示到前台了
   1.Navigator
```dart
在Flutter中，当您从页面A使用 `Navigator.push` 跳转到页面B，并在页面B中使用 `Navigator.pop` 返回页面A时，Flutter会自动调用页面A相关的 `State` 对象的生命周期方法。

要在页面A重新展示到前台时执行特定操作，您可以使用 `RouteObserver` 来监听路由的变化。以下是一个示例，展示如何使用 `RouteObserver` 来监听页面A的生命周期：

1. 创建一个 `RouteObserver` 对象：
final RouteObserver<PageRoute> routeObserver = RouteObserver<PageRoute>();
2. 在您的应用程序中注册 `RouteObserver` ：
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      navigatorObservers: [routeObserver],
      home: PageA(),
    );
  }
}
3. 在页面A的状态类中，使用 `RouteAware` 混入类并实现相关的回调方法：
class _PageAState extends State<PageA> with RouteAware {
  String _displayText = '';

  @override
  void initState() {
    super.initState();
    routeObserver.subscribe(this, ModalRoute.of(context));
  }

  @override
  void dispose() {
    routeObserver.unsubscribe(this);
    super.dispose();
  }

  @override
  void didPush() {
    print('Page A is pushed to the top');
    setState(() {
      _displayText = 'Welcome back to Page A!';
    });
  }

  @override
  void didPopNext() {
    print('Page A is now visible again');
    setState(() {
      _displayText = 'Welcome back to Page A!';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Page A'),
      ),
      body: Center(
        child: Text(_displayText),
      ),
    );
  }
}
在这个示例中，我们使用 `RouteObserver` 来监听路由的变化。在 `didPush` 和 `didPopNext` 方法中，我们更新了页面A的状态并重新构建UI。当页面A被推到顶层时， `didPush` 方法被调用；当页面A重新变为可见时， `didPopNext` 方法被调用。

通过使用 `RouteObserver` 和 `RouteAware` 混入类，页面A可以在重新展示到前台时执行特定操作。

希望这个示例对您有所帮助！如果您有任何进一步的问题，请随时提问。
```

   2.go_router
```dart
   final bool? result = await context.push<bool>('/page2');
   if(result ?? false)...
```

### Positioned 元素底部和屏幕底部有空隙
   设置bottom:0，注意 要保持高度或者top为null，即未设置。

### 3 Easy Steps to Close Keyboard in Flutter with Code
   1. Step 1: Wrap your widget (probably the scaffold widget) with the GestureDetector.
   2. Add the onTap callback. Adding onTap callback allows us to detect the single touch events on the screen.
   3. Step 3: Finally, add FocusManager.instance.primaryFocus?.unfocus() method to dismiss the keyboard.

### 如何消除checkbox外部padding
   将其包裹在SizedBox中
```dart
   SizedBox(
      height: 24.0,
      width: 24.0,
      child: Checkbox(...),
   )
```

### showBottomSheet中更新 调用该方法的widget 的state，不会立即生效，要关闭bottomSheet后才生效
   解决方式：
```dart
import 'package:flutter/material.dart';
 void main() {
  runApp(MyApp());
}
 class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MyWidget(),
    );
  }
}
 class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}
 class _MyWidgetState extends State<MyWidget> {
  int _value = 0;
   void _updateValue(int newValue) {
    setState(() {
      _value = newValue; // Update the value here
    });
  }
   void _showBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      builder: (BuildContext context) {
        return MyBottomSheet(updateValue: _updateValue);
      },
    );
  }
   @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Current Value: $_value'),
            ElevatedButton(
              child: Text('Open Bottom Sheet'),
              onPressed: () {
                _showBottomSheet(context);
              },
            ),
          ],
        ),
      ),
    );
  }
}
 class MyBottomSheet extends StatelessWidget {
  final Function(int) updateValue;
   MyBottomSheet({required this.updateValue});
   @override
  Widget build(BuildContext context) {
    return Container(
      height: 200,
      color: Colors.white,
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('Update Value:'),
            ElevatedButton(
              child: Text('Increment Value'),
              onPressed: () {
                updateValue(1); // Call the callback function to update the value
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

### error: Scaffold.of() called with a context that does not contain a Scaffold.
   Scaffold在widget根部存在，但仍报这个错误。
   解决方法：you need to wrap the content of the  showBottomSheet  with a  Builder  widget.
```dart
   final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();
   _showBottomSheet(context);
   void _showBottomSheet(BuildContext context) {
      _scaffoldKey.currentState!.showBottomSheet<void>(
                 (BuildContext context) {
            return Container(
               height: 200,
               color: Colors.white,
               child: Center(
                  child: Text('Bottom Sheet Content'),
               ),
            );
         },
      );
   }
```

### 安卓闪退问题
   需关闭混淆
```dart
   ///关闭混淆
  minifyEnabled false //删除无用代码
  shrinkResources false //删除无用资源
```
完整代码
```dart
   buildTypes {
      release {
         // TODO: Add your own signing config for the release build.
         // Signing with the debug keys for now, so `flutter run --release` works.
         signingConfig signingConfigs.release
         //关闭混淆
         minifyEnabled false //删除无用代码
         shrinkResources false //删除无用资源
      }
   }
```

### Bruno form元素间距
```dart
BrnRadioInputFormItem(
  title: "经营方式",
  options: [
    "批发",
    "零售",
    "批发兼零售",
  ],
  value: "批发",
  onChanged: (oldValue, newValue) {
    BrnToast.show("点击触发回调${oldValue}_${newValue}_onChanged", context);
  },
  themeData: BrnFormItemConfig(
      formPadding:EdgeInsets.zero,
      titlePaddingSm:EdgeInsets.zero,
      titlePaddingLg:EdgeInsets.zero,
      titleTextStyle:BrnTextStyle(
        fontSize: 30.rpx,
        color:const Color(0xFF999999),
      ),
      contentTextStyle:BrnTextStyle(
        fontSize: 30.rpx,
        color:const Color(0xFF333333),
      ),
  ),
),
```

### 如果要滚动的时候，appBar跟随滚动
方案：自定义appBar，把appBar放到body中。

### "多分辨率适配"或"多像素密度适配"
   高分辨率加载 @2x @3x 图片，flutter中如何实现？
   假设有个mine文件夹 - images/mine
   在 - images/mine 下 新建 2.0x或者3.0x文件夹，把高分辨率图片放入其中。 注意，不要加@2x，@3x后缀，保持与mine下图片同名。

### 获取bottomNavigationBar高度
kBottomNavigationBarHeight

### Column中如何等分行？
具体采用哪种布局，要看实际场景。
方法一：
```dart
///方案一有以下特点
///1.只有在子项超出屏幕后才会滚动
///2.宽度固定的情况下，高度就固定，因为宽高比固定
///3.更适合占满全屏的UI
  Flexible(
     child: GridView.count(
       shrinkWrap: true,
       crossAxisCount: 4,
       childAspectRatio: 1,
       children: List.generate(8, (index) {
         return Container(
           color: Colors.blue,
           child: Center(
             child: Text(
               'Item ${index + 1}',
               style: TextStyle(
                 color: Colors.white,
                 fontSize: 16,
               ),
             ),
           ),
         );
       }),
     ),
  ),
```
方法二：
```dart
  Wrap(
     children: List.generate(5, (index) {
       return FractionallySizedBox(
         widthFactor: 0.25,
         child: Container(
           color: Colors.blue,
           child: Center(
             child: Text(
               'Item ${index + 1}',
               style: TextStyle(
                 color: Colors.white,
                 fontSize: 16,
               ),
             ),
           ),
         ),
       );
     }),
  ),
```

### input 输入框图标左右有空隙
    用自定义图片，或者 无空隙的第三方图标库font_awesome_flutter。

### input 左侧图标 和 输入框空隙过大
    // minWidth设置输入框距离左侧的距离，要大于图标的宽度，这样距离图标有一定空隙。
    prefixIconConstraints: BoxConstraints(minWidth:56.rpx,maxWidth: 56.rpx,minHeight: 48.rpx,maxHeight: 48.rpx),

### 底部导航 IndexedStack 全部初始化问题
    https://www.cnblogs.com/qqcc1388/p/11678085.html
    正确来说应该是切换到具体页面的时候才进行初始化，而不是一开始就加载所有的页面的数据，避免资源浪费。
    采用 PageView + AutomaticKeepAliveClientMixin 替换IndexedStack

### GestureDetector bug: 内容空白处点击不会触发onTap事件
    解决方式：child使用Container，并加上背景色

### 报错：Scaffold.of() called with a context that does not contain a Scaffold.
    调用 Scaffold.of 的组件用 Builder包一下

### Don't use 'BuildContext's across async gaps.
    // 解决方式：Future.delayed
    Future.delayed(Duration.zero,(){
        context.pushReplacement('/index');
    });

### 报错 Invalid argument(s): No host specified in URI file:///
    在主体档案跳转到店铺详情时，报这个错误。
    原因：主体档案里有使用Image.network，如果参数是空字符串，跳转时会报错。
    这里证明一个问题：跳转时会重绘当前页面，这应该是个bug。
    解决方式：给个http地址
    widget.store["doorstepImg"]??"http://www.backup.com",

### 回到第一个问题：安卓5.0 屏幕适配方案后代页面失效
    该问题是，跳转其他页面，自身重绘引起的。
    上面的报错给了启发，判断是否已初始化，若已初始化，不再执行。
    解决方式：添加hasInitialized标识，初始化后置为true，后面再执行到初始化，判断hasInitialized，若为真，跳过执行。

### 扫码查看跳转到下一页后，对准二维码，仍能扫描
    解决方式：跳转用pushReplacement
#### 引申：push把页面放到缓存中，但页面宽度变成0，并且原来页面rebuild，所以出现了适配失效的问题。
    安排时间研究下页面堆栈了。

### 日期时间选择器
    引入bruno，采用bruno日期时间选择器，但其不包含适配功能，若要在pad使用，需测试pad显示是否正常。
    pad已测试，显示正常。

### 若Row 包含 TextField or TextFormField，需将子元素包含在有宽度限制的元素中。
    If your Row contains a TextField or TextFormField or any other widget which tries to grow to infinity, you need to provide an intrinsic width to it.

### 报错 image_picker: compressing is not supported for type .png. Returning the image with original quality
    改用 wechat_assets_picker
#### wechat_assets_picker改变相册名
    pathNameBuilder

### Stack onTap不生效
#### Stack->Positioned->GestureDetector not work
##### -> 指代子元素
    see https://api.flutter.dev/flutter/widgets/Stack-class.html
    The stack paints its children in order with the first child being at the bottom. If you want to change the order in which the children paint, you can rebuild the stack with the children in the new order. If you reorder the children in this way, consider giving the children non-null keys. These keys will cause the framework to move the underlying objects for the children to their new locations rather than recreate them at their new location.
    you can put the taped widget at the last of the Stack children!
    解决方式：要把点击元素放在最后。

#### Stack->Positioned->GestureDetector 定位组件超出Stack部分，点击无反应
    方式一：改变布局，放到Stack界面内。
    方式二：改变布局，超出部分也改到Stack内，超出部分看着超出，实际还是在Stack内。


### textFormField 指定行数后，键盘没有完成按钮
    解决方式：
    ```dart
        keyboardType:TextInputType.text
    ```

### dio 动态文件上传 (参考证据采集页面图片上传)
    ```dart
        var formData = FormData();
        for (var file in filepath) {
            formData.files.addAll(
                [
                  MapEntry("assignment", await MultipartFile.fromFile(file)),
                ]
            );
        }
    ```
### 设置状态栏颜色
#### 方法一：只生效了一次
    ```dart
        //设置状态栏颜色
        SystemChrome.setSystemUIOverlayStyle(const SystemUiOverlayStyle(
        statusBarColor: Colors.transparent, //状态栏背景颜色
        statusBarIconBrightness: Brightness.dark // dark:一般显示黑色   light：一般显示白色
        ));
    ```
#### 方法二：有效
[change-status-bar-color-in-flutter](https://www.flutterbeads.com/change-status-bar-color-in-flutter/)

    ```dart
        theme: ThemeData(
            appBarTheme: const AppBarTheme(
                systemOverlayStyle: SystemUiOverlayStyle( //<-- SEE HERE
                // Status bar color
                statusBarColor: Colors.transparent,
                statusBarIconBrightness: Brightness.dark,
                statusBarBrightness: Brightness.light,
            ),
          ),
        ),
    ```

### 数组获取index
    https://fireship.io/snippets/dart-how-to-get-the-index-on-array-loop-map/
#### 方式一
    ```dart
        myList.asMap().entries.map((entry) {
            int idx = entry.key;
            String val = entry.value;
            return something;
        }
    ```

#### 方式二
    ```dart
        final List fixedList = Iterable<int>.generate(myList.length).toList();
        fixedList.map((idx) {
          String val = myList[idx];
          return something;
        }
    ```

#### 方式三
    ```dart
        final List uniqueList = Set.from(myList).toList();
            uniqueList.map((val) {
            String idx = uniqueList.indexOf(val);
            return something;
        }
    ```

### 打开网络文件
    用webview打开
#### 补充：远程pdf可在ios打开，但android不行，解决方法在下面的问题记录中。

### 多个请求 EasyLoading.dismiss(); 会互相影响
    方式一：await，多个请求顺序执行。
    方式二：loading 抽出来，放到顶层,不合适，算了。

### Stack及其alignment属性结合使用，可对Positioned子组件绝对居中。

### 疑问：元素宽度包含边框吗？
    包含

### bug：Consumer 跳回上一页时，还会执行

### scaffold.of 子组件的事件调用setState不生效，场景：列表切换页码
    解决方式：声明到函数中，调用函数。

### 项目在windows android studio 点击组件无法跳转到声明的地方
#### Cannot find declaration to go to
    解决方式：
    1.close your project
    2.delete .idea folder and *.iml files in root path
    3.open your project

### 百度地图连续定位不生效
    报错：
    Assertion failure in -[CLLocationManager setAllowsBackgroundLocationUpdates:], CLLocationManager.m:1
    解决方式:
    info.plist中添加
    ```
    <key>UIBackgroundModes</key>
        <array>
           <string>location</string>
        </array>
    ```

### Container包Container
    如果父Container不加alignment属性，子Container会忽略自身宽高，撑满父Container。
    ```
    Container(
                              alignment: Alignment.center,
                              width: 48.rpx,
                              height: 48.rpx,
                              decoration: BoxDecoration(
                                color: Colors.transparent,
                                border: Border.all(
                                  width: 2.rpx,
                                  // color: Colors.green,
                                ),
                              ),
                              child: Container(
                                width: 28.rpx,
                                height: 28.rpx,
                                decoration: BoxDecoration(
                                  color: Colors.green,
                                ),
                              ),
                            ),
    ```

### android打开远程pdf失败，ios无该问题
    android打开远程pdf空白，但ios可以打开
    解决方法
    方法一，先下载到本地，再打开
[stackoverflow how-to-load-and-present-a-pdf-file-from-the-web](https://stackoverflow.com/questions/62476108/how-to-load-and-present-a-pdf-file-from-the-web-in-flutter)

    方法二，flutter_cached_pdfview


### Execution failed for task ':flutter_pdfview:verifyReleaseResources'.
    安装flutter_pdfview 1.2.9
    这个错误是引入 flutter_cached_pdfview后抛出，依赖上面的插件，github有该问题，通过安装上面插件解决。
[Execution failed for task ':flutter_pdfview:verifyReleaseResources'](https://github.com/binSaed/flutter_cached_pdfview/issues/95)

### dependency_overrides
    解决依赖版本冲突问题

### 后台切前台黑屏，切换其他app再切回来正常。
    解决方案一：https://www.jianshu.com/p/c21ce0f730f3
    解决方案二：https://www.jianshu.com/p/f25f2d0c6ae6
    
    尝试解决：
    MainActivity中添加
    override fun onRestart() {
        super.onRestart() // Always call the superclass method first
        flutterEngine!!.lifecycleChannel.appIsResumed()
        // Activity being restarted from stopped state
    }

### Couldn't create directory for SharedPreferences file /data/user/0/com.tzjzib.ebike/shared_prefs/authStatus_com.tzjzib.ebike
      "Android : Couldn't create directory for SharedPreferences file" will come when you are already created default shared preference in project. 
      So, one shared preference is exist and again, you are adding with same name default preference. 
      Solution - Create only one default shared preference in app. 
      If do you want to create more shared preference then create shared preference with some name.
      Note - Only one default shared preference in one app.
      解决方式：创建一个全局shared preference
   
      In Flutter, you can make a SharedPreferences instance global by using a Singleton pattern. This ensures that there is only one instance of the SharedPreferences class throughout your application and it can be accessed from any widget or class.
      Here's an example of how you can create a SharedPreferences Singleton in Flutter:
      1. Create a new file named  `SharedPreferencesService.dart` .
      2. In  `SharedPreferencesService.dart` , import the  `SharedPreferences`  package:
   ```dart
      import 'package:shared_preferences/shared_preferences.dart';
   ```
      3. Create a class named  `SharedPreferencesService` .
   ```dart
    class SharedPreferencesService {
      // Private constructor
      SharedPreferencesService._internal();
      // Singleton instance
      static final SharedPreferencesService _instance =
      SharedPreferencesService._internal();
      // Getter for the instance
      static SharedPreferencesService get instance => _instance;
      // SharedPreferences instance variable
      late SharedPreferences _preferences;
      // Initializes the SharedPreferences instance variable
      Future<void> init() async {
         _preferences = await SharedPreferences.getInstance();
      }
   
      // Getters and Setters for SharedPreferences values
      String? getString(String key) => _preferences.getString(key);
      Future<bool> setString(String key, String value) =>
              _preferences.setString(key, value);
      int? getInt(String key) => _preferences.getInt(key);
      Future<bool> setInt(String key, int value) =>
              _preferences.setInt(key, value);
      bool? getBool(String key) => _preferences.getBool(key);
      Future<bool> setBool(String key, bool value) =>
              _preferences.setBool(key, value);
      Future remove(String key) => _preferences.remove(key);
   }
   ```
      In the  `SharedPreferencesService`  class, we first create a private constructor and a static instance variable. We then create a getter for the instance variable and initialize it using the  `SharedPreferences.getInstance()`  function in the  `init()`  method.
      We also create getters and setters for the SharedPreferences values, so that we can easily access them from any widget or class.
      4. In the  `main()`  method of your Flutter app, initialize the SharedPreferences Singleton by calling the  `init()`  method:
   ```dart
      void main() async {
         WidgetsFlutterBinding.ensureInitialized();
         await SharedPreferencesService.instance.init();
         runApp(MyApp());
      }
   ```
      By calling the  `init()`  method in the  `main()`  method, we ensure that the SharedPreferences Singleton is initialized before the app starts.
      Now you can access the SharedPreferences Singleton from anywhere in your Flutter app using the following code:
      
      SharedPreferencesService.instance.getString('key');
      SharedPreferencesService.instance.setInt('key', 123);
      SharedPreferencesService.instance.getBool('key');

      By making the SharedPreferences instance global, you can easily store and retrieve data from SharedPreferences from any widget or class in your Flutter app.

### MediaQuery引起界面Rebuild
[Flutter调优--深入探究MediaQuery引起界面Rebuild的原因及解决办法 | 京东云技术团队](https://juejin.cn/post/7238431303094485047)

      上面的方案对3.10.x不再适用。针对3.10.x解决方案如下：
      在MaterialAppbuilder方法中获取状态栏高度，存为常量，在后续页面中引用这个常量，不会引起rebuild。
      针对键盘弹出引起重绘的问题，将输入框直接包裹在GestureDetector中，可以阻止重绘问题。代码如下：
   ```dart
      GestureDetector(behavior:HitTestBehavior.opaque,child:TextField(decoration:InputDecoration(hintText:'Entertext'),),);
   ```

### W/OnBackInvokedCallback(27573): OnBackInvokedCallback is not enabled for the application.
      W/OnBackInvokedCallback(27573): Set 'android:enableOnBackInvokedCallback="true"' in the application manifest.
    输入框键盘弹出时，debug打印该警告。
    文档解释：如需选择启用预测性返回手势，请在 AndroidManifest.xml 的 <application> 标记中将 android:enableOnBackInvokedCallback 标志设置为 true。
    可以忽略该问题。

### 集成百度地图
#### andriod
1. 在android/build.gradle添加如下代码：
   classpath "com.baidu.lbsyun:base:7.5.5@aar"
   完整代码如下：
```
buildscript {
    ext.kotlin_version = '1.7.10'
    repositories {
        google()
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:7.3.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "com.baidu.lbsyun:base:7.5.5@aar"
    }
}
```
2. 在android/app/build.gradle中添加如下代码：
   implementation 'com.baidu.lbsyun:BaiduMapSDK_Map:7.5.5'
   implementation 'com.baidu.lbsyun:BaiduMapSDK_Search:7.5.5'
   implementation 'com.baidu.lbsyun:BaiduMapSDK_Util:7.5.5'
   implementation 'com.baidu.lbsyun:BaiduMapSDK_Location_All:9.4.0'
   完整代码如下：
```
    dependencies {
        implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
        implementation 'com.baidu.lbsyun:BaiduMapSDK_Map:7.5.5'
        implementation 'com.baidu.lbsyun:BaiduMapSDK_Search:7.5.5'
        implementation 'com.baidu.lbsyun:BaiduMapSDK_Util:7.5.5'
        implementation 'com.baidu.lbsyun:BaiduMapSDK_Location_All:9.4.0'
    }
```
3. 在android/app/src/main/AndroidManifest.xml中添加
```
<!-- 用于访问wifi网络信息，wifi信息会用于进行网络定位 -->
<uses-feature
    android:name="android.hardware.telephony"
    android:required="false" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<!-- 获取网络状态，根据网络状态切换进行数据请求网络转换 -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!-- 写外置存储。如果开发者使用了离线地图，并且数据写在外置存储区域，则需要申请该权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<!-- 读取外置存储。如果开发者使用了so动态加载功能并且把so文件放在了外置存储区域，则需要申请该权限，否则不需要 -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<!-- 访问网络，进行地图相关业务数据请求，包括地图数据，路线规划，POI检索等 -->
<uses-permission android:name="android.permission.INTERNET" />
```
以及
```
    <meta-data
        android:name="com.baidu.lbsapi.API_KEY"
        android:value="百度开发者平台申请的安卓应用key" />
```
4. 在android/app/src/main/kotlin/com/tzjzib/包名/MainActivity.kt添加如下代码：
```
    SDKInitializer.setAgreePrivacy(applicationContext, true)
    SDKInitializer.initialize(applicationContext)
```
完整代码如下：

```

import com.baidu.mapapi.SDKInitializer

class MainActivity: FlutterActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        SDKInitializer.setAgreePrivacy(applicationContext, true)
        SDKInitializer.initialize(applicationContext)

    }
    
}
```


#### ios
1. 在ios/Podfile中添加如下代码：
   pod 'BMKLocationKit'
   pod 'BaiduMapKit', '6.5.5' # 默认集成全量包
   完整代码如下：
```
target 'Runner' do
use_frameworks!
use_modular_headers!

pod 'BMKLocationKit'
pod 'BaiduMapKit', '6.5.5' # 默认集成全量包

flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
target 'RunnerTests' do
inherit! :search_paths
end
end
```
2. 在ios目录下执行
```
    pod install
```
3. 在ios/Runner/Info.plist中添加如下配置
```
<key>NSLocationWhenInUseUsageDescription</key>
<string>开启定位以便获取您的位置</string>
<key>UIBackgroundModes</key>
<array>
   <string>location</string>
</array>
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
<key>LSApplicationQueriesSchemes</key>
<array>
   <string>baidumap</string>
</array>
```
其中`UIBackgroundModes`用于连续定位，`LSApplicationQueriesSchemes`用于跳转到百度地图app。

#### 配置flutter依赖
为何指定`3.3.1`版本，因为之后的版本不再支持安卓模拟器。
```
dependencies:    
    flutter_baidu_mapapi_base: 3.3.1
    flutter_baidu_mapapi_map: 3.3.1
    flutter_baidu_mapapi_utils: 3.3.1
    flutter_baidu_mapapi_search: 3.3.1
    flutter_bmflocation: 3.3.1
```

#### main.dart中配置
```
    // 百度地图sdk初始化鉴权
    if (Platform.isIOS) {
        myLocPlugin.authAK('百度开发者平台申请得到的ios应用key');
        BMFMapSDK.setApiKeyAndCoordType(
        '百度开发者平台申请得到的ios应用key', BMF_COORD_TYPE.BD09LL);
    } else if (Platform.isAndroid) {
        /// 初始化获取Android 系统版本号，如果低于10使用TextureMapView 等于大于10使用Mapview
        await BMFAndroidVersion.initAndroidVersion();
        // Android 目前不支持接口设置Apikey,
        // 请在主工程的Manifest文件里设置，详细配置方法请参考官网(https://lbsyun.baidu.com/)demo
        BMFMapSDK.setCoordType(BMF_COORD_TYPE.BD09LL);
    }
```
#### 完成以上步骤，集成百度地图。
