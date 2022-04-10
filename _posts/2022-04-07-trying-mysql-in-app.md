---
title: "Azure App Service の MySQL In App を Web アプリケーションのデータベースとして使用してみるまでの記録"
author_name: "Toxumuharu"
tags:
    - AppService
    - MySQLInApp
    - php
    - MySQL
---

# このドキュメントの内容
Azure App Service の MySQL In App をデータベースとして使用する事を目的としてアプリケーションを作成します。例として、MySQL in App を使用したアクセスされた回数を表示するだけの簡単な Web アプリケーションをデプロイします。

MySQL in App Test Web App - [https://toxumuharu-mysql-test-web-app.azurewebsites.net/](https://toxumuharu-mysql-test-web-app.azurewebsites.net/)

![2022-04-07-trying-mysql-in-app](/media/20220407/16.png)

# MySQL In App とは
MySQL In App とは、Windows ベースの環境へ Web アプリケーションをデプロイする場合に、アプリケーションのデータベースとして MySQL が追加料金なしで使える App Service の機能です。

Title: Announcing MySQL in-app for Web Apps (Windows)<br>
URL: https://azure.github.io/AppService/2016/08/18/Announcing-MySQL-in-app-for-Web-Apps-(Windows).html

## MySQL in App の注意点
https://azure.github.io/AppService/2016/08/18/Announcing-MySQL-in-app-for-Web-Apps-(Windows).html にあるプレビュー段階時の Limitations の項目に挙げられている制限事項です。
> Limitations
> - Auto scale feature is not supported since MySQL currently runs on on a single instance .
> - Enabling Local cache is not supported.
> - MySQL database cannot be accessed remotely. You can only access your database content using PHPMyadmin or using MySQL utilities in KUDU debug console. This is described in detail below.
> - WordPress and Web App + MySQL templates currently support MySQL in-app in the create experience.We are working on bringing this feature in for other MySQL based applications in Web category for Azure marketplace.

超ざっくりですが制限事項として以下を気をつけて下さいね、と言う内容です。
- 自動スケールはサポートされていません。
- ローカルキャッシュの有効化はサポートされていません。
- MySQL データベースにリモートでアクセスすることはできません。
- WordPress および WebApp + MySQL テンプレートを、Azure マーケットプレイスの Web カテゴリにある他の MySQL ベースのアプリケーションに導入するよう取り組んでいます。

運用向けではなく、開発やテスト向けである事が分かります。それでは上記を注意しながら実際に MySQL in App を使用した簡単な Web アプリケーションを作成してみます。

# MySQL In App を実際に使用した Web アプリをデプロイする。
## 作成する Web アプリケーションの仕様
- MySQL in App をデータベースとして使用する
- index.php へのアクセス回数が表示される

## 作成の流れ
1. App Service で Windows 環境の Web App を作成する。
2. データベースを作成する
3. データベースを使用したコードを作成する
4. デプロイする

それでは作成します。

## 1. App Service で Windows 環境の Web App を作成する。
Azure Portal より Web App を作成します。Operating System を忘れず「Windows」に設定します。

![2022-04-07-trying-mysql-in-app](/media/20220407/1.png)

![2022-04-07-trying-mysql-in-app](/media/20220407/2.png)

Web App がデフォルトの状態でデプロイされますので、一度開いておきましょう。[こちら](https://toxumuharu.github.io/2022/03/23/2002-error-of-mysql-in-app-on-appservice.html)で公開しているエラーを回避することにも繋がります。

![2022-04-07-trying-mysql-in-app](/media/20220407/3.png)

Web App がデプロイされたデフォルトの状態です。

![2022-04-07-trying-mysql-in-app](/media/20220407/4.png)


## 2. データベースを作成する
何回アクセスされたかを記録するため、データベースを作成します。
作成した Web App の項目「Settings」より、「MySQL in App」を選択します。

![2022-04-07-trying-mysql-in-app](/media/20220407/5.png)

項目「MySQL in App」を「ON」に設定し、「Save」を押下し保存します。

![2022-04-07-trying-mysql-in-app](/media/20220407/6.png)

「↗︎Manage」が押下できるようになります。押下してデータベースを編集します。

![2022-04-07-trying-mysql-in-app](/media/20220407/7.png)

私のようなデータベースを作成したことの無い人間は初めて見る画面が立ち上がります。
  
検索してみると、データベース界隈、Web 開発界隈ではよくある画面なのか「見慣れた画面」「よく見る画面」「ここから先の操作はご存じだと思いますので割愛します」など出てきました。存じませんので割愛していただきたく無いです。

![2022-04-07-trying-mysql-in-app](/media/20220407/8.png)

それではデータベースを作成しましょう。
  
左ペインのデータベースの構造より項目「New」を選択後、Create database の画面にてデータベース名、Collation を設定します。今回は以下の内容で作成します。
- データベース名: test_database
- Collation: `utf8mb4_general_ci`
    - Collation に関しては、投稿作成時の MySQL in App の MySQL のバージョンは `5.7.9` でした (Toxumuharu 調べ)。MySQL 5.7 にてひらがな/カタカナ/漢字が全て区別される `utf8mb4_general_ci` を設定しました。

データベース名、Collation を設定した後「Create」を押下し、データベースを作成します。

![2022-04-07-trying-mysql-in-app](/media/20220407/9.png)

データベースが作成されると、続いてテーブルを作成する画面になります。Name と Number of columns を設定します。今回は次の内容で作成します。
- Name: access
- Number of columns: 2
    - 詳細は次で述べますが、今回はページのアクセス データに関するデータベースですので、次の2 つの列があれば良いと考え、2 としました。データベース全く無知なので、どのように設定すべきであるかコメントございましたら是非ご連絡くださいませ。
        - データの通し番号
        - アクセス時のタイム スタンプ

![2022-04-07-trying-mysql-in-app](/media/20220407/10.png)

テーブルが作成されると、続いてテーブルの列の詳細を設定する画面になります。先ほど設定した 2 つの列に対して、それぞれ設定を行います。
1. `code` - Type: INT、Index: Primary (自動で設定されます)、A_I: ON (チェックを入れる)
    - データを管理するための管理番号として設定する。自然数である必要があるため型は INT を設定します。
2. `timestamp` - Type: TIMESTAMP
    - アプリケーションの要件からするとこの列はもはや必要ないが、アクセスされた日時を念の為保存するための列。アクセスされた日時のため TIMESTAMP 型として設定します。

![2022-04-07-trying-mysql-in-app](/media/20220407/11.png)

`code` の A_I (恐らく AUTO_INCREMENT) にチェックを入れると以下のようなダイアログが表示されましたが、そのまま「Go」を押下します。

![2022-04-07-trying-mysql-in-app](/media/20220407/12.png)

上記を設定し、「Save」を押下すると今回使用するためのデータベースの設定が終了します。

## 3. データベースを使用したコードを作成する
それでは次のコードをコピーするなどして `index.php` として下さい。

先ほど作成したデータベース `test_database` に対し下記の内容を行なっています。
- アクセスした際にレコードを更新する
- 更新後、一番後に追加されたレコード (つまり上記の更新されたレコード) の通し番号を表示する

サンプルかつ私が Web の技術を勉強中ですので凝ったことが出来ず、非常にシンプルな内容となっています。ご参考までに。

```
<!DOCTYPE html>
<html>

<head>
  <!-- Bootstrap -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
</head>

<header>
  <div class="p-3 mb-2 bg-primary text-white text-center">
    <h1>MySQL in App Test Web App</h1>
  </div>
</header>

<body>
  <div class="jumbotron">
    <div class="container">
      <h2>Welcome to this page!</h2>

      <?php
      // azure database login
      $azure_mysql_connstr = $_SERVER["MYSQLCONNSTR_localdb"];
      $azure_mysql_connstr_match = preg_match(
        "/" .
          "Database=(?<database>.+);" .
          "Data Source=(?<datasource>.+);" .
          "User Id=(?<userid>.+);" .
          "Password=(?<password>.+)" .
          "/u",
        $azure_mysql_connstr,
        $_
      );

      $dsn = "mysql:host={$_["datasource"]};dbname=test_database;charset=utf8";
      $user = $_["userid"];
      $password = $_["password"];

      $dbh = new PDO($dsn, $user, $password);
      $dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

      $sql = "INSERT INTO access(timestamp) VALUES(?)";
      $stmt = $dbh->prepare($sql);
      $data[] = date('Y-m-d H:i:s');
      $stmt->execute($data);

      $sql = "SELECT MAX(code) AS LargestPrice FROM access";
      $stmt = $dbh->prepare($sql);
      $stmt->execute();

      $rec = $stmt->fetch(PDO::FETCH_ASSOC);
      echo "このページのアクセス回数は";
      echo $rec["LargestPrice"];
      echo "です。";
      ?>
    </div>
  </div>
</body>

</html>
```

重要な部分のみ抜粋しますと、下記のコードにより MySQL in App にログインし、SQL クエリを実行出来るようです。
```
      // azure database login
      $azure_mysql_connstr = $_SERVER["MYSQLCONNSTR_localdb"];
      $azure_mysql_connstr_match = preg_match(
        "/" .
          "Database=(?<database>.+);" .
          "Data Source=(?<datasource>.+);" .
          "User Id=(?<userid>.+);" .
          "Password=(?<password>.+)" .
          "/u",
        $azure_mysql_connstr,
        $_
      );
```

`MYSQLCONNSTR_localdb` と言う文字列は MySQL in App の環境変数として MySQL in App のページにて指定されています。

![2022-04-07-trying-mysql-in-app](/media/20220407/13.png)

## 4. デプロイする
上記の `index.php` を作成した Web App へデプロイして下さい。(ソース管理やエディタの選別、Web App へのデプロイ方法等は開発者の皆様それぞれ好みがあると思いますので、この表現になりました。)

Web App へのデプロイは Deployment Center より、GitHub や Azure Repos 等選択可能です。

![2022-04-07-trying-mysql-in-app](/media/20220407/14.png)

参考までに、私は Azure Repos を用いてデプロイしました。

![2022-04-07-trying-mysql-in-app](/media/20220407/15.png)

それでは Web App へアクセスしてみましょう。下記のようなページとなれば成功です。

![2022-04-07-trying-mysql-in-app](/media/20220407/16.png)

# 参考文献
- Announcing Azure App Service MySQL in-app (preview) - [https://azure.microsoft.com/ja-jp/blog/mysql-in-app-preview-app-service/](https://azure.microsoft.com/ja-jp/blog/mysql-in-app-preview-app-service/)
- Azure App ServiceのMysql In Appが自由でよかったメモ - [https://uzulla.hateblo.jp/entry/2016/09/25/155830](https://uzulla.hateblo.jp/entry/2016/09/25/155830)

# あとがき
本投稿作成時には、https://docs.microsoft.com/ja-jp/ に情報がまとまっているページが存在しなかったので (Toxumuharu 調べ) 、Azure の公式ブログや個人の方々のブログ等を参考に使用してみました。

私自身 Web アプリケーションやデータベースを作成するのは初めてでしたので非常に良い機会でした。データベースを作成したいときは手軽に作成/使用出来るのは良いなと思いました。また何かの機会があれば MySQL in App を使用しようと考えています。
<br>

本ドキュメントが何かの役に立てば幸いです。

Toxumuharu

<br>
<br>

---

<br>
<br>

2022 年 4 月 7 日時点の内容となります。<br>
本記事の内容は予告なく変更される場合がございますので予めご了承ください。

<br>
<br>