## 型クラスの解剖

型クラス（type class）パターンには３つの重要な要素がある。
**型クラス**そのもの、
特定の型に対する**インスタンス**、
そして型クラスを**使う**方法だ。

Scala では、型クラスは**暗黙の値（implicit values）**か**暗黙のパラメーター（implicit parameter）**、それと場合によって**暗黙のクラス（implicit class）** として実装される。
Scala言語の言語機能は、型クラスの要素に次のように対応する。

- トレイト（trait）： 型クラス
- 暗黙の値： 型クラスのインスタンス
- 暗黙のパラメーター： 型クラスの使用
- 暗黙のクラス： 型クラスを使いやすくするために選べるユーティリティ

この対応がどういうことか詳細に見ていこう。


### 型クラス

**型クラス**とは、ぼくたちが実装したいいくつかの機能を表すインターフェースあるいは API だ。
Scala では、型クラスは少なくとも１つの型パラメーターを持つトレイトとして表現される。
例えば、「JSON にシリアライズできる」という普遍的な挙動を、次のように表せる。

```scala mdoc:silent:reset-object
// 単純化した JSON AST を定義する
sealed trait Json
final case class JsObject(get: Map[String, Json]) extends Json
final case class JsString(get: String) extends Json
final case class JsNumber(get: Double) extends Json
final case object JsNull extends Json

// 「JSONにシリアライズできる」という挙動がこのトレイトにエンコードされている
trait JsonWriter[A] {
  def write(value: A): Json
}
```

この例では、`JsonWriter` が型クラスで、`Json` とそのサブタイプは脇役のコードだ。
`JsonWriter` のインスタンスを実装することになるときは、型パラメーター `A` は実装していくデータの具体的な型になる。

### 型クラスのインスタンス

型クラスの**インスタンス**とは、ぼくらが関心のある特定の型（Scala標準ライブラリの型でも、固有のドメインモデルの型でもよい） に、型クラスの実装を提供してくれるものだ。

Scala で型クラスを定義するには、まず具体的な実装を作り、そこに `implicit` キーワードをくっつける。

```scala mdoc:silent
final case class Person(name: String, email: String)

object JsonWriterInstances {
  implicit val stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json =
        JsString(value)
    }

  implicit val personWriter: JsonWriter[Person] =
    new JsonWriter[Person] {
      def write(value: Person): Json =
        JsObject(Map(
          "name" -> JsString(value.name),
          "email" -> JsString(value.email)
        ))
    }

  // 以下略
}
```

これは暗黙の値として知られる。


### 型クラスの使用

型クラスの**使用**とは、動作するために型クラスインスタンスを必要とする機能のことだ。
Scala では、暗黙のパラメーターとして型クラスインスタンスを受け取るメソッドを意味する。


Cats は型クラスを作りやすくするユーティリティを提供しているが、こうしたパターンを他のライブラリで見たことがあるかもしれない。
作りやすくしているパターンには２つある。**インターフェースオブジェクト**と**インターフェース構文**だ。


**インターフェースオブジェクト**

型クラスが使うインターフェースを作る一番簡単な方法は、シングルトンオブジェクトに置くメソッドだ。

```scala mdoc:silent
object Json {
  def toJson[A](value: A)(implicit w: JsonWriter[A]): Json =
    w.write(value)
}
```

このオブジェクトを使うには、関心がある型クラスインスタンスをインポートし、関連するメソッドを使う。

```scala mdoc:silent
import JsonWriterInstances._
```

```scala mdoc
Json.toJson(Person("Dave", "dave@example.com"))
```

コンパイラーは、ぼくらが呼び出した `toJson` メソッドに暗黙のパラメーターが与えられてないことを発見する。
次にこの問題を解決するために、関連する型の型クラスインスタンスを探し出して、呼び出し箇所に挿入する。

```scala mdoc:silent
Json.toJson(Person("Dave", "dave@example.com"))(personWriter)
```

**インターフェースシンタックス**

別の手段として、すでに存在する型をインターフェースメソッド[^pimping] で拡張する、**拡張メソッド** も使える。
Cats の用語では、型クラスに対する **シンタックス** と呼んでいる。

[^pimping]: 拡張メソッドのことを、型を強化（enrich）するものだとか、型をごてごて飾り立てる（pimp）ものだ、と言っているのを見たことがあるかもしれない。これらは古い用語で、ぼくらはもう使わない（訳注：pimp の原義は犯罪者を指すことから、不適切とされたようだ）。

```scala mdoc:silent
object JsonSyntax {
  implicit class JsonWriterOps[A](value: A) {
    def toJson(implicit w: JsonWriter[A]): Json =
      w.write(value)
  }
}
```

シンタックスと欲しい型に対するインスタンスを一緒にインポートすることで、シンタックスが使える。

```scala mdoc:silent
import JsonWriterInstances._
import JsonSyntax._
```

```scala mdoc
Person("Dave", "dave@example.com").toJson
```

Again, the compiler searches for candidates
for the implicit parameters and fills them in for us:

```scala mdoc:silent
Person("Dave", "dave@example.com").toJson(personWriter)
```

***implicitly* メソッド**

Scala 標準ライブラリは `implicitly` と呼ばれるジェネリックな型クラスインターフェースを提供している。
その定義はとても単純だ。

```scala
def implicitly[A](implicit value: A): A =
  value
```

暗黙のスコープから何らかの値を召喚したいときに `implicitly` を使える。
ぼくらは欲しい型を指示だけして、`implicitly` が残りの仕事をやってくれる。

```scala mdoc
import JsonWriterInstances._

implicitly[JsonWriter[String]]
```

Cat におけるほとんどの型クラスは、インスタンスを呼び出す他の方法を提供している。
だが、`implicitly` はデバッグ目的では頼みの綱になる。
コードの大まかな流れの中で `implicitly` の実行を付け加えていけば、コンパイラーが型クラスのインスタンスを見つけられること、`implicit` のエラーが起きてないことを確かめられる。
