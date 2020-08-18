---
title: "Go Guidelines: Documentation"
keywords: guidelines golang
permalink: golang_documentation.html
folder: golang
sidebar: golang_sidebar
---

{% include draft.html content="Goガイドラインはドラフト版です。" %}

クライアントライブラリーに含めるか、クライアントライブラリーの付属品として含める必要があるドキュメントの成果物がいくつかあります。コード自体に含まれる完全で役立つAPIドキュメント ([GoDoc]) に加えて、優れたREADMEおよびその他のサポートドキュメントが必要です。

* `README.md` - SDKリポジトリー内のライブラリーのディレクトリーのルートにあります; パッケージのインストールとクライアントライブラリーの使用法に関する情報が含まれています。 ([例][README-EXAMPLE])
* `APIリファレンス` - コード内のドキュメント文字列から生成されます; [godoc.org](https://godoc.org) および [docs.microsoft.com](https://docs.microsoft.com) で公開されています。
* `コードスニペット` - ライブラリーで特定したチャンピオンシナリオの単一 (極小) 操作を示す短いコード例; README、ドキュメント文字列、クリックスタートに含まれています。
* `クイックスタート` - READMEコンテンツに似ているが拡張された docs.microsoft.com の記事； 通常は、サービスのコンテンツ開発者が作成します。
* `コンセプチュアル` - クイックスタート、チュートリアル、ハウツーガイドなどの長い形式のドキュメント、および docs.microsoft.com 上のその他のコンテンツ; 通常は、サービスのコンテンツ開発者が作成します。

{% include requirement/MUST id="golang-docs-contentdev" %} ライブラリーのアーキテクチャーボードのレビューにサービスのコンテンツ開発者を含めてください。協力すべきコンテンツ開発者を見つけるには、チームのプログラムマネージャーに確認してください。

{% include requirement/MUST id="golang-docs-contributors-guide" %} [Azure SDK コントリビューター ガイド] に従ってください. (MICROSOFT 内部)

{% include requirement/MUST id="golang-docs-style-guide" %} 公開ドキュメントを作成するときは、Microsoftスタイルガイドに記載されている仕様に準拠してください。これは、READMEなどの長い形式のドキュメントとコード内のドキュメント文字列の両方に適用されます。 (MICROSOFT 内部)

* [Microsoft ライティング スタイル ガイド].
* [Microsoft クラウド スタイル ガイド].

{% include requirement/SHOULD id="golang-docs-to-silence" %} あなたのライブラリーを沈黙の中に文書化しようとするべきです。[GoDoc] でAPIを明確に説明することで、開発者の使用法に関する質問を先取りし、GitHubの問題を最小限に抑えます。発生する可能性のあるサービス制限とエラーに関する情報と、それらのエラーを回避および回復する方法を含めます。

コードを書くときは、*それを文書化して、二度とそれについて聞かないようにします*。クライアントライブラリーについて回答する必要のある質問が少ないほど、サービスの新機能を構築するための時間が長くなります。

### コード例

コード例は、クライアントライブラリーに関連する特定の機能を示す小さな関数です。例を使用すると、開発者はクライアントライブラリーの完全な使用要件をすばやく理解できます。コード例は、機能を説明するために必要以上に複雑であってはなりません。完全なアプリケーションを作成しないでください。例では、関連のない理由で、有用なコードとボイラープレートコードのS/N比が高くなっている必要があります。

{% include requirement/MUST id="golang-include-code-examples" %} パッケージのコード内にコード例を含めてください。例は、ほとんどの開発者がライブラリーを使用して記述する必要があるコードを明確かつ簡潔に示す必要があります。すべての一般的な操作の例を含めます。ライブラリーの新規ユーザーにとって、複雑な操作や難しい操作に注意してください。ライブラリーで特定したチャンピオンシナリオの例を含めます。

{% include requirement/MUST id="golang-code-example-location" %} コード例は、`examples_test.go` ファイルなど、パッケージディレクトリー内に配置してください。

{% include requirement/MUST id="golang-code-example-comments" %} 出力が確定的で単体テストとしての実行に適している場合は、例の最後に `Output:` または `Unordered output:` [コメント](https://golang.org/pkg/testing/#hdr-Examples) を追加してください。

{% include requirement/MUST id="golang-code-example-graft" %} コード例をドキュメントからユーザー独自のアプリケーションに簡単に移植できることを確認してください。

{% include requirement/MUST id="golang-code-example-readability" %} 読みやすさとコードのコンパクトさと効率的な理解のために、コード例を記述してください。

{% include requirement/MUST id="golang-code-example-platforms" %} 例がWindows、macOS、Linuxの開発環境で実行できることを確認してください。

{% include requirement/MUST id="golang-code-example-builds" %} リポジトリーの継続的インテグレーション (CI) を使用してコード例をビルドおよびテストし、機能し続けることを確認してください。

{% include requirement/MUST id="golang-code-example-exports" %} エクスポートされたすべての型、メンバー、関数、メソッドが文書化されていることを確認してください。

{% include requirement/MUSTNOT id="golang-code-example-combination" %} 型またはメンバーのデモに必要でない限り、コード例で複数の操作を組み合わせないでください。たとえば、Cosmos DBのコード例には、アカウントとコンテナーの両方の作成操作は含まれていません。アカウント作成の例とコンテナー作成の例を別々で作成します。

複数の操作には、ユーザーの現在の焦点から外れている可能性のある追加の操作の知識が必要です。開発者はまず、自分が取り組んでいる操作を取り巻くコードを理解する必要があり、コード例をコピーしてプロジェクトに貼り付けることはできません。

{% include refs.md %}
{% include_relative refs.md %}
