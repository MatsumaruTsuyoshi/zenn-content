---
title: "UIの構築"
free: false
---

実際の動いている画面はこんな感じです。

| 立ち上げ時の記事取得待ち画面 | 記事取得後の表示画面 | Webページ表示画面 |
| ---- | ---- | ---- |
| ![](https://storage.googleapis.com/zenn-user-upload/0606e3559e296319bac1ad7c.png) | ![](https://storage.googleapis.com/zenn-user-upload/cbcea91cd23c80a132b9fb7d.png) | ![](https://storage.googleapis.com/zenn-user-upload/8612d98646413365c0c0a67d.png) |


とりあえず、ProviderScopeで全体を包んでおきます。
```dart:main.dart
void main() {
  runApp(ProviderScope(child: App()));
}
```

MaterialAppで包み、ArticleListPageクラスに飛びます。
```dart:app.dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: SafeArea(
          child: ArticleListPage(),
        ),
      ),
    );
  }
}
```

ArticleListPageクラスではFutureBuilderによって、controller.load()が実行されてAPIを使ってZennの記事データを取得しにいきます。

取得できたら_buildListでデータの表示をしていきます。実際に表示しているWidgetはArticleListItemWidgetに任せています。

:::message
useProvider(適当なProvider.notifer)でメソッドを呼び出る。useProvider(適当なProvider.select((value) => value.data)でStateNotiferのstateを監視できる。
:::

```dart:article_list_page.dart
class ArticleListPage extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final controller = useProvider(articleListControllerProvider.notifier);
    return FutureBuilder(
        future: controller.load(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            return _buildList();
          } else {
            return const Center(child: CircularProgressIndicator());
          }
        });
  }

  Widget _buildList() {
    return HookBuilder(builder: (context) {
      final data = useProvider(
          articleListControllerProvider.select((value) => value.data));
      return ListView.builder(
          itemCount: data.length,
          itemBuilder: (context, index) {
            return ArticleListItemWidget(data[index]);
          });
    });
  }
}

```

ArticleListItemWidgetクラスでは記事のタイトルとハートの数を表示しています。リストをタップしたら記事リンクからwebビューページに飛びます。

```dart:article_list_item_widget.dart
class ArticleListItemWidget extends StatelessWidget {
  final ArticleListItemModel _model;
  const ArticleListItemWidget(ArticleListItemModel model)
      : _model = model,
        super();
  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        title: Text('${_model.articleTitle}'),
        trailing: Column(children: [
          Icon(
            Icons.favorite,
            color: Colors.pinkAccent,
          ),
          Text('${_model.likedCount}'),
        ]),
        onTap: () {
          Navigator.push(context, MaterialPageRoute(builder: (context) {
            print(
                'https://zenn.dev/${_model.username}/articles/${_model.slug}');
            return ArticleWebView(
              linkUrl:
                  'https://zenn.dev/${_model.username}/articles/${_model.slug}',
            );
          }));
        },
      ),
    );
  }
}
```

ここは取得したデータを表示しているだけなので、さほど難しくはありません。
次はデータ取得に至るまでのレイヤーがどうなっているかを説明します。