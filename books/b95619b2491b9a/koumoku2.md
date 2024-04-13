---
title: "項目2：多くのコンストラクタパラメータに直面したときにはビルダーを検討する"
free: false
---
## 1. はじめに


この章では以下コンストラクタパターンについて3つ事例とともに紹介します。

1. テレスコーピングコンストラクタパターン
2. JavaBeansパターン
3. ビルダーパターン

ここでは実装の比較をすることで、可読性や安全性の観点からビルダーパターンの方が優れていることを示しています。

:::message
Kotlinだと各コンストラクタパターンの課題が見えづらいので、今回はJavaで見ていきます。
:::

## 2. コンストラクタパターン
### 2.1. テレスコーピングコンストラクタ

#### 2.1.1. 概要

このパターンの名前は、望遠鏡（telescope）が段階的に伸縮する様子に例えられています。テレスコーピングコンストラクタは、必須パラメータだけでなくオプショナルなパラメータも段階的に追加していく方法で、それぞれの新しいパラメータに対応するために新たなコンストラクタを追加していくという特徴があります。このように徐々に「伸びていく」コンストラクタ群が望遠鏡のように見えることから、この名前がつけられました。

#### 2.1.2. 事例

NutritionFactsは加工食品の栄養成分のクラスになります。2つの必須パラメータと残りはオプションになります。それぞれのパラメータごとにコンストラクタがあります。

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

呼び出し側は以下の通りです。
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
インスタンスを生成する場合、何がどこのパラメータか分かりづらいです。


#### 2.1.3. 課題 

- パラメータが増えるたびに、コンストラクタが増える
- パラメータの特定が難しい
- コードが読みづらい


### 2.2. JavaBeansパターン
#### 2.2.1. 概要
「JavaBeans」という名前は、Javaプラットフォームにおいて再利用可能なソフトウェアコンポーネントを指すために用いられます。JavaBeans規格に従って設計されたクラスは、プロパティ、イベント、メソッドを備えており、ビジュアルプログラミング環境で容易に操作できるようになっています。JavaBeansパターンとは、この規格に従い、プロパティに対してセッターとゲッターメソッドを提供することにより、オブジェクトの状態を柔軟に管理できるようにする設計手法です。この名前の由来は、Javaプログラミング言語と、コーヒー豆（beans）にちなんだプレイフルな命名から来ています。

もっと詳しく知りたい方は[こちら](https://blog1.mammb.com/entry/2019/12/06/090000)を参考にしてください。

#### 2.2.2. 事例

さきほどと同じようにNutritionFactsは加工食品の栄養成分のクラスを作成します。

```java
public class NutritionFacts {
    private int servingSize = -1; // 必須：デフォルト値はない
    private int servings = -1;    // 必須：デフォルト値はない
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}

    // セッターメソッド
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```

呼び出し側は以下の通りです。パラメータ登録が複数の呼び出しに分割されているので呼び忘れなどの懸念がありそうです。
```java
NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);      // mL
        cocaCola.setServings(8);           // 容器当たり
        cocaCola.setCalories(100);         // 一食当たり
        cocaCola.setFat(0);                // オプション
        cocaCola.setSodium(35);            // オプション
        cocaCola.setCarbohydrate(27);      // オプション
```

#### 2.2.3. 課題

- 不完全な状態でのオブジェクトの利用になる可能性がある
- クラスを不変にできない

### 2.3. ビルダーパターン

#### 2.3.1. 概要
ビルダーパターンの名前は、そのパターンの性質から来ています。このパターンは、最終的なオブジェクトを段階的に構築するビルダー（建設者）に喩えられます。具体的には、ビルダークラスが途中の構築状態をカプセル化し、最終的なオブジェクトの構築を一連のステップを通じて行うという特徴があります。実際の建設現場で建築士が設計図に基づき部品を組み立てるプロセスに似ており、そのアナロジーからこの名前がつけられています。

#### 2.3.2. 事例
以下はNutritionFactsクラスのビルダーパターンを使用した実装例です。この例では、必要なパラメータを段階的に設定し、最終的に完全なオブジェクトを構築します。

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 必須パラメータ
        private final int servingSize;
        private final int servings;

        // オプション
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
呼び出し側は以下の通りです。必須パラメータとオプションが分かれている点や、生成が分割されていない点も良さそうです。
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .fat(0)
    .sodium(35)
    .carbohydrate(27)
    .build();
```

従って、テレスコーピングコンストラクタよりもコードが読みやすく、JavaBeansよりも安全にインスタンスを生成できます。

#### 2.3.3. 課題
- 冗長になる

## 3. Kotlinで考えてみる

KotlinでもJavaと同様に、コンストラクタを用いてクラスのインスタンスを生成します。ただし、Kotlinでは「プライマリコンストラクタ」と「セカンダリコンストラクタ」という、二つの異なるタイプのコンストラクタが存在します。

:::details プライマリコンストラクタとセカンダリコンストラクタについて

Javaと同じようにコーディングしたければ、セカンダリコンストラクタだけを定義すればできます。

```kotlin
class NutritionFacts(val servingSize: Int, val servings: Int) { // プライマリコンストラクタ
    var calories: Int = 0
    var fat: Int = 0
    var sodium: Int = 0
    var carbohydrate: Int = 0

    // セカンダリコンストラクタ: すべてのオプショナルパラメータを追加
    constructor(servingSize: Int, servings: Int, calories: Int, fat: Int, sodium: Int, carbohydrate: Int) : this(servingSize, servings, calories, fat, sodium) {
        this.carbohydrate = carbohydrate
    }
}
```
:::

さきほどのNutritionFactsをKotlinで書くとこうなります。
```kotlin
data class NutritionFacts(
    val servingSize: Int,
    val servings: Int,
    val calories: Int = 0,
    val fat: Int = 0,
    val sodium: Int = 0,
    val carbohydrate: Int = 0
)
```

data classは、データ保持用のクラスに最適で、自動的にequals(), hashCode(), toString()、copy()などのメソッドを生成します。

呼び出し側は以下の通りです。
```Kotlin
val cocaCola = NutritionFacts(servingSize = 240, servings = 8, calories = 100, fat = 0, sodium = 35, carbohydrate = 27)
```

Kotlinであれば、ボイラープレートコードを追加することなく実装できるのでよりシンプルになります。

:::message
裏側ではボイラープレートコードを生成してくれていることでシンプルさを実現しているのでKotlinを使うとしてもJavaの理解が深いとより良いですね。
:::

## 4. まとめ

比較すると、多くのパラメータを持つクラスを設計する際はビルダーパターンが良い選択になりそうです。