---
title: "Power Automate で SharePoint 上のファイルを Azure Blob へアップロードするフローを作成するまでの記録"
author_name: "Toxumuharu"
tags:
    - PowerAutomate
    - PowerPlatform
    - AzureBlobStrage
    - CloudFlow
---

<div align="center">
<img src="/media/20220428/0.png" width="75%">
</div>
<br>

# このドキュメントの内容
Teams のチャットに投稿されたファイルなどが保存されてある SharePoint 上のファイル (https://ORGANIZATIONNAME.sharepoint.com/sites/TEAMNAME/Shared%20Documents/ 以下のファイル) を Azure Blob ストレージへアップロードするフローを作成します。
サブフォルダーが存在する場合にも対応出来るよう複数のフローを作成し実現します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/1.png)

> 前提事項
> - フローを作成するユーザーは、SharePoint のファイルおよびアップロード先として指定する Azure Blob ストレージ のコンテナーへのアクセス権を有していること。
> - Power Automate の子フローを作成するため、ソリューションを作成すること。
> - Azure Blob ストレージ コネクタ (プレミアム コネクタ) を使用すること。Microsoft 365 付帯のライセンスのみでは実現できず、実現のためには別途 Power Apps や Power Automate のスタンドアロン プランを購入するか、試用版ライセンスを有効にする必要があります。
> - Azure Blob ストレージへアップロードする対象となるファイルは 104 MB よりも小さいこと(Power Automate のバッファーの制限より) 。


# フローの作成
## 作成するフローの仕様
- https://ORGANIZATIONNAME.sharepoint.com/sites/TEAMNAME/Shared%20Documents/ で表される SharePoint のドキュメント内にあるファイルを Azure Blob ストレージへアップロードする。
- 複数のフォルダー階層にも対応出来るフロー (今回は例として次のフォルダ階層/ファイル配置を用いて動作確認を行います。)
```
TestTeam
├── General
│   ├── FolderHierarchy1
│   │   ├── FolderHierarchy2
│   │   │   ├── FolderHierarchy3
│   │   │   │   ├── fileInFolderHierarchy3_1.txt
│   │   │   │   ├── fileInFolderHierarchy3_2.txt
│   │   │   │   └── fileInFolderHierarchy3_3.txt
│   │   │   ├── fileInFolderHierarchy2_1.txt
│   │   │   └── fileInFolderHierarchy2_2.txt
│   │   └── fileInFolderHierarchy1_1.txt
│   └── fileInTestGeneral.txt
└── TestChannel
    └── fileInTestChannel.txt
```

## フロー作成の大まかな流れ
1. メインフロー
    1. 変数の初期化 (SharePoint のドキュメント フォルダーの指定、Azure Blob Strage の指定)
    2. 指定した SharePoint のフォルダーの一覧を獲得
    3. 1.2. で獲得した各ファイルまたはフォルダーに対し、ファイルであれば Azure Blob ストレージへアップロード、フォルダーであればサブフロー 1 を呼び出し
2. サブフロー 1
    1. 受け取った変数の初期化
    2. 変数として指定した SharePoint のフォルダーの一覧を獲得
    3. 2.2. で獲得した各ファイルまたはフォルダーに対し、ファイルであれば Azure Blob ストレージへアップロード、フォルダーであればサブフロー 2 を呼び出し
    4. 呼び出し元 (メインフローまたはサブフロー 2) へ応答
3. サブフロー 2
    1. 受け取った変数の初期化
    2. 変数として指定した SharePoint のフォルダーの一覧を獲得
    3. 3.2. で獲得した各ファイルまたはフォルダーに対し、ファイルであれば Azure Blob ストレージへアップロード、フォルダーであればサブフロー 1 を呼び出し
    4. 呼び出し元 (サブフロー 1) へ応答

初見では訳が分かりませんね。訳が分からないのは分かります。私も分かりませんでした。

フォルダー階層にも対応出来るフローとする箇所が、本フローを分かりにくくしています。フローチャートで見てみます。

サブフロー 1 とサブフロー 2 は基本的に同じ動きをし、**フォルダーを検出した際お互いに呼び合う動作**をします。これは、Power Automate の「フローはそのフロー自体を子フローとして呼び出すことができない」と言う制約を回避し、**擬似再起処理を実現するため**です。

これらサブフローを実装することで、階層数分の分岐やループを実装する必要がない点が、本フローを用いるメリットです。

<div align="center">
<img src="/media/20220428/2.png">
</div>

例として利用するフォルダー階層を担当するフロー毎に枠で囲むと次のようになります。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/24.png)

ちなみに、自分自身を子フローとして実装しようとすると下記のエラーが表示され、フローが正常に保存できないようになっています。

> Request to XRM API failed with error: 'Message: Flow client error returned with status code "BadRequest" and details "{"error":{"code":"FlowCannotInvokeItself","message":"フローはそのフロー自体を子フローとして呼び出すことができないため、フローを保存できませんでした。"}}". Code: 0x80060467 InnerError: '.

それでは実際にフローを作成します。<font color="Red">事前準備として、環境にソリューションを作成します。</font>私は `Toxumuharu_UploadFilesFromSharePointToBlob` としました。

## 1. MainFlow (メインフロー)
フロー名を「MainFlow」、トリガーを「Manually trigger a flow」として作成します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/4.png)

### 1.0 下準備
フロー内で別のフローを子フローとして呼び出すので、メインフローと同じように、次の内容で子フロー　1 と子フロー 2 を作成します。名前は「SubFlow1」「SubFlow2」としました。

準備段階なので、「手動でフローを起動する」トリガーと、「Power Apps またはフローに応答する」アクションのみで構成されているものを作成します。

また、MainFlow より変数を受け取る必要があるので、「Add an input」より、次のインプットを指定します。(値は後に MainFlow より受け取るのでなんでも良いですが、何か入れておかないと保存出来ない仕様になっているのでインプットする変数名と同じ内容を入れます。)
- `SiteAddress`:`SiteAddress`
- `CurrentItem`:`CurrentItem`
- `StorageAccountName`:`StorageAccountName`
- `ContainerName`:`ContainerName`

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/9.png)

子フローの作成は以下に詳細がありますのでご参照ください。

Title: 子フローの作成<br>
URL: [https://docs.microsoft.com/ja-jp/power-automate/create-child-flows](https://docs.microsoft.com/ja-jp/power-automate/create-child-flows)

### 1.1 変数の初期化 (SharePoint のドキュメント フォルダーの指定、Azure Blob Strage の指定)

[本セクションのフローの内容]
![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/5.png)

フローの中で扱う変数として、今回は 4 つの変数を用います。

1. `SiteAddress` (SharePoint サイトのアドレス) - `https://tokawatadev.sharepoint.com/sites/Tokawatadev`
    > `SiteAddress` には Teams のチャンネル内に表示される「Files」タブの項目「Open in SharePoint」より得られる URL のうち、`https://ORGANIZATIONNAME.sharepoint.com/sites/TEAMNAME` までを指定します。

    ![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/3.png)
2. `TargetPath` (SharePoint サイト内のファイルパス) - `/Shared Documents`
    > SharePoint 上のフォルダ構造上、SharePoint サイトの `/Shared Documents` で表されるフォルダー パスが Teams の「Files」タブ内とリンクしているため、`/Shared Documents` を `TargetPath` に指定します。。
3. `StorageAccountName` (Azure Blob ストレージのストレージ アカウント名) - `toxumuharudevblob`
    > アップロード先として指定する Azure Blob ストレージのアカウント名を値として指定します。`toxumuharudevblob` としました。

    ![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/6.png)

4. `ContainerName` (Azure Blob ストレージのコンテナー名) - `target-container`
    > アップロード先として指定する Azure Blob ストレージのコンテナー名を値として指定します。`target-container` としました。

    ![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/7.png)

### 1.2. 指定した SharePoint のフォルダーの一覧を獲得
[本セクションのフローの内容]
![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/8.png)

SharePoint コネクターの List Folder アクションを追加し、それぞれに動的なコンテンツより値を以下のように設定します。
- `Site Address`: `SiteAddress`
- `File Identifier`: `TargetPath` 

Title: リスト フォルダー <br>
URL: [https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#リスト-フォルダー](https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#%E3%83%AA%E3%82%B9%E3%83%88-%E3%83%95%E3%82%A9%E3%83%AB%E3%83%80%E3%83%BC)

これにより、https://ORGANIZATIONNAME.sharepoint.com/sites/TEAMNAME/Shared%20Documents/ で表されるフォルダー内のファイルまたはフォルダーの一覧が取得されます。

### 1.3. 1.2. で獲得した各ファイルまたはフォルダーに対し、ファイルであれば Azure Blob ストレージへアップロード、フォルダーであればサブフロー 1 を呼び出し
[本セクションのフローの内容]
![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/10.png)

本セクションでは、セクション 1.2. で獲得した各ファイルまたはフォルダーに対し、ファイルであれば Azure Blob ストレージへアップロード、フォルダーであればサブフロー 1 に変数を渡し呼ぶ部分を作成します。

セクション 1.2 で、https://ORGANIZATIONNAME.sharepoint.com/sites/TEAMNAME/Shared%20Documents/ で表されるサイト内のファイルまたはフォルダーの一覧を獲得しました。

これらの実行結果は JSON 形式で Power Automate に応答が返ってきており、JSON の `body` というキー内に格納されています。`body` 内には、各アイテム (つまりファイルまたはフォルダー) ごとに [BlobMetadata](https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#blobmetadata) が存在し、この `BlobMetadata` 内に、`IsFolder` というパラメーターが存在します。そのため、`body` の各内容に対して繰り返し処理 (Apply to each) を行います。(JSON てなんやねん、という方は無視していただいて問題ありません。)

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/11.png)

上記で述べたパラメータ `IsFolder` が `true` のときはフォルダーの場合の処理を、`false` のときはファイルの場合の処理を行います。そのため `IsFolder` で条件分岐を行います。条件式の `true` は図のように「Expressions」タブより `true` と入力します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/12.png)

アイテムがフォルダーの場合、つまり `IsFolder` が `true` の場合の処理 (If yes の処理) として、サブフローである SubFlow1 を呼びます。また、その際に動的コンテンツより各変数を SubFlow1 に受け渡します。<font color="Red">`CurrentItem` には、現在対象としているフォルダのパス (`ID`) を設定する点に注意します。</font>
- `SiteAddress`:`SiteAddress`
- `CurrentItem`:`ID`
- `StorageAccountName`:`StorageAccountName`
- `ContainerName`:`ContainerName`

Title: ソリューションに親フローを作成する - 子フローの作成<br>
URL: [https://docs.microsoft.com/ja-jp/power-automate/create-child-flows#create-the-parent-flow-in-a-solution](https://docs.microsoft.com/ja-jp/power-automate/create-child-flows#create-the-parent-flow-in-a-solution)

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/13.png)

またアイテムがファイルの場合、つまり `IsFolder` が `false` の場合の処理 (If no の処理) として、ファイル コンテンツを獲得し、Blob を作成します。

ここで、SharePoint (というか ASP.NET アプリケーション？) の特性上 `.aspx` という拡張子を持ったファイルが、上記の中で検出されてしまいますが、<font color="Red">これは SharePoint にユーザーがアップロードしたものではないので、無視する処理を加えます。</font>

`IsFolder` が `false` の場合の処理の中に、再度条件分岐を追加します。条件として、「`Id` が文字列 `.aspx` で終わらない」という内容を追加します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/16.png)

上記の「`Id` が文字列 `.aspx` で終わらない」という条件が true (If yes) の場合に、ファイル コンテンツを獲得し、Blob を作成するアクションを追加します。

ファイル コンテンツを獲得アクションには、それぞれ以下の値を設定します。
- `Site Address`: `SiteAddress`
- `File Identifier`:`ID (動的コンテンツより)`

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/14.png)

Blob を作成アクションには、動的なコンテンツよりそれぞれ以下を設定します。
- `Storage account name`:`StrageAccountName`
- `Folder path`:`ContainerName`
- `Blob name`:`Name`
- `Blob content`:`File Content`

Title: ファイル コンテンツの取得 - SharePoint<br>
URL: [https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#ファイル-コンテンツの取得](https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB-%E3%82%B3%E3%83%B3%E3%83%86%E3%83%B3%E3%83%84%E3%81%AE%E5%8F%96%E5%BE%97)

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/15.png)

これにて MainFlow の作成は終わりです。次のセクションでは、SubFlow1 の編集を開始します。


## 2. SubFlow1 (子フロー 1)

実は SubFlow1 は MainFlow と対して内容は変わりません。異なる点は変数を初期化する点 (既に作成済み) と SubFlow2 を呼ぶ点、呼び出し元のフローへ応答する 3 点です。MainFlow の作成とほぼ同じ操作なので、本セクションは既に説明済みの内容は簡易的な説明のみとします。

### 2.1. 受け取った変数の初期化
セクション 1. にて SubFlow1 を呼び出す際に指定した変数を指定するため、図のように動的コンテンツよりそれぞれ変数の初期化を行います。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/17.png)

### 2.2. 変数として指定した SharePoint のフォルダーの一覧を獲得
セクション 1.2 と同様に以下のように設定します。ここで、`CurrentItem` には MainFlow (または後に作成する SubFlow2) の `body` で表される値 (つまりサブフォルダーのパス) の一つが MainFlow で指定されて渡されてきているいる点に注意します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/18.png)

### 2.3. 2.2. で獲得した各ファイルまたはフォルダーに対し、ファイルであれば Azure Blob ストレージへアップロード、フォルダーであればサブフロー 2 を呼び出し

サブフローの呼び出しを SubFlow2 とする以外は MainFlow と全く同じ処理を作成します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/19.png)

### 2.4. 呼び出し元 (メインフローまたはサブフロー 2) へ応答
セクション 1.0 にて作成しておりますが、呼び出し元のフローへ応答します。アウトプットは指定しません。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/20.png)

## 3. SubFlow2 (子フロー 2)
SubFlow2 は、SubFlow1 を呼び出す以外、SubFlow1 と内容は全く変わりません。SubFlow1 の作成と同じ手順で SubFlow1 を呼ぶ SubFlow2 を作成します。

なお、フローの作成途中に保存ボタンを押下すると次のようなエラーが発生すると思われます。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/21.png)

これは、子フローのコネクタの利用に際し、誰の接続を使用するか設定されていないという内容のエラーです。フローの詳細画面に戻り、「Run only users」の項目を編集し、自身の接続を選択します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/23.png)
![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/22.png)

以上でフローの作成は終了です。

# 実行と結果
実行してみます。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/25.png)

初回は次のようなダイアログが出るので、そのまま「Continue」を押下し、実行します。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/26.png)

成功しました。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/27.png)

Azure Blob ストレージ側も見てみます。無事ファイルがアップロードされていました。

![2022-04-28-create-power-automate-flow-which-uploads-files-on-sharepoint-to-azure-blob](/media/20220428/28.png)






# 参考文献
- ファイル コンテンツの取得 - SharePoint - [https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#ファイル-コンテンツの取得](https://docs.microsoft.com/ja-jp/connectors/sharepointonline/#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB-%E3%82%B3%E3%83%B3%E3%83%86%E3%83%B3%E3%83%84%E3%81%AE%E5%8F%96%E5%BE%97)
- Power Automate Desktop：フォルダー配下のすべての子、孫サブフォルダーやファイルを取得して処理する方法（再帰処理のサンプル） - [https://cravelweb.com/rpa/power-automate-desktop/how-to-get-folder-hierarchy-using-recursive-call-on-power-automate-desktop](https://cravelweb.com/rpa/power-automate-desktop/how-to-get-folder-hierarchy-using-recursive-call-on-power-automate-desktop)

# あとがき
C や C++ などでは簡単に実装できる再起処理ですが、自身を呼び出せないという制約のためフローを 3 つも用意する羽目になってしまいました。どうにか実装できないかと思い、Web を検索していたところ、参考文献に挙げた Power Automate Desktop での記事しか見当たらず、できないのかと思っていました。しかし、記事を読んでみると著者の方もサブフローを駆使して実装されていらっしゃったので、もしやと思い実装してみると出来てしまった、という次第でした。
<br>

本投稿が何かの役に立てば幸いです。

Toxumuharu

<br>
<br>

---

<br>
<br>

2022 年 4 月 28 日時点の内容となります。<br>
本記事の内容は予告なく変更される場合がございますので予めご了承ください。

<br>
<br>