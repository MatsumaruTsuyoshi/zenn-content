---
title: "FlutterでGoldenTestを試してみた"
emoji: "🌎"
type: "tech"
topics:
  - "flutter"
  - "test"
  - "golden"
published: true
published_at: "2021-10-18 20:34"
---

# GoldenTestとは

WidgetTestの手法の一つになります。

いわゆるFlutterのWidget TestとGolden Test比較すると、

Widget Test　-> WidgetTreeを確認
Golden Test -> 画像を確認

となります。

従って、Golden Testの特徴としてはこんな感じになります。
-  ユーザーが使用する画面そのものをテストできる
-  速い（らしい）
-  画面サイズが変わっても対応できる

なんといっても**ユーザーが使用する画面そのものをテストできる**のはメリットがあると思います。

例えば、WidgetTree上ではTextがきちんと表示されているけど、実際は文字が隠れちゃっているなんてことが有りうるからです。（画像の字が小さくてすみません・・・・）

| 文字が隠れている | 文字が全部表示 | 
| ---- | ---- | 
| ![](https://storage.googleapis.com/zenn-user-upload/ddf23fd6fb5481c0999d50b7.png =300x) | ![](https://storage.googleapis.com/zenn-user-upload/70680a90777d5671c169e990.png =300x) | 

メリットを分かってもらえたところで、簡単なGoldenTestをやっていきます。



# GoldenTestの手順
今回は**カウンターアプリでWidgetのカラーが違う場合をテストします。**
ライブラリーはgolden_toolkitを使います。
https://pub.dev/packages/golden_toolkit

1.　dev_dependenciesにgolden_toolkitを追加
2.　testフォルダにgolden_test.dartを作成
3.　golden_test.dartを編集
4.　Golden作成（マスターのスクリーンショット作成）
5.　実際にテストしてみる（OK編）
6.　実際にテストしてみる（NG編）

今回のソースコードはGithubにおいてあるので詳細気になる方は見てください。
https://github.com/MatsumaruTsuyoshi/flutter_golden_test_demo

# dev_dependenciesにgolden_toolkitを追加
pubspec.ymalに[golden_toolkit](https://pub.dev/packages/golden_toolkit)を追加してpub getしましょう。
```yaml:pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  golden_toolkit: ^0.11.0
```

# testフォルダにgolden_test.dartを作成
ファイル名はgolden_test.dartでなくても良いです。

# golden_test.dartを編集
中身を編集していきます。
```dart:golden_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:golden_toolkit/golden_toolkit.dart';
import 'package:flutter_golden_test_demo/main.dart';

void main() {
  testGoldens('app', (WidgetTester tester) async {
    //デバイスの画面サイズ
    final size = Size(415, 896);
    
    //第一引数はどのWidgetをビルドするのか指定、どのサイズにビルドするかがsurfaceSize
    await tester.pumpWidgetBuilder(MyApp(), surfaceSize: size);
    
    //マスターのスクリーンショットと同じかテストする
    await screenMatchesGolden(tester, 'myApp');
  });
}
```

デバイスの画面サイズを変えることでレスポンシブなテストが可能になります。

# Golden作成（マスターのスクリーンショット作成）
Golden作成とはマスターのスクリーンショット作成のことを示しています。Golden = マスターのスクリーンショットらしいです。

ターミナルで以下コマンドを叩いてください。
```command
flutter test --update-goldens
```

成功すれば、testフォルダにgoldensファイルが作成され、myApp.pngができているはずです。

| ビルド画面 | Golden(myApp.png) |
| ---- | ---- |
| ![](https://storage.googleapis.com/zenn-user-upload/ed48c41d63c699a21f4da894.png =300x) | ![](https://storage.googleapis.com/zenn-user-upload/c4f529ffd7fb38fbc21b919b.png =250x) |

フォントを読み込めていないので文字化けしていますが、今回はこのまま進めます。

# 実際にテストしてみる（OK編）
さきほど、Goldenを作成したまま変更せずにテストを実行してみます。当然テストはパスできるはずです。

以下コマンドでテストします。
```
flutter test
```
結果は

```
All tests passed! 
```
のはず。

##  実際にテストしてみる（NG編）
Goldenは変えずに、main.dartのプライマリーカラーを変えてみます。
![](https://storage.googleapis.com/zenn-user-upload/f833630e716636f0a2abe101.png =300x)

以下コマンドでテストします。
```
flutter test
```
結果は

```
Some tests failed. 
```
のはず。

テストに失敗すると、testフォルダにfailuresフォルダが作成され、その配下に何が違っていたかのスクショができあがっているはずです。
それを見ると、差分をちゃんと判定してくれています。

| myApp_masterImage.png | myApp_testImage.png | myApp_maskedDiff.png | myApp_isolatedDiff.png |
| ---- | ---- | ---- | ---- |
| ![](https://storage.googleapis.com/zenn-user-upload/fc2bef37762b0da4db3089ab.png) | ![](https://storage.googleapis.com/zenn-user-upload/7964fa3afa36013a39354dfb.png) | ![](https://storage.googleapis.com/zenn-user-upload/c5b6b47a750e5b51c1aaef11.png) | ![](https://storage.googleapis.com/zenn-user-upload/799b170dd2e3300e9c9e8a88.png) |


# 終わりに
今回はお試しに簡単なテストをしてみましたが、ライブラリを使えばかなりお手軽でした。






