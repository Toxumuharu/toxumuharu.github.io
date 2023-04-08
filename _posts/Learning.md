# Xamarin について

## [Xamarin とは](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin)

> Xamarin は、.NET を使用して、iOS、Android、Windows 向けの最新で高性能なアプリケーションをビルドするためのオープンソースのプラットフォームです。 Xamarin は、基になるプラットフォーム コードと共有コードの通信を管理する抽象化レイヤーです。

- .NET を使用
- オープンソース プラットフォーム
- 基になるプラットフォーム コード (iOS、Android、Windows) と共有コードの<span style="color: red; ">通信を管理する抽象化レイヤー</span>

### [Xamarin の対象者](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin#who-xamarin-is-for)
> - プラットフォーム間でコード、テスト、ビジネス ロジックを共有する。
> - Visual Studio を使用して C# でクロスプラットフォーム アプリケーションを作成する。

### [Xamarin のしくみ](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin#how-xamarin-works)

> この図は、クロスプラットフォーム Xamarin アプリケーションのアーキテクチャ全体を示しています。 Xamarin を使用すると、プラットフォームごとにネイティブ UI を作成し、プラットフォーム間で共有されるビジネス ロジックを C# で記述できます。 ほとんどの場合、アプリケーション コードの 80% は Xamarin を使用して共有できます。
\
\
Xamarin は .NET 上に構築されています。このため、メモリの割り当て、ガベージ コレクション、基になるプラットフォームとの相互運用性などのタスクが自動的に処理されます。

![image](https://docs.microsoft.com/ja-jp/xamarin/get-started/what-is-xamarin-images/xamarin-architecture.png)

- プラットフォームごとにネイティブ UI を作成
- プラットフォーム間で共有されるビジネス ロジックを C# で記述可能
- Xamarin は .NET 上に構築されている。
- メモリの割り当て、ガベージ コレクション、基になるプラットフォームとの相互運用性などのタスクが自動的に処理される

# [Xamarin.Android のアーキテクチャ](https://docs.microsoft.com/ja-jp/xamarin/android/internals/architecture)



# その他
## [Android ランタイム（ART）と Dalvik](https://source.android.com/devices/tech/dalvik?hl=ja)
> Android ランタイム（ART）とは、<span style="color:red">Android 上のアプリや一部のシステム サービスが使用する管理対象ランタイムのこと</span>を指します。ART とその前身である Dalvik は、元々 Android プロジェクト用に作成されました。ART はランタイムとして、Dalvik 実行可能ファイル形式と Dex バイトコード仕様を実行します。

## [JNI に関するヒント](https://developer.android.com/training/articles/perf-jni)
> JNI（Java Native Interface）では、マネージコード（Java または Kotlin プログラミング言語で記述されたコード）からコンパイルするバイトコードが、C / C++ で記述された Android のネイティブ コードと連携するためのインターフェース仕様が定義されています。JNI はベンダーに依存せず、動的な共有ライブラリからコードをロードすることができます。場合によっては、扱いにくいこともありますが、かなり効率的です。

マネージド コード (Java/Kotlin) からネイティブ コード (C/C++) へコンパイルする仕様が定義されている。

## [.NET とは](https://dotnet.microsoft.com/ja-jp/learn/dotnet/what-is-dotnet)
> .NET は、さまざまな種類のアプリケーションを構築するために Microsoft によって作成された<span style="color:red">オープン ソースの開発者プラットフォームです。<span>

>.NET は、さまざまな種類のアプリケーションを構築するための、無料のクロスプラットフォームのオープン ソース開発者用プラットフォームです。
\
\
.NET を使用すると、複数の言語、エディター、ライブラリを使用して、Web、モバイル、デスクトップ、ゲーム、IoT など向けにビルドすることができます。

>.NET アプリは、C#、F#、または Visual Basic で記述できます。

- 無料
- クロスプラットフォーム
- C#、F#、Visual Basic で記述
