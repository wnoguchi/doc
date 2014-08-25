Apache Maven
===============

ビルド・テスト・デプロイのライフサイクルを管理するツール。

GettingStarted
----------------

### 簡単な流れ

```
mvn archetype:create -DgroupId=com.example -DartifactId=sample
mvn compile
mvn test
mvn javadoc:javadoc
mvn site
```

### 依存性の解決

ここで検索する。

[Maven Repository: commons lang](http://mvnrepository.com/search.html?query=commons+lang)

```xml
<dependency>
	<groupId>commons-lang</groupId>
	<artifactId>commons-lang</artifactId>
	<version>2.6</version>
</dependency>
```

### プラグインについて

[Maven – Available Plugins](http://maven.apache.org/plugins/index.html)

```
<dependency>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.1</version>
</dependency>
```

### リモートリポジトリ

Seasar2 の場合はこんな感じ？わかんない。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.pg1x</groupId>
  <artifactId>s2sample</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>s2sample</name>
  <url>http://blog.pg1x.com/</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <repositories>
    <repository>
      <id>maven.seasar.org</id>
      <name>The Seasar Foundation Maven Repository</name>
      <url>http://maven.seasar.org/maven2</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>commons-lang</groupId>
      <artifactId>commons-lang</artifactId>
      <version>2.6</version>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

### マルチモジュールプロジェクト

```
mvn archetype:create -DgroupId=com.pg1x.myproject -DartifactId=myproject
cd myproject
mvn archetype:create -DgroupId=com.pg1x.myproject.sub -DartifactId=myproject-sub
mvn archetype:create -DgroupId=com.pg1x.myproject.main -DartifactId=myproject-main
```

エラーになってる！

```
[INFO] --- maven-archetype-plugin:2.2:create (default-cli) @ standalone-pom ---
[WARNING] This goal is deprecated. Please use mvn archetype:generate instead
[INFO] Defaulting package to group ID: com.pg1x.myproject
Downloading: http://repo.maven.apache.org/maven2/org/apache/maven/archetypes/maven-archetype-quickstart/maven-metadata.xml
Downloaded: http://repo.maven.apache.org/maven2/org/apache/maven/archetypes/maven-archetype-quickstart/maven-metadata.xml (531 B at 0.4 KB/sec)
```

```
mvn archetype:generate -DgroupId=com.pg1x.myproject -DartifactId=myproject
cd myproject
mvn archetype:generate -DgroupId=com.pg1x.myproject.sub -DartifactId=myproject-sub
mvn archetype:generate -DgroupId=com.pg1x.myproject.main -DartifactId=myproject-main
```

するとずらずらと出力される。

```
(snip)

439: remote -> org.apache.maven.archetypes:maven-archetype-quickstart (An archetype which contains a sample Maven project.)

(snip)

Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 439:
```

そのままエンター。
とりあえずいけた。

ここが問題じゃなかった。

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-archetype-plugin:2.2:generate (default-cli) on project myproject: Unable to add module to the current project as it is not of packaging type 'pom' -> [Help 1]
```

`pom.xml` を編集して `<packaging>pom</packaging>` とする。

そして再トライ。

うまくいったうまくいった。

```
  <modules>
    <module>myproject-sub</module>
    <module>myproject-main</module>
  </modules>
```

は自動でやってくれた。

サブモジュールへの親 pom の指定もうまくやってくれた。

```
  <parent>
    <groupId>com.pg1x.myproject</groupId>
    <artifactId>myproject</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
```

進化してるなぁ。

そして

```
% mvn compile
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO]
[INFO] myproject
[INFO] myproject-sub
[INFO] myproject-main
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject-sub 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ myproject-sub ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/wnoguchi/Documents/tmp/maven/myproject/myproject-sub/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:2.5.1:compile (default-compile) @ myproject-sub ---
[INFO] Compiling 1 source file to /Users/wnoguchi/Documents/tmp/maven/myproject/myproject-sub/target/classes
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject-main 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ myproject-main ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/wnoguchi/Documents/tmp/maven/myproject/myproject-main/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:2.5.1:compile (default-compile) @ myproject-main ---
[INFO] Compiling 1 source file to /Users/wnoguchi/Documents/tmp/maven/myproject/myproject-main/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] myproject .......................................... SUCCESS [  0.001 s]
[INFO] myproject-sub ...................................... SUCCESS [  0.707 s]
[INFO] myproject-main ..................................... SUCCESS [  0.028 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.821 s
[INFO] Finished at: 2014-08-26T00:09:05+09:00
[INFO] Final Memory: 12M/306M
[INFO] ------------------------------------------------------------------------
```

### Eclipse 連携

- `M2_REPO` 変数の設定。

```
mvn -Declipse.workspace=/Users/wnoguchi/Documents/workspace eclipse:add-maven-repo
```

- Maven プロジェクトを Eclipse プロジェクトに変換

```
mvn eclipse:eclipse
```

Tips
------

### リポジトリ検索サービス

[Maven Repository: Search/Browse/Explore](http://mvnrepository.com/)

### 依存性をダウンロードしたい

```
mvn dependency:copy-dependencies
```

### Mac で文字化けする

```xml
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <encoding>UTF-8</encoding>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

そして `.zshrc` には

```
export _JAVA_OPTIONS="-Dfile.encoding=UTF-8"
```

を指定する。こんなグローバルに指定しちゃっていいの？

うーん、確かに解決したけどなんか違う気がする。

見ているものが間違っているようだ。

まずは `JAVA_HOME` を設定しようか。

```
export M2_HOME=/opt/apache/apache-maven/latest
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_11.jdk/Contents/Home/
```

最近の Java プロダクト（Eclipse とか JMeter とか Maven とか）は勝手に Java のある場所を
補完してくれちゃうのでこういうの省略するとあんまり良くないみたいね。
たぶん Mac でデフォルトで入ってる 1.6 のJVM使おうとしてたんだろうか・・・。

こうしたらわざわざエンコーディング指定をしなくてもちゃんと文字化けせずに処理してくれた。

```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

試しに JAVA_HOME を unset してみると Java 1.6 を使おうとしている。

```
[wnoguchi@noguchiwataru-no-MacBook-Pro] ~
% echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/jdk1.8.0_11.jdk/Contents/Home/
[wnoguchi@noguchiwataru-no-MacBook-Pro] ~
% unset JAVA_HOME
[wnoguchi@noguchiwataru-no-MacBook-Pro] ~
% mvn -version
Apache Maven 3.2.2 (45f7c06d68e745d05611f7fd14efb6594181933e; 2014-06-17T22:51:42+09:00)
Maven home: /opt/apache/apache-maven/latest
Java version: 1.6.0_65, vendor: Apple Inc.
Java home: /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
Default locale: ja_JP, platform encoding: SJIS
OS name: "mac os x", version: "10.9.4", arch: "x86_64", family: "mac"
```

しかも platform encoding が SJIS になっちゃってるし。最悪だ。

まずは Java 1.6 のVMの compliant level が 1.8 に対応しているわけがないのでそれを修正した上で、
先ほどの `_JAVA_OPTIONS` でエンコーディング指定する。

あまりいけてない。

結論としてはちゃんと JAVA_HOME を設定しましょうということですね。

References
------------

1. [Maven | TECHSCORE(テックスコア)](http://www.techscore.com/tech/Java/ApacheJakarta/Maven/index/)
