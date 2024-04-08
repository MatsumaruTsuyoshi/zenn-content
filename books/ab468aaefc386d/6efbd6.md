---
title: "ディレクトリ構成"
free: false
---

# ライブラリ構成
- lib
  - core
    - domain
      - entity
        - article.dart
      - repository
        - article_repository.dart
    - gateway
      - article_repository_impl
        - article_repository_impl.dart
      - datasource
        - article_datasource.dart
      - datasource_impl
        - article_datasource_impl.dart
      - provider
        - article_datasource_provider.dart
        - article_repository_provider.dart

  - modules
    - article_list
      - controller
        - article_list_controller.dart
        - article_list_controller_provider.dart
        - article_list_state.dart
        - article_list_state.freezed.dart
      - model
        - article_list_item_model.dart
      - widget
        - article_list_item_widget.dart
        - article_web_view.dart
      - article_list_page.dart
  - app.dart
  - main.dart



# フォルダ名の命名規則

libは大きくcoreとmodulesの二つに分かれています。coreはデータを保持するものとデータベースと接続する汎用的なものを持っています。modulesはUIとインターフェイス（UIと実際のデータを繋げる役割）を持っています。

coreはdomainとgatewayに分かれており、domainはデータを保持するクラスを持っています。gatewayはデータベースとの接合を担っています。

domainにはentityとrepositoryに分かれており、entityはデータを保持するクラスを持ちます。今回、article.dartには記事データのリンク先やユーザー名、ハートの数を持ちます。repositoryは実態をもっておらず、抽象的なものです。ただ、repositoryという概念をひとつ設けることで、データを欲しいと呼び出す側はローカルDB、リモートDBを気にしなくて良いし、テストも簡単にできるので便宜上存在しています。

ほにゃららimplのimplは「implementation」の略で、訳すと「実装」という意味です。なので、語尾にimplが付く箇所は、データ取得をする実装が書かれています。

modulesの配下にあるcontrollerはページと実際にデータを取得するロジックの架け橋を担っています。modelは取得した記事データをリストとして保持しています。widgetはUIに表示される部分を持っています。




