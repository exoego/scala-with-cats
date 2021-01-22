## バージョンについて {-}

本書は Scala @SCALA_VERSION@ と Cats @CATS_VERSION@ について書かれている。
関連するライブラリと設定[^sbt-version] を含んだ、最小限の `build.sbt` はこうなる。

```scala
scalaVersion := "@SCALA_VERSION@"

libraryDependencies +=
  "org.typelevel" %% "cats-core" % "@CATS_VERSION@"

scalacOptions ++= Seq(
  "-Xfatal-warnings"
)
```

[^sbt-version]: SBT 1.0.0 以上を使っている想定。

### テンプレートプロジェクト {-}

始めるにあたって便利なので、Giter8[^giter8] テンプレートを作っておいた。
テンプレートをクローンするには、次のように入力する。

```bash
$ sbt new scalawithcats/cats-seed.g8
```

これによって、Cats を依存関係に追加した実験用のプロジェクトが作られる。
生成された `README.md` を見ると、サンプルコードの動かし方や Scala のインタラクティブシェルの始め方が分かる。

`cats-seed` テンプレートは最小限だ。
最初からもっと全部入りのものをお好みなら、Typelevel[^typelevel] の `sbt-catalysts` テンプレートをチェックしてみよう。

```bash
$ sbt new typelevel/sbt-catalysts.g8
```

これにより、一連のライブラリ、コンパイラープラグイン、ユニットテストやドキュメントのテンプレートまで込みのプロジェクトが作られる。
より詳しく知るには、[catalysts][link-catalysts] や [sbt-catalysts][link-sbt-catalysts] のプロジェクトページを見よう。


[^giter8]: 訳注： GitHub などで公開されたテンプレートから、雛形ファイルやディレクリ構成を自動生成する CLI ツール。主に Scala プロジェクトで使われ、SBT から使用できる。
[^typelevel]: 訳注： Typelevel は、Scala で関数型プログラミングを使いやすくし、普及させることを志向されているプログラマーのコミュニティ。Cats などの Scala ライブラリのほか、Scala に提案する先進的な言語機能を開発したりしている。