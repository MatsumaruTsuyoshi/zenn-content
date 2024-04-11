---
title: "項目2：多くのコンストラクタパラメータに直面したときにはビルダーを検討する"
free: false
---
## 1. はじめに
### 1.1. 問題の紹介



### 1.2. 検討すべき要素


## 2. コンストラクタの課題
### 2.1. テレスコーピングコンストラクタ
#### 2.1.1. 概要
テレスコーピングコンストラクタは、クラスに必要なパラメータを少しずつ増やしていく方法です。

#### 2.1.2. 課題と限界

テレスコーピングコンストラクタパターンではKotlinだと課題が見えづらいので、今回はJavaで見ていきます。

NutritionFactsは加工食品の栄養成分のクラスになります。2つの必須パラメータと残りはオプションになります。

```java
public class NutritionFacts {
    private final int servingSize; // (mL) 必須
    private final int servings;    // (容器当たり) 必須
    private final int calories;    // (一食当たり) オプション
    private final int fat;         // オプション
    private final int sodium;      // オプション
    private final int carbohydrate;// オプション

    // 必須パラメータのみのコンストラクタ
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0, 0, 0, 0);
    }

    // 必須パラメータとカロリー
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0, 0, 0);
    }

    // 必須パラメータ、カロリー、脂質
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0, 0);
    }

    // 必須パラメータ、カロリー、脂質、ナトリウム
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    // すべてのパラメータを含むコンストラクタ
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

```

オプションごとにコンストラクタを作る必要があり、パラメータが増えると手に負えなくなりそうです。

インスタンスを生成する場合、すべてのパラメータを持つコンストラクタを使います。何がどこのパラメータか分かりづらいです。
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

まとめると以下の通りです。

:::message
- パラメータが増えるたびに、コンストラクタが増える
- パラメータの特定が難しい
- コードが読みづらい
:::

:::details Kotlinの場合
Kotlinでテレスコーピングコンストラクタパターンで書くとこうなります。Javaに比べるとだいぶスッキリしています。
```Kotlin
class NutritionFacts(
    val servingSize: Int, // 必須 (mL)
    val servings: Int,    // 必須 (容器当たり)
    val calories: Int = 0, // オプション (一食当たり)
    val fat: Int = 0,      // オプション
    val sodium: Int = 0,   // オプション (g/一食当たり)
    val carbohydrate: Int = 0 // オプション (g/一食当たり)
)
```

呼び出し側は以下となります。

```Kotlin
val cocaCola = NutritionFacts(servingSize = 240, servings = 8, calories = 100, fat = 0, sodium = 35, carbohydrate = 27)
```
:::




### 2.2. JavaBeansパターン
#### 2.2.1. 概要
- オブジェクトを初期化後に、セッターメソッドでパラメータを設定するアプローチ。

#### 2.2.2. 課題と限界
- 一貫性のない状態になる可能性や生成プロセスが複雑になる問題。

## 3. ビルダーパターンの紹介
### 3.1. ビルダーパターンとは
- テレスコーピングとJavaBeansの中間に位置するデザインパターンの説明。

### 3.2. 具体例
#### 3.2.1. 基礎
- ビルダーパターンを使ったシンプルな実装例。

#### 3.2.2. クラス階層での適用
- 階層的なクラス構造にビルダーパターンを適用する方法。

### 3.3. ビルダーパターンのメリット
- より多くのパラメータを持つ場合に、安全性と可読性をどのように向上させるか。

### 3.4. 安全性と可読性の向上
- 生成コードのクリアさとバグ検出のしやすさ。

## 4. ビルダーパターンの詳細
### 4.1. 不変性
- ビルダーを利用した後のオブジェクトの不変性について。

### 4.2. 検証と検査
- オブジェクト生成前のパラメータの正しさを確認する重要性。

### 4.3. 階層的デザイン
#### 4.3.1. ピザの例
- ピザとそのサブクラスを用いてビルダーパターンを説明。

#### 4.3.2. ビルダー階層
- より複雑なクラス構造にビルダーパターンをどのように適用するか。

## 5. 結論と応用
### 5.1. デザインパターンの選択
- どのシナリオでどのパターンを使うべきかについての指針。

### 5.2. ビルダーパターンの実践的価値
- 実際のプロジェクトでビルダーパターンを使用する利点と例。

## 6. 参考資料
- 本文書で言及された文献や追加情報。
