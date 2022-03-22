---
title: "Azure App Service の MySQL In App において #2002 - An attempt was made to access a socket in a way forbidden by its access permissions. と出るエラーの対処法"
author_name: "Toxumuharu"
tags:
    - AppService
    - TroubleShoot
    - MySQLInApp
---

# このドキュメントの内容
Azure App Service の MySQL In App の管理画面 https://APPLICATIONNAME.scm.azurewebsites.net/phpMyAdmin/ (APPLICATIONNAME はアプリケーション名) を表示する際、下記のエラーに遭遇したので自身の忘備録として残しておきます。
> #2002 - An attempt was made to access a socket in a way forbidden by its access permissions.  
\- The server is not responding (or the local server's socket is not correctly configured).  
mysqli_real_connect(): (HY000/2002): An attempt was made to access a socket in a way forbidden by its access permissions.

<br>

![2022-03-23-2002-error-of-mysql-in-app-on-appservice](/media/20220319/1.png)

<br>

## 解決策
### 解決策 1
App Service の プランを Premium にスケールアップし、Always On を有効にする。

<br>

![2022-03-23-2002-error-of-mysql-in-app-on-appservice](/media/20220319/2.png)
![2022-03-23-2002-error-of-mysql-in-app-on-appservice](/media/20220319/3.png)

<br>

### 解決策 2
Web アプリケーションへアクセスした後、再度管理画面を開く。

この場合の注意点として、キャッシュによりきちんと表示されていない場合もあるので、同じエラーが出る場合は Command + Shift + R (Windows の場合は Ctrl + Shift + R) でハード読み込みを行う事を推奨します。

## 原因
原因としては、どうやら Web アプリケーションが一定時間リクエストを受信しない場合、App Service のインスタンスが自動的にシャットダウンし、リクエストがあるたびに再起動するために発生するもののようです。

解決策 1 では、インスタンスにシャットダウンさせないためのオプション「Always On」を有効にする事で、常にインスタンスが存在しているためにこのエラーを回避する事が出来るというものです。

解決策 2 では、シャットダウンしたインスタンスを再起動させるべく Web アプリケーションへ予めアクセスしておき管理画面へアクセスを行うというものです。私は富豪ではないのでこちらで様子見をしてみます。

## あとがき
stackoverflow にもしっかり書いてありました。
> It's simple. If you have stopped the website and started it again, you must connect to the website URL from a browser, before you can use phpMyAdmin.  
This only affects app services without Always On enabled.

Title: Access denied error while connecting to MySQL in App in Azure App Service<br>
URL: https://stackoverflow.com/a/57545678/12045751

## 参考文献
- https://stackoverflow.com/q/56028539/12045751

<br>

本ドキュメントが何かの役に立てば幸いです。

Toxumuharu

<br>
<br>

---

<br>
<br>

2022 年 3 月 23 日時点の内容となります。<br>
本記事の内容は予告なく変更される場合がございますので予めご了承ください。

<br>
<br>