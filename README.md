## 要旨

GitHubが公開している GitHub Packages registryにかんするドキュメント ["Working with the Gradle registry"](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)に書いてある手順を適用してMavenレポジトリを作り、自作したプロジェクト２つがMavenレポジトリを介してjarファイルを受け渡しするという構成を作ることに成功した。プログラミング言語はJava、ビルドツールはGradle。事例として報告します。

## 解決したい問題

Java+Gradleなプロジェクトを２つ自作した。片方を「libraryプロジェクト」、もう一つを「consumerプロジェクト」と略記する。libraryプロジェクトの成果物としてjarファイルを生成し、consumerプロジェクトがそのjarファイルを参照する、という形だ。




ある日のことTom Gregoryの記事 ["How to use Gradle api vs. implementation dependencies"](https://tomgregory.com/how-to-use-gradle-api-vs-implementation-dependencies-with-the-java-library-plugin/)を読んでサンプルコードを写経した。GitHubプロジェクトを二つ作った。記事が示したコードをそのまま打ち込んだだけ。

- https://github.com/kazurayam/gradle-java-library-plugin-library
- https://github.com/kazurayam/gradle-java-library-plugin-consumer

libraryプロジェクトはjarを生成して自PCのローカルディスク上にあるMavenレポジトリのキャッシュ（mavenLocalともいう）に保存する。consumerプロジェクトはmavenLocalにアクセスしてlibraryプロジェクトのjarを参照する。consumerプロジェクトのtestタスクを実行すれば成功した。

```
$ cd gradle-java-library-plugin-consumer
$ gradle test
Starting a Gradle Daemon (subsequent builds will be faster)

BUILD SUCCESSFUL in 9s
1 actionable task: 1 executed
```

内部では二つのプロジェクトは下図のような関係にある。

![diagram1](docs/diagrams/out/01_libary_mavenLocal_consumer/diagram1.png)


さて、わたしは某日、[GitHub Actions](https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions#create-an-example-workflow)を使って自動化テストを実践してみようと思った。CI環境つまりリモートマシン上で `gradle test` コマンドを実行する。具体的にはconsumerプロジェクトに下記ファイルを追加した。

- [.github/workflows/tests.yml](https://github.com/kazurayam/gradle-java-library-plugin-consumer/blob/develop/.github/workflows/tests.yml)

consumerプロジェクトのdevelopブランチをpushすればGitHub Actionsのワークフローで `gradle testが` 実行される。いざ実行したら失敗した。

![consumer_test_on_CI_failed](docs/images/consumer_test_on_CI_failed.png)

なぜ失敗？CI環境で起動されたconsumerプロジェクトのビルドがlibraryプロジェクトのjarを得ようとしてmavenLocalレポジトリを参照したが見つからなかったから。mavenLocalを参照するのではダメだ。

もう一つの方法としてMaven Centralレポジトリを使うこともできるが気がすすまない。Maven Centralは表舞台だ。人様の目に晒すに値すると自分が思える程度に真剣に作ったプロジェクトならMaven Centralに上げるのもいい。しかしプログラミングの学習ためにちょっと作った程度の軽いものをMaven Centralに上げるのは適当でない。それにMaven Centralに向けてpublishすると、コマンドを実行してからサイトでjarが実際に公開されるまで二、三日も待たされる（なんで待たされるのか理由をわたしは知らない）。三日も待たされるからMaven Centralを学習の道具として使うべきでない。

##


