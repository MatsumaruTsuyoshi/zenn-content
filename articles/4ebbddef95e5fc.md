---
title: "keystoreを変更したときアプリリリースするまでにやること"
emoji: "😽"
type: "tech"
topics: []
published: false
---




1. 新しくkeystoreを作成する
2. keystoreをpemファイルにしてGooglePlayConsoleから問い合わせる
3. ローカルマシンのkeystore.jksとkeystore.propertiesを更新する
4. アプリへの署名を行う




詰まったところ１
・新しいkeystore.jksとkeystore.propertiesを使うとandroidでビルドできなくなった。
・原因はkeyaliasが間違っていた。keystoreを作成するコマンドでkeyaliasを指定していたのに忘れていた。
そのコマンドが↓
```
$ keytool -export -rfc -keystore keystore.jks -alias upload -file upload_certificate.pem
```
https://support.google.com/googleplay/android-developer/answer/9842756?visit_id=638095466393621110-2600470827&rd=1

シンプルに正しいkeyaliasをkeystore.propertiesに入力したらビルドできた

詰まったところ２
GooglePlayConsoleにアプリアップロードできなくなった。
エラー内容は以下。
```
アップロードした App Bundle の署名に使われている証明書のフィンガープリント・・・・
```
原因はエラー通りフィンガープリントが違うと怒られている。
まずはフィンガープリントがどうなっているか確認
以下コマンドで手元なりクラウドでビルドされたアプリのフィンガープリントを確認します。
ここで表示されたフィンガープリントとGooglePlayConsoleのアプリ完全性タブのフィンガープリントが一致しているか確認する。違っていたら以下手順をすればOK

```
❯ keytool -printcert -jarfile build/app/outputs/bundle/prodRelease/app-prod-release.aab
署名者番号1:
証明書#1:
所有者: 
発行者: 
シリアル番号: 
有効期間の開始日:
証明書のフィンガプリント:
         SHA1: 
         SHA256: 
署名アルゴリズム名: 

```

android studioでandroidのプロジェクトを開いて以下手順でアプリへの署名を行わないといけないみたい。

https://developer.android.com/studio/publish/app-signing?hl=ja#sign_release