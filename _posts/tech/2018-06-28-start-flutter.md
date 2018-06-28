# 快速开始Flutter开发

Flutter 是 Google推出并开源的移动应用开发框架，主打跨平台、高保真、高性能。开发者可以通过 Dart语言开发 App，一套代码同时运行在 iOS 和 Android平台。 Flutter提供了丰富的组件、接口，开发者可以很快地为 Flutter添加 native扩展。同时 Flutter还使用 Native引擎渲染视图，这无疑能为用户提供良好的体验。对于开发者来说，只需一套代码即可实现多平台，大大提高了产品开发效率，另外Dart运行效率较JavaScript有所提升，应用无疑会运行更为流畅。

![flutter-01](//cofcool.net/imgs/flutter-01.png)

Flutter 架构设计如下所示:

<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 480">
  <defs>
    <style>
      @import url('https://fonts.googleapis.com/css?family=Roboto:500');
      .cls-1 {
        fill: none;
        stroke: #bdbdbd;
        stroke-miterlimit: 10;
        stroke-width: 2px;
      }
      .cls-2 {
        fill: #8bc34a;
      }
      .cls-3 {
        font-size: 16px;
      }
      .cls-3,
      .cls-4 {
        fill: #fff;
      }
      .cls-3,
      .cls-4,
      .cls-8,
      .cls-9 {
        font-family: Roboto;
      }
      .cls-4 {
        font-size: 14px;
      }
      .cls-5 {
        fill: #aed581;
      }
      .cls-6 {
        fill: #f5f5f5;
      }
      .cls-7 {
        fill: #bdbdbd;
      }
      .cls-8 {
        font-size: 18px;
        fill: #607d8b;
      }
      .cls-9 {
        font-size: 20px;
        fill: #9e9e9e;
      }
      .cls-10 {
        fill: #039be5;
      }
      .cls-11 {
        fill: #81d4fa;
      }
    </style>
  </defs>
  <path class="cls-1" d="M584,91v57a12,12,0,0,1-12,12H196a12,12,0,0,0-12,12v16"/><path class="cls-1" d="M183,246v49a12,12,0,0,1-12,12H90a12,12,0,0,0-12,12v14"/><path class="cls-1" d="M183,246v49a12,12,0,0,0,12,12h64a12,12,0,0,1,12,12v14"/><path class="cls-1" d="M584,91v57a12,12,0,0,0,12,12h86a12,12,0,0,1,12,12v16"/><rect class="cls-2" x="99" y="188" width="168" height="64" rx="4" ry="4"/>
  <text class="cls-3" transform="translate(124.55 226.26)">StatelessWidget</text><rect class="cls-2" x="27" y="333" width="98" height="48" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(39.45 362.26)">AssetImage</text><rect class="cls-5" x="243" y="333" width="54" height="48" rx="4" ry="4"/><rect class="cls-5" x="42" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-5" x="105" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-5" x="168" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-5" x="231" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-6" x="607.5" y="188.5" width="167" height="63" rx="3.5" ry="3.5"/><path class="cls-7" d="M771,189a3,3,0,0,1,3,3v56a3,3,0,0,1-3,3H611a3,3,0,0,1-3-3V192a3,3,0,0,1,3-3H771m0-1H611a4,4,0,0,0-4,4v56a4,4,0,0,0,4,4H771a4,4,0,0,0,4-4V192a4,4,0,0,0-4-4h0Z"/>
  <text class="cls-8" transform="translate(683.89 220.26)">...</text><rect class="cls-6" x="500.5" y="35.5" width="167" height="63" rx="3.5" ry="3.5"/><path class="cls-7" d="M664,36a3,3,0,0,1,3,3V95a3,3,0,0,1-3,3H504a3,3,0,0,1-3-3V39a3,3,0,0,1,3-3H664m0-1H504a4,4,0,0,0-4,4V95a4,4,0,0,0,4,4H664a4,4,0,0,0,4-4V39a4,4,0,0,0-4-4h0Z"/>
  <text class="cls-9" transform="translate(552.64 75.26)">Widget</text><path class="cls-1" d="M183,344V264"/><rect class="cls-2" x="135" y="333" width="98" height="48" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(170.48 362.26)">Text</text><path class="cls-1" d="M471,246v49a12,12,0,0,1-12,12H378a12,12,0,0,0-12,12v14"/><path class="cls-1" d="M471,246v49a12,12,0,0,0,12,12h64a12,12,0,0,1,12,12v14"/><rect class="cls-10" x="315" y="333" width="98" height="48" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(333.9 362.26)">Scrollable</text><rect class="cls-11" x="531" y="333" width="54" height="48" rx="4" ry="4"/><rect class="cls-11" x="330" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-11" x="393" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-11" x="456" y="390" width="54" height="48" rx="4" ry="4"/><rect class="cls-11" x="519" y="390" width="54" height="48" rx="4" ry="4"/><path class="cls-1" d="M471,344V264"/><rect class="cls-10" x="423" y="333" width="98" height="48" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(436.29 362.26)">Animatable</text><rect class="cls-10" x="385" y="188" width="168" height="64" rx="4" ry="4"/>
  <text class="cls-3" transform="translate(415.82 226.26)">StatefulWidget</text><path class="cls-1" d="M499,160H483a12,12,0,0,0-12,12v16"/></svg>

<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 480">
  <defs>
    <style>
      @import url('https://fonts.googleapis.com/css?family=Roboto:500');
      .cls-1 {
        fill: #dcedc8;
      }
      .cls-2,
      .cls-7 {
        font-size: 20px;
      }
      .cls-2 {
        fill: #689f38;
      }
      .cls-2,
      .cls-4,
      .cls-7 {
        font-family: Roboto;
      }
      .cls-3 {
        fill: #8bc34a;
      }
      .cls-4 {
        font-size: 16px;
        fill: #fff;
      }
      .cls-5 {
        fill: #bbdefb;
      }
      .cls-6 {
        fill: #2196f3;
      }
      .cls-7 {
        fill: #1976d2;
      }
    </style>
  </defs>
  <title>layer-cake</title>
  <rect class="cls-1" x="16" y="16" width="768" height="342" rx="4" ry="4"/>
  <text class="cls-2" transform="translate(42 180)">Framework<tspan x="0" y="24">(Dart)</tspan></text>
  <rect class="cls-3" x="225" y="32" width="546" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(468.49 59.26)">Material</text>
  <rect class="cls-3" x="225" y="86" width="546" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(461.62 113.26)">Cupertino</text>
  <rect class="cls-3" x="225" y="140" width="546" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(468.79 167.26)">Widgets</text>
  <rect class="cls-3" x="225" y="194" width="546" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(461.62 221.26)">Rendering</text>
  <rect class="cls-3" x="225" y="248" width="174" height="42" rx="4" ry="4"/>
  <rect class="cls-3" x="411" y="248" width="174" height="42" rx="4" ry="4"/>
  <rect class="cls-3" x="597" y="248" width="174" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(468.43 275.26)">Painting</text>
  <text class="cls-4" transform="translate(275.31 275.26)">Animation</text>
  <text class="cls-4" transform="translate(651.84 275.26)">Gestures</text>
  <rect class="cls-5" x="16" y="390" width="768" height="72" rx="4" ry="4"/>
  <rect class="cls-6" x="225" y="405" width="174" height="42" rx="4" ry="4"/>
  <rect class="cls-6" x="411" y="405" width="174" height="42" rx="4" ry="4"/>
  <rect class="cls-6" x="597" y="405" width="174" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(482.78 432.26)">Dart</text>
  <text class="cls-4" transform="translate(296.63 432.26)">Skia</text>
  <text class="cls-4" transform="translate(668.55 432.26)">Text</text>
  <rect class="cls-3" x="225" y="302" width="546" height="42" rx="4" ry="4"/>
  <text class="cls-4" transform="translate(457.75 329.26)">Foundation</text>
  <text class="cls-7" transform="translate(42 433)">Engine (C++)</text>
</svg>

## 1. 安装

[下载SDK](https://flutter.io)

```sh
ln -s ~/bin/flutter/bin/flutter  ~/bin/flutter
```

VSCode

* Flutter
* Dart(安装Flutter时会自动安装)

```sh
$ flutter create myapp
$ cd myapp
```
目录结构如下：
```
.
├── README.md
├── android
├── ios
├── lib
├── myapp.iml
├── myapp_android.iml
├── pubspec.lock
├── pubspec.yaml
└── test
```
入口文件为`lib/main.dart`

## 2. 运行

### 2.1 Android

运行Android模拟器：
```sh
$ cd Library/Android/sdk/platform-tools
$ emulator @Pixel_2_API_25
```
运行demo：
```sh
$ flutter devices
1 connected device:

Android SDK built for x86 64 • emulator-5554 • android-x64 • Android 7.1.1 (API 25) (emulator)

$ flutter run
```
![flutter-01](//cofcool.net/imgs/flutter-02.png)
### 2.2 iOS

1. 启动模拟器
2. `$ flutter run`

![flutter-01](//cofcool.net/imgs/flutter-03.png)
```dart
import 'package:flutter/material.dart';

void main() => runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Welcome to Flutter',
      home: new Scaffold(
        appBar: new AppBar(
          title: new Text('Welcome to Flutter'),
        ),
        body: new Center(
          child: new Text('Hello World'),
        ),
      ),
    );
  }
}

```

Flutter的SDK的examples目录中提供了一些示例，更多可查看 [快速开始](https://flutter.io/get-started)。

注: *Flutter架构相关图片引用自 [flutter.io](https://flutter.io)*。