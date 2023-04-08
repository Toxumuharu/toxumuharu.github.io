---
title: "Xamarin を理解しにいく記録"
author_name: "Toxumuharu"
tags:
    - Xamarin
    - C#
    - CrossPlatform
    - iOS
    - Android
    - Mobile
    - MobileApp
   
---

<!-- <div align="center">
<img src="/media/20220428/0.png" width="75%">
</div> -->

<div align="center">
<img src="https://img.icons8.com/color/96/000000/xamarin.png"/>
</div>

<!-- ![2022-08-07-trying-understanding-about-xamarin](/media/20220807/0.png) -->



# このドキュメントの内容

私 Toxumuharu は元々 iOS や Android のモバイル アプリケーション開発や、顧客がカスタマイズした Android の核となる Linux Kernel の修正などを行うモバイル屋でした。

モバイル開発の経験はあるものの、クロス プラットフォームである Xamarin に関しては触れてこなかったので、自分なりに理解しながらまとめるまでの記録です。

# 先にあとがき

# [Xamarin とは](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin)

> Xamarin は、.NET を使用して、iOS、Android、Windows 向けの最新で高性能なアプリケーションをビルドするためのオープンソースのプラットフォームです。 Xamarin は、基になるプラットフォーム コードと共有コードの通信を管理する抽象化レイヤーです。

- .NET を使用
- オープンソース プラットフォーム
- 基になるプラットフォーム コード (iOS、Android、Windows) と共有コードの<span style="color: red; ">通信を管理する抽象化レイヤー</span>

## [Xamarin の対象者](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin#who-xamarin-is-for)
> - プラットフォーム間でコード、テスト、ビジネス ロジックを共有する。
> - Visual Studio を使用して C# でクロスプラットフォーム アプリケーションを作成する。

## [Xamarin のしくみ](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin#how-xamarin-works)

> この図は、クロスプラットフォーム Xamarin アプリケーションのアーキテクチャ全体を示しています。 Xamarin を使用すると、プラットフォームごとにネイティブ UI を作成し、プラットフォーム間で共有されるビジネス ロジックを C# で記述できます。 ほとんどの場合、アプリケーション コードの 80% は Xamarin を使用して共有できます。
\
\
Xamarin は .NET 上に構築されています。このため、メモリの割り当て、ガベージ コレクション、基になるプラットフォームとの相互運用性などのタスクが自動的に処理されます。

![image](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin-images/xamarin-architecture.png)

- プラットフォームごとにネイティブ UI を作成
- プラットフォーム間で共有されるビジネス ロジックを C# で記述可能
- Xamarin は .NET 上に構築されている。
- メモリの割り当て、ガベージ コレクション、基になるプラットフォームとの相互運用性などのタスクが自動的に処理される

## [Xamarin.Android](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin#xamarinandroid)
> Xamarin.Android アプリケーションでは、アプリケーションの起動時に、C# が 中間言語 (IL) にコンパイルされてから、ネイティブのアセンブリに Just-in-Time (JIT) コンパイルされます。 Xamarin.Android アプリケーションは、Mono 実行環境内で Android Runtime (ART) 仮想マシンとサイド バイ サイドで実行されます。 Xamarin には、Android.* と Java.* 名前空間への .NET バインドが用意されています。 Mono 実行環境では、Managed Callable Wrappers (MCW) 経由でこれらの名前空間を呼び出し、Android Callable Wrappers (ACW) を ART に提供して、両方の環境で相互にコードを呼び出せるようにします。

- Xamarin.Android のアプリケーションは Mono 実行環境内で実行される
- Mono は Android Runtime (ART) とサイド バイ サイドで実行される
- `Android.* ` と `Java.*` へは `Managed Callable Wrappers (MCW)` 経由で呼び出しを行う
- 逆に `Android Callable Wrappers (ACW)` を ART に提供し、Android Runtime でマネージド コードを呼び出す仕組みを実現している

## [Xamarin.iOS](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin#xamarinios)



# 参考文献



<br>

本投稿が何かの役に立てば幸いです。

Toxumuharu

<br>
<br>

---

<br>
<br>

2022 年 7 月 30 日時点の内容となります。<br>
本記事の内容は予告なく変更される場合がございますので予めご了承ください。

<br>
<br>

<a target="_blank" href="https://icons8.com/icon/38641/xamarin">Xamarin icon by Icons8</a>