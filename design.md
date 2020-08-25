---
title: "Go Guidelines: API Design"
keywords: guidelines golang
permalink: golang_design.html
folder: golang
sidebar: golang_sidebar
---

{% include draft.html content="Goガイドラインはドラフト版です。" %}

# パッケージ

Goは、パッケージ内の関連するタイプをグループ化します。 Goでは、パッケージの名前を `<prefix><service>` にする必要があります。`<prefix>` は `arm` または `az` のいずれかで、`<service>` は単一の単語で表されるサービス名です。

{% include requirement/MUST id="golang-package-prefix" %} パッケージを `arm` または `az` で開始して、Azureクライアントパッケージを示します。管理プレーンパッケージには `arm` を使用し、他のすべてのパッケージには `az` を使用します。

{% include requirement/MUST id="golang-package-name" %} パッケージ名はすべて小文字で構成してください (大文字、ハイフン、アンダースコアは使用できません) 。 たとえば、Azureコンピュート管理パッケージの名前は `armcompute` で、Azure blobストレージパッケージの名前は `azblob` です。

{% include requirement/MUST id="golang-package-registration" %} 選択したパッケージ名を [アーキテクチャ ボード] に登録してください。 イシューを開いてパッケージ名をリクエストします。 現在登録されているパッケージのリストについては、[登録済みパッケージリスト]](registered_namespaces.html) を参照してください。

## ディレクトリー構造

{% include requirement/MUST id="golang-pkgpath-construction" %} 利用者がパッケージを使用中のサービスに結び付けることができるパッケージインポートパスを構築します。 製品のブランドが変更されても、パッケージパスは変更され **ません** 。 変更される可能性のあるマーケティング名の使用は避けてください。

{% include requirement/MUST id="golang-pkgpath-leaf" %} パッケージリーフディレクトリー名がソースコードで宣言されたパッケージ名と一致することを確認してください。

{% include requirement/MUST id="golang-pkgpath-apiver" %} 各サービスAPIバージョンが独自のディレクトリーにあることを確認してください。そのサービスは複数のAPIバージョンをサポートします。

`arm` パスに管理 (Azureリソースマネージャー) APIを配置してください。パッケージパスには、グループ化の `./sdk/arm/<group>/<api-version>/arm<service>` を使用します。 データプレーンAPIよりも多くのサービスが管理APIを必要とするため、他のパスは管理目的でのみ明示的に使用できます。データプレーンの使用は例外的にのみです。コントロールプレーンSDKに使用できる追加のパスは次のとおりです：

{% include tables/mgmt_namespaces.md %}
| 名前空間グループ | 機能領域 |
| appmodel | 関数やアプリフレームワークなどのアプリケーションモデル |
| compute | 仮想マシン、コンテナ、その他のコンピューティングサービス |
| integration | 統合サービス (Logic Appsなど) |
| management | 管理サービス (コスト分析など) |
| networking | VPN、WAN、ネットワーキングなどのサービス |

多くの管理APIは、Azureアカウントの管理を扱うため、データプレーンを備えていません。 管理パッケージを `arm` パスに配置します。 たとえば、`sdk/arm/management/costanalysis` の代わりに `sdk/arm/costanalysis/...` を使用します。

以下は完全な例です。

データプレーンパッケージ:

- github.com/Azure/azure-sdk-for-go/sdk/keyvault/7.0/azkeyvault
- github.com/Azure/azure-sdk-for-go/sdk/storage/blob/2019-12-19/azblob
- github.com/Azure/azure-sdk-for-go/sdk/storage/queue/2019-12-19/azqueue
- github.com/Azure/azure-sdk-for-go/sdk/storage/table/2019-12-19/aztable

管理プレーンパッケージ:

- github.com/Azure/azure-sdk-for-go/sdk/arm/keyvault/2019-09-01/armkeyvault
- github.com/Azure/azure-sdk-for-go/sdk/arm/storage/2019-01-01/armstorage
- github.com/Azure/azure-sdk-for-go/sdk/arm/storage/2019-02-01/armstorage

## バージョン管理

{% include requirement/MUST id="golang-versioning-modules" %} 各パッケージを[Goモジュール](https://blog.golang.org/using-go-modules)としてリリースしてください。`dep` や `glide` などの従来の依存関係管理ツールはサポートされていません。

{% include requirement/MUST id="golang-versioning-semver" %} [semver 2.0](https://semver.org/spec/v2.0.0.html) に準拠したモジュールのバージョンをリリースしてください。

{% include requirement/MUST id="golang-versioning-preview" %} リリース前のモジュールを明確にバージョン管理します。 新しいモジュールの場合は、サフィックスのないv0メジャーバージョン (v0.1.0) を使用します。 既存のモジュールの場合は、`-preview` サフィックス (v1.1.0-preview, v2.0.0-preview) を使用します。

## 依存関係

パッケージは、次の理由により、標準ライブラリーの外部にあるパッケージへの依存関係を回避するよう努めるべきです:

- **バージョン管理** - 標準ライブラリーの外部で定義された型 (つまり、`交換型`) を公開すると、バージョン管理が複雑になる可能性があります。パッケージFooのv3から型を公開するクライアントパッケージがあり、利用者がパッケージFooのv5を使用したい場合、利用者はv5型を使用してv3要件を満たすことができません。
- **サイズ** - 利用者アプリケーションは、クラウドに可能な限り迅速に展開し、ネットワーク間でさまざまな方法で移動できる必要があります。追加のコード (依存関係など) を削除すると、デプロイのパフォーマンスが向上します。
- **ライセンス** - 依存関係のライセンス制限を意識する必要があり、多くの場合、それらを使用するときに適切な属性と通知を提供します。
- **互換性** - 多くの場合、依存関係を制御せず、元の使用と互換性のない方向に進化することを選択する場合があります。
- **セキュリティ** - 依存関係でセキュリティの脆弱性が発見された場合、Microsoftが依存関係のコードベースを制御していないと、脆弱性を修正することが困難または時間がかかる可能性があります。

{% include requirement/MUST id="golang-dependencies-exch-types" %} 交換型を標準ライブラリーで提供される型に制限します (**例外はありません**) 。

{% include requirement/MUST id="golang-dependencies-azure-core" %} すべてのクライアントパッケージに共通する機能については、`azcore` パッケージに依存してください。このパッケージには、HTTP接続、グローバル構成、ロギング、資格情報処理などのAPIが含まれています。

{% include requirement/MUST id="golang-dependencies-azure-core" %} 公開してはならないすべてのクライアントパッケージに共通する機能については、`sdk/internal` パッケージに依存してください。このパッケージには、スタックフレーム情報などでエラーを作成するためのヘルパーが含まれています。

{% include requirement/MUSTNOT id="golang-dependencies-approved-list" %} 次の場合を除いて、クライアントパッケージ配布パッケージ内の他のパッケージに依存しないでください:

{% include_relative approved_dependencies.md %}
| ライブラリー | バージョン | 使用法 |
| library | バージョン | 何か役立つことをする |

## サービス固有の共通パッケージ

複数のクライアントパッケージ間で共通のコードを共有する必要がある場合があります。 たとえば、協力するクライアントパッケージのセットは、例外またはモデルのセットを共有したい場合があります。

{% include requirement/MUST id="golang-commonlib-approval" %} 一般的なパッケージを実装する前に、[アーキテクチャーボード] の承認を得てください。

{% include requirement/MUST id="golang-commonlib-minimize-code" %} 共通のパッケージ内のコードを最小限に抑えます。共通パッケージ内のコードは、他のクライアントパッケージと同じように扱われます。

{% include requirement/MUST id="golang-commonlib-namespace" %} 共通パッケージは、関連するクライアントパッケージと同じディレクトリーに保存してください。

共通パッケージは、次の場合にのみ承認されます:

* 非共有パッケージの利用者は、共通パッケージ内のオブジェクトを直接消費し、かつ
* 情報は複数のクライアントパッケージ間で共有されます。

2つの例を見てみましょう:

1. 2つのCognitive Servicesクライアントパッケージを実装すると、1つのCognitive Servicesクライアントパッケージによって生成され、別のCoginitive Servicesクライアントパッケージによって使用されるモデルが必要であるか、または、同じモデルが2つのクライアントパッケージによって生成されます。利用者は、コードでモデルを渡す必要があります。または、あるクライアントパッケージで作成されたモデルと別のクライアントパッケージで作成されたモデルを比較する必要がある場合があります。これは、一般的なパッケージを選択するのに適した候補です。

2. 2つのCognitive Servicesクライアントパッケージが `BoundingBox` モデルを返し、画像内でオブジェクトが検出された場所を示します。各クライアントパッケージの `BoundingBox` モデル間にはリンクがなく、別のクライアントパッケージに渡されません。これは、共通パッケージの作成に適した候補ではありません (名前空間にすでに存在する場合は、このモデルを共通パッケージに配置することもできます) 。代わりに、各クライアントパッケージに1つずつ、2つの異なるモデルを作成します。

# サービスクライアント

APIサーフェスは、利用者がサービスに接続するためにインスタンス化する1つ以上のサービスクライアントと、サポート型のセットで構成されます。

{% include requirement/MUST id="golang-client-naming" %} ネームサービスのクライアント型には、`Client` という接尾辞を付けます。

```go
type WidgetClient struct {
	// すべてのフィールドをエクスポートしてはなりません
}
```

{% include requirement/MUST id="golang-api-service-client-immutable" %} すべてのサービスクライアント型が、複数のゴルーチンによる同時使用に対して安全であることを確認してください。理想的には、すべてのクライアントの状態は不変であり、このガイドラインを満たします。

{% include requirement/MUSTNOT id="golang-api-service-client-fields" %} クライアント型のフィールドはエクスポートしないでください。これは、インターフェース型によるクライアントのモックをサポートするためのものです。

## サービスクライアントコンストラクター

{% include requirement/MUST id="golang-client-constructors" %} サービスクライアント型の新しいインスタンスを返す次の形式の2つのコンストラクターを提供してください。

```go
// NewWidgetClientは、指定された値でWidgetClientの新しいインスタンスを作成します。デフォルトのパイプライン構成を使用します。
// endpoint - ウィジェットのURI。
// cred - ウィジェットサービスでの認証に使用される資格。
// options - オプションのWidgetClient値。デフォルト値を受け入れるにはnilを渡します。
func NewWidgetClient(endpoint string, cred azcore.Credential, options *WidgetClientOptions) (*WidgetClient, error) {
	// ...
}

// NewWidgetClientWithPipelineは、指定された値とカスタムパイプラインを使用してWidgetClientの新しいインスタンスを作成します。
// endpoint - ウィジェットのURI。
// p - このWidgetClientのHTTPリクエストとレスポンスを処理するために使用されるパイプライン。
func NewWidgetClientWithPipeline(endpoint string, p azcore.Pipeline) (*WidgetClient, error) {
	// ...
}
```

{% include requirement/MUST id="golang-client-constructors" %} デフォルトのエンドポイントを持つサービスには、次の形式でデフォルトのコンストラクターを提供してください (管理プレーンが最も一般的な例です) 。

```go
// NewDefaultClientは、指定された値でWidgetClientの新しいインスタンスを作成します。デフォルトのエンドポイントとパイプライン構成を使用します。
// cred - ウィジェットサービスでの認証に使用される資格。
// options - オプションのWidgetClient値。デフォルト値を受け入れるにはnilを渡します。
func NewDefaultClient(cred azcore.Credential, options *WidgetClientOptions) (*WidgetClient, error) {

}
```

{% include requirement/MUST id="golang-client-constructors-params" %} すべてのコンストラクターパラメーターをメソッドブロックコメントの一部として文書化します。

## 認証と資格情報

Azureサービスは、さまざまな種類の認証スキームを使用して、クライアントがサービスにアクセスできるようにします。概念的には、このプロセスに責任がある2つのエンティティがあります: 資格情報と認証ポリシーです。資格情報は、機密の認証データを提供します。認証ポリシーは、資格情報によって提供されるデータを使用して、サービスに送信される前にHTTP要求を変更します。

{% include requirement/MUST id="golang-auth-support" %} サービスがサポートするすべての認証技術をサポートしてください。

{% include requirement/MUST id="golang-auth-use-azidentity" %} 可能な場合は、`azcore` または `azidentity` パッケージの資格情報と認証ポリシーの実装を使用してください。

{% include requirement/MUST id="golang-auth-concurrency" %} サービスへのリクエストを認証するために必要なすべてのデータをフェッチするために使用できる資格情報タイプを提供してください。サービス固有の資格情報タイプを使用する場合、実装は複数のゴルーチンによる同時使用に対して安全でなければなりません。

{% include requirement/MUSTNOT id="golang-auth-connection-strings" %} ツール (Azureポータル、コピー/貼り付け操作など) 内でそのような接続文字列が利用可能でない限り、接続文字列を使用したサービスクライアントの構築をサポートしません。接続文字列は、エンドポイント、資格情報データ、およびサービスクライアントの構成を簡略化するために使用されるその他のオプションの組み合わせです。接続文字列は、ポータルからのコピー/貼り付けにより、アプリケーションに簡単に統合できます。ただし、接続文字列内の認証情報は、実行中のプロセス内でローテーションすることはできません。本番環境のアプリでは、それらの使用は推奨されません。クライアントライブラリーが接続文字列をサポートしている場合、コンストラクターは次のようになります:

```go
// NewWidgetClientFromConnectionStringは、指定された値でWidgetClientの新しいインスタンスを作成します。デフォルトのパイプライン構成を使用します。
func NewWidgetClientFromConnectionString(ctx context.Context, con string, options *WidgetClientOptions) (*WidgetClient, error) {
	// ...
}
```

認証を実装する場合、PII (個人を特定できる情報) の漏えいや資格情報の漏えいなどのセキュリティホールを利用者に開放しないでください。資格情報は通常、時間制限付きで発行され、サービス接続が期待どおりに機能し続けるように、定期的に更新する必要があります。クライアントライブラリーが現在のすべてのセキュリティ推奨事項に従っていることを確認し、クライアントライブラリーの独立したセキュリティレビューを検討して、ユーザーに潜在的なセキュリティの問題が発生していないことを確認します。

{% include requirement/MUSTNOT id="golang-auth-persistence" %} セキュリティ資格情報を永続化、キャッシュ、または再利用しないでください。セキュリティ資格情報は、セキュリティの懸念と資格情報の更新状況の両方をカバーするために、有効期間が短いと見なされるべきです。

{% include requirement/MUST id="golang-auth-policy-impl" %} サービスが非標準の認証システム (Azure Coreでサポートされていない認証システム) を実装している場合は、適切な認証ポリシーを提供してください。また、サービスによって提供される代替の認証メカニズムを前提として、リクエストに認証情報を追加できるHTTPパイプラインの認証ポリシーを作成する必要もあります。カスタム認証情報は、`azcore.Credentials` インターフェースを実装する必要があります。

## サービスメソッド

{% include requirement/MUST id="golang-client-crud-verbs" %} CRUD操作には次の用語を使用することをお勧めします:

| バーブ         | パラメーター   | コメント |
| `Set<名詞>`    | キー、アイテム | 新しいアイテムを追加するか、既存のアイテムを更新します。 |
| `Add<名詞>`    | キー、アイテム | 新しいアイテムを追加します。アイテムがすでに存在する場合は失敗します。 |
| `Update<名詞>` | キー、アイテム | 既存のアイテムを更新します。アイテムが存在しない場合は失敗します。 |
| `Delete<名詞>` | キー           | 既存のアイテムを削除します。アイテムが存在しなくても失敗しません。 |
| `Get<名詞>`    | キー           | アイテムが存在しない場合、エラーを返します。 |
| `List<名詞>`   |                | アイテムのリストを返します。アイテムが存在しない場合は空のリストを返します。 |
| `<名詞>Exists` | キー           | アイテムが存在する場合は `true` を返します。 |

{% include requirement/SHOULD id="golang-client-verbs-flexible" %} 柔軟性を維持し、開発者のエクスペリエンスに最適な名前を使用する必要があります。サービスチームのドキュメント、ブログ、プレゼンテーションで使用されている用語と矛盾しないでください。

{% include requirement/MUSTNOT id="golang-api-multimethods" %} 単一のRESTエンドポイントに複数のメソッドを提供しないでください。

## サービスメソッドパラメータ

{% include requirement/MUST id="golang-api-service-client-byref" %} クライアント型のすべてのメソッドが参照によってレシーバーを渡すことを確認してください。

{% include requirement/MUST id="golang-api-context" %} 入出力操作を実行するすべてのメソッドの最初のパラメーターとして `context.Context` オブジェクトを受け入れます。

{% include requirement/MUST id="golang-api-mandatory-params" %} すべての入出力メソッドが必須の `context.Context` オブジェクトの後に必要なすべてのパラメーターを受け入れるようにします。

{% include requirement/MUST id="golang-api-options-struct" %} オプションのパラメーターを使用して、すべてのメソッドの `<MethodNameOptions>` 構造を定義してください。この構造には、必須ではないすべてのパラメーターのフィールドが含まれます。バージョン管理を簡素化するために、時間の経過とともに構造にフィールドを追加できます。名前を明確にするために、接頭辞にはクライアント型名を使用します。

{% include requirement/MUST id="golang-api-options-ptr" %} ユーザーが構造体へのポインターを最後のパラメーターとして渡せるようにします。ユーザーが `nil` を渡すと、メソッドはすべての構造体のフィールドに適切なデフォルト値を想定する必要があります。`nil` とゼロで初期化された `<MethodNameOptions>` 構造は、意味的に同等である必要は **ない** ことに注意してください。

{% include requirement/MUST id="golang-api-params" %} メソッドブロックコメントの一部としてすべてのパラメーターを文書化します。

```go
// GetWidgetは、指定されたウィジェットを取得します。
// ctx - リクエストの有効期間を制御するために使用されるコンテキスト。
// name - 取得するウィジェットの名前。
func (c *WidgetClient) GetWidget(ctx context.Context, name string) (*WidgetResponse, error) {
	// ...
}
```

### パラメータの検証

サービスクライアントには、サービスでリクエストを実行するいくつかのメソッドがあります。_サービスパラメーター_ は、ネットワークを介してAzureサービスに直接渡されます。_クライアントパラメーター_ はサービスに直接渡されませんが、要求を満たすためにクライアントライブラリー内で使用されます。クライアントパラメーターの例には、URIの構築に使用される値や、ストレージにアップロードする必要のあるファイルが含まれます。

{% include requirement/MUST id="golang-params-client-validation" %} クライアントパラメーターを検証します。

{% include requirement/MUSTNOT id="golang-params-service-validation" %} サービスパラメーターを検証しません。これには、nullチェック、空の文字列、およびその他の一般的な検証条件が含まれます。サービスにリクエストパラメーターを検証させます。

{% include requirement/MUST id="golang-params-devex" %} サービスパラメーターが無効な場合は、開発者エクスペリエンスを検証して、適切なエラーメッセージがサービスによって生成されるようにします。サービス側のエラーメッセージが原因で開発者エクスペリエンスが損なわれている場合は、サービスチームと協力して、リリース前に修正してください。

## サービスメソッドの戻り値の型

サービスへのリクエストは2つの基本グループに分類されます: 1つは論理的なリクエストを行うメソッド、もう1つは確定的なリクエストのシーケンスを行うメソッドです。_単一の論理リクエスト_ の例は、オペレーション内で再試行されるリクエストです。_リクエストの確定的なシーケンス_ の例は、ページ操作です。

_レスポンスエンベロープ_ は、プロトコルニュートラルなレスポンスの表現です。レスポンスエンベロープは、ヘッダー、本文、およびHTTPレスポンスからのデータを組み合わせる場合があります。たとえば、`ETag` ヘッダーをレスポンスエンベロープのプロパティとして公開できます。`<Resource>Response` は'レスポンスエンベロープ'です。これには、HTTPヘッダー、オブジェクト (レスポンス本文から作成された逆シリアル化されたオブジェクト) 、および未加工のHTTPレスポンスが含まれます。

{% include requirement/MUST id="golang-response-logical-entity" %} 通常のサービスメソッドのレスポンスエンベロープを返します。レスポンスエンベロープは、99％以上の場合に必要な情報を表す必要があります。

```go
// WidgetResponseは、ウィジェット型を返す操作のレスポンスエンベロープです。
type WidgetResponse struct {
	// ETagには、ETagヘッダーの値が含まれています。
	ETag *string

	// LastModifiedには、最終変更ヘッダーの値が含まれています。
	LastModified *time.Time

	// ウィジェットには、非マーシャリングされたレスポンス本文がウィジェット形式で含まれています。
	Widget *Widget

	// RawResponseには、基になるHTTPレスポンスが含まれています。
	RawResponse *http.Response
}

type Widget struct {
	Name string
	Color WidgetColor
}

func (c *WidgetClient) GetWidget(ctx context.Context, name string) (*WidgetResponse, error) {
	// ...
}
```

{% include requirement/MUST id="golang-response-examples" %} クライアントライブラリーによって公開されている、リクエストのストリーミングされたレスポンスにアクセスする方法の例を提供してください。すべてのメソッドがストリーミングされたレスポンスを公開するとは限りません。

```go
func (c *WidgetClient) GetBinaryResponse(ctx context.Context, name string) (*http.Response, error) {
	// ...
}

// 呼び出し元は、HTTPレスポンスのio.ReadCloser Bodyフィールドから読み取ります。
```

{% include requirement/MUST id="golang-response-logical-paging" %} ページ操作のすべての論理エンティティを列挙する慣用的な方法を提供し、必要に応じて新しいページを自動的にフェッチします。リスト操作で何を返すかについての詳細は、[ページネーション](#pagination)を参照してください。

複数のリクエストを1つの呼び出しに結合するメソッドの場合:

{% include requirement/MUSTNOT id="golang-response-no-headers" %} メソッドの戻り値がどの特定のHTTPリクエストに対応するかが明らかでない限り、ヘッダーやその他のリクエストごとのメタデータを返さないでください。

{% include requirement/MUST id="golang-response-failure-info" %} アプリケーションが適切な修正アクションを実行できるように、障害が発生した場合に十分な情報を提供してください。

モデル構造は、利用者がクライアントライブラリーメソッドに必要な情報を提供するために使用する型です。クライアントメソッドから返すこともできます。これらの構造は通常、ドメインモデル、またはリクエストを行う前に構成する必要があるオプション構造を表します。

{% include requirement/MUST id="golang-model-types" %} モックを可能にするために、モデル型のすべてのフィールドをエクスポートします。

{% include requirement/MUST id="golang-model-types-ro" %} ネットワーク経由で送信される構造をマーシャリングするときは、すべての読み取り専用フィールドを文書化し、それらの値を除外してください。

## ページネーションメソッド

{% include requirement/MUST id="golang-pagination" %} ページを返す操作のPagerインターフェースを実装する値を返します。Pagerインターフェースを使用すると、利用者はサービスで定義されているすべてのページを反復できます。

{% include requirement/MUST id="golang-pagination-pagers" %} それぞれの操作から返される `<Resource>Pager` という名前のPagerインターフェース型を作成してください。

{% include requirement/MUST id="golang-pagination-pagers-interface-page" %} `<Resource>Pager` 型でメソッド `NextPage()` 、`Page()` 、および `Err()` を公開します。

```go
type WidgetPager interface {
	// NextPageは、ページャーが次のページに進んだ場合はtrueを返します。
	// ページがなくなった場合、またはエラーが発生した場合はfalseを返します。
	NextPage(context.Context) bool

	// Pageは、現在のWidgetsPageを返します。
	PageResponse() *ListWidgetsResponse

	// Errは、ページング中に発生した最後のエラーを返します。
	Err() error
}

type ListWidgetsResponse struct {
	RawResponse *http.Response
	Widgets *[]Widget
}
```

{% include requirement/MUST id="golang-pagination-methods" %} Pagerを返すメソッドのメソッド名には、接頭辞 `List` を使用してください。`List` メソッドはページャーを作成しますが、入出力操作を実行しません。

```go
func (c *WidgetClient) ListWidgets(options *ListWidgetOptions) *WidgetPager {
	// ...
}

pager := client.ListWidgets(options)
for pager.NextPage(ctx) { 
	for _, w := range pager.PageResponse().Widgets {
		process(w)
	}
}
if pager.Err() != nil {
	// エラー処理...
}
```

{% include requirement/MUST id="golang-pagination-serialization" %} ページャーが一時停止して続行できるように、ページャーをシリアライズおよびデシリアライズする手段を提供してください。

## 長時間実行オペレーション

{% include requirement/MUST id="golang-lro-poller" %} 長時間実行オペレーションメソッドのPollerインターフェースを実装する値を返します。Pollerインターフェースは、長時間実行オペレーションのポーリングとステータスをカプセル化します。

{% include requirement/MUST id="golang-lro-poller-name" %} それぞれの操作から返される `<Resource>Poller` という名前のPollerインターフェース型を作成してください。

{% include requirement/MUST id="golang-lro-poller-def" %} `<Resource>Poller` 型で次のメソッドを提供してください: `Done()` 、`ResumeToken()` 、`Poll()` 、および `FinalResponse()` 。

```go
type WidgetPoller interface {
	// Doneは、LROが完了するとtrueを返します。
	Done() bool

    // ResumeTokenは、LROの再開に使用できるpollerを表す値を返します。
    // ResumeTokensは、操作に固有です。
	ResumeToken() string

    // Pollは、LROの最新の状態をフェッチします。
    // Pollが失敗した場合、WidgetPollerは変更されず、エラーが返されます。
    // Pollが成功し、操作が失敗して完了した場合、WidgetPollerが更新され、エラーが返されます。
    // Pollが成功し、操作が正常に完了した場合、WidgetPollerが更新され、ウィジェットが返されます。
    // Pollが成功し、操作が完了していない場合、WidgetPollerが更新され、最新のHTTP応答を返します。    
	Poll(context.Context) (*http.Response, error)

    // FinalResponseは、サービスに対して最後のGETを実行し、ポーリング操作の最終応答を返します。
    // 最後のGETの実行中にエラーが発生した場合、エラーが返されます。
    // 最後のGETが成功した場合、最後のWidgetResponseが返されます。
	FinalResponse(context.Context) (*WidgetResponse, error)
}
```

{% include requirement/MUST id="golang-lro-wait-method" %} サービスからの関連する再試行後ヘッダーがない場合に使用される `PollUntilDone()` メソッドの `pollingInterval` 引数を受け入れます。

{% include requirement/MUST id="golang-lro-method-naming" %} `Begin` で `<Resource>Poller` を返す接頭辞メソッドを実行します。

```go
// WidgetPollerResponseは、ウィジェット型を非同期で返す操作のレスポンスエンベロープです。
type WidgetPollerResponse struct {
    // PollUntilDoneは、最終状態に到達するか、エラーが受信されるまで、サービスエンドポイントをポーリングします。
	PollUntilDone func(context.Context, time.Duration) (*WidgetResponse, error)

	// Pollerには、初期化されたWidgetPollerが含まれています。
	Poller WidgetPoller

	// RawResponseには、基になるHTTPレスポンスが含まれています。
	RawResponse *http.Response
}

// BeginCreateは、指定された名前で新しいウィジェットを作成します。
func (c *WidgetClient) BeginCreate(ctx context.Context, name string, options *BeginCreateOptions) (*WidgetPollerResponse, error) {
	// ...
}
```

{% include requirement/MUST id="golang-lro-resuming-operations" %} 接頭辞 `Resume` を含むメソッドを提供して、前回の `Poller.ResumeToken()` の呼び出しからの `ResumeToken` で `<Resource>Poller` 型をインスタンス化します。

```go
// ResumeWidgetPollerは、指定されたResumtTokenから新しいWidgetPollerを作成します。
// resumeToken - 値は、WidgetPoller.ResumeToken()への以前の呼び出しから取得する必要があります。
func (c *WidgetClient) ResumeWidgetPoller(resumeToken string) WidgetPoller {
	// ...
}
```

{% include requirement/MUSTNOT id="golang-lro-cancel" %} コンテキストを介してキャンセルが要求されている場合は、LROをキャンセルしないでください。コンテキストはポーリング操作をキャンセルしており、サービスに影響を与えてはなりません。

{% include requirement/MUST id="golang-lro-pattern" %} すべてのLROの操作パターンに従ってください。

```go
// 例1、PollUntilDone()の呼び出しをブロックする
resp, err := client.BeginCreate(context.Background(), "blue_widget", nil)
if err != nil {
	// エラー処理 ...
}
w, err = resp.PollUntilDone(context.Background(), 5*time.Second)
if err != nil {
	// エラー処理 ...
}
process(w)

// 例2、カスタマイズされたポーリングループ
resp, err := client.BeginCreate(context.Background(), "green_widget")
if err != nil {
	// エラー処理 ...
}
poller := resp.Poller
for {
	resp, err := poller.Poll(context.Background())
	if err != nil {
		// エラー処理 ...
	}
	if poller.Done() {
		break
	}
	if delay := azcore.RetryAfter(resp); delay > 0 {
		time.Sleep(delay)
	} else {
		time.Sleep(frequency)
	}
}
w, err := poller.FinalResponse(ctx)
if err != nil {
	// エラー処理 ...
}
process(w)

// 例3、前のポーラーインスタンスから再開トークンを取得する前の操作から再開する
poller := resp.Poller
tk, err := poller.ResumeToken()
if err != nil {
	// エラー処理 ...
}
// 以前に保存された再開トークンから再開する
poller, err := client.ResumeWidgetPoller(tk)
if err != nil {
	// エラー処理 ...
}
for {
	resp, err := poller.Poll(context.Background())
	if err != nil {
		// エラー処理 ... 
	}
	if poller.Done() {
		break
	}
	if delay := azcore.RetryAfter(resp); delay > 0 {
		time.Sleep(delay)
	} else {
		time.Sleep(frequency)
	}
}
w, err := poller.FinalResponse(ctx)
if err != nil {
	// エラー処理 ...
}
process(w)
```

## モック化

私たちがサポートしたい重要なことの1つは、パッケージの利用者が、サービスをアクティブ化することなく、アプリケーションの繰り返し可能な単体テストを簡単に作成できるようにすることです。これにより、基盤となるサービス実装 (ネットワークの状態やサービスの停止など) の変化を心配することなく、コードを確実かつ迅速にテストできます。モックは、障害、エッジケース、再現困難な状況のシミュレーションにも役立ちます (例: 2月29日にコードは機能します) 。

{% include requirement/MUST id="golang-mock-interface-types" %} クライアント型のエクスポートされたすべてのメソッドを含むオペレーショングループごとに1つのインターフェース型を生成します。インターフェース型名は `<op_group_name>Operations` になります。

{% include requirement/MUST id="golang-mock-lro-pages" %} LROのインターフェース型と、それぞれの型のすべてのメソッドを含むページング可能なレスポンス型を生成してください。インターフェース型名は、LRO/ページング可能なレスポンス型名と同じになります。

{% include requirement/MUST id="golang-mock-interface-check" %} インターフェース定義とそれぞれの型が同じメソッド宣言を持っていることを確認するコードを生成してください。これは通常、インターフェース型の変数に型へのnilポインターを割り当てることによって実行されます。

```go
// WidgetOperationsには、ウィジェットグループのメソッドが含まれています。
type WidgetOperations interface {
	BeginCreate(ctx context.Context, options *BeginCreateOptions) (*WidgetPollerResponse, error)
	GetWidget(ctx context.Context, name string) (*WidgetResponse, error)
	ListWidgets(options *ListWidgetsOptions) (ListWidgetsPager, error)
	// other methods...
}

// クライアントとインターフェースの定義が同期していることを確認する
var _ WidgetOperations = (*WidgetClient)(nil)
```

{% include requirement/MUST id="golang-test-recordings" %} パイプライン経由のHTTPリクエストとレスポンスの記録/再生をサポートします。

## 列挙型

{% include requirement/MUST id="golang-enum-type" %} 列挙型を定義して、ネットワーク経由で送受信される型と一致させます (文字列が最も一般的な例です) 。

{% include requirement/MUST id="golang-enum-value-naming" %} すべての値には、型の名前の接頭辞を付けてください。

{% include requirement/MUST id="golang-enum-value-grouping" %} 列挙型のすべての値を独自のconstブロック内に配置します。これは、型の宣言の直後に続きます。

{% include requirement/MUST id="golang-enum-type-values" %} 列挙のすべての可能な値を含むスライスを返す `<EnumTypeName>Values()` という名前の関数を定義してください。

{% include requirement/MUST id="golang-enum-type-values" %} 列挙型へのポインターを返す列挙型に `ToPtr()` という名前のメソッドを定義してください。

```go
// WidgetColorは、可能な値のリストからウィジェットの色を指定します。
type WidgetColor string

const (
	WidgetColorBlue  WidgetColor = "blue"
	WidgetColorGreen WidgetColor = "green"
	WidgetColorRed   WidgetColor = "red"
)

// WidgetColorValuesは、WidgetColorの可能な値のスライスを返します。
func WidgetColorValues() []WidgetColor {
	// ...
}

func (c WidgetColor) ToPtr() *WidgetColor {
	return &c
}

```

## サービスクライアントの構成

{% include requirement/MUST id="golang-config-global" %} デフォルトで、またはユーザーが明示的に要求したときに、たとえば、構成オブジェクトをクライアントコンストラクターに渡すことによって、関連するグローバル構成設定を使用してください。

{% include requirement/MUST id="golang-config-client" %} 同じ型の異なるクライアントが異なる構成を使用できるようにしてください。

{% include requirement/MUST id="golang-config-optout" %} サービスクライアントの利用者がすべてのグローバル構成設定を一度にオプトアウトできるようにしてください。

{% include requirement/MUST id="golang-config-global-override" %} すべてのグローバル構成設定をクライアント提供のオプションで上書きできるようにしてください。これらのオプションの名前は、ユーザー向けのグローバル構成キーと一致する必要があります。

{% include requirement/MUSTNOT id="golang-config-behavior-changes" %} クライアントの構築後に発生する構成変更に基づいて動作を変更しないでください。クライアントの階層は、明示的に変更またはオーバーライドされない限り、親クライアント構成を継承します。この要件の例外は次のとおりです。

1. ログレベル。Azure SDK全体ですぐに有効になる必要があります。
2. トレースのオン/オフ。Azure SDK全体ですぐに有効になる必要があります。

# エラー処理と診断

{% include requirement/MUST id="golang-errors" %} メソッドが目的の機能を実行できない場合は、エラーを返します。複数のアイテムを返すメソッドの場合、エラーオブジェクトは常に戻り署名の最後のアイテムです。

{% include requirement/SHOULD id="golang-errors-wrapping" %} 根本的な障害の診断に役立つ場合は、エラーを別のエラーでラップする必要があります。利用者は、`errors.As()` や `errors.Is()` などの [エラーヘルパー関数](https://blog.golang.org/go1.13-errors) を使用することを期待します。

```go
err := xml.Unmarshal(resp.Payload, v)
if err != nil {
	return fmt.Errorf("unmarshalling type %s: %w", reflect.TypeOf(v).Elem().Name(), err)
}
```

{% include requirement/MUST id="golang-errors-on-request-failed" %} HTTPリクエストが失敗し、サービスで定義されているHTTPステータスコードが失敗した場合、サービス/操作固有のエラー型を返します。エラーの種類を定義しない操作の場合、HTTPレスポンスの本文が利用可能な場合は文字列形式で返します。それ以外の場合は、HTTPレスポンスでステータス文字列を返します。

{% include requirement/MUST id="golang-errors-include-response" %} 返されるエラーには、HTTPレスポンスと元のリクエストを含めてください。この情報を提供するには、`sdk/internal` モジュールの `runtime.NewResponseError()` を使用します。

複数のHTTPリクエストを行うメソッドの場合、最初に発生したエラーによって残りの操作が停止し、このエラー (またはそれをラップする別のエラー) が返されます。

{% include requirement/MUST id="golang-errors-distinct-types" %} 利用者がクライアントエラー  (不完全/不正なAPIパラメーター値) と他のSDKエラー (リクエストの送信の失敗、マーシャリング/非マーシャリング、解析エラー) を区別できるように、明確なエラー型を返します。

{% include requirement/MUST id="golang-errors-documentation" %} 各メソッドによって返されるエラーの種類を文書化してください。HTTPリクエストがタイムアウトしたときに、`context.DeadlineExceeded` など、よく返されるエラーの種類を文書化しないでください。

{% include requirement/MUSTNOT id="golang-errors-other-types" %} 任意のエラー型を作成しないでください。サービス、標準ライブラリー、または `azcore` によって提供されるエラー型を使用します。

## ロギング

クライアントライブラリーは、利用者がメソッド呼び出しの問題を適切に診断し、問題が利用者コード、クライアントライブラリーコード、またはサービスのいずれにあるかをすばやく判断できるように、堅牢なロギングメカニズムをサポートする必要があります。

{% include requirement/MUST id="golang-log-api" %} `azcore` 内で提供されるLogger APIを、すべてのクライアントライブラリー全体で唯一のロギングAPIとして使用してください。

{% include requirement/MUST id="golang-log-classification" %} `azcore.LogClassification` 型を使用して定数分類文字列を定義し、これらの値を使用してログを記録します。

{% include requirement/MUST id="golang-log-inclue" %} HTTPリクエスト行、レスポンス行、およびすべてのヘッダー/クエリーパラメーター名をログに記録します。

{% include requirement/MUSTNOT id="golang-log-exclude" %} 許可リストに含まれていないペイロードまたはHTTPヘッダー/クエリーパラメーター値をログに記録しないでください。許可リストにないヘッダー/クエリーパラメーターの場合、実際の値の代わりに値 `<REDACTED>` を使用します。

## 分散トレース

{% include requirement/MUST id="golang-tracing-abstraction" %} 基盤となるトレース機能を抽象化し、利用者が選択したトレース実装を使用できるようにします。

{% include requirement/MUST id="golang-tracing-span-per-call" %} API呼び出しごとに新しいトレーススパンを作成してください。新しいスパンは、渡されたコンテキストの子である必要があります。

{% include requirement/MUST id="golang-tracing-span-name" %} スパンの名前として、`<package name>.<type name>.<method name>` を使用してください。

{% include requirement/MUST id="golang-tracing-propagate" %} [Azure Monitor](https://azure.microsoft.com/en-us/services/monitor/) や [ZipKin](https://zipkin.io/) などのトレースサービスをサポートするために、適切なヘッダーを介して各送信サービスリクエストのトレースコンテキストを伝達します。これは通常、HTTPパイプラインで行われます。

# HTTPパイプラインとポリシー

サポートされている各言語には、構成やHTTPリクエストの実行などの懸念を横断するための一般的なメカニズムを含むAzure Coreライブラリーがあります。

{% include requirement/MUST id="golang-network-use-http-pipeline" %} サービスRESTエンドポイントとの通信には、`azcore` ライブラリー内のHTTPパイプラインコンポーネントを使用してください。

HTTPパイプラインは、複数のポリシーでラップされたHTTPトランスポートで構成されています。各ポリシーは、パイプラインがリクエストまたはレスポンス、あるいはその両方を変更できる制御ポイントです。クライアントライブラリーがAzureサービスとやり取りする方法を標準化するポリシーの既定のセットを規定しています。リストの順序は、実装の最も適切な順序です。

{% include requirement/MUST id="golang-network-policies" %} HTTPパイプラインに次のポリシーを実装してください:

- テレメトリー
- ユニークリクエストID
- リトライ
- 認証
- レスポンスダウンローダー
- 分散トレース
- ロギング
- HTTPトランスポート自体

{% include requirement/SHOULD id="golang-network-azure-core-policies" %} 可能な限り、Azure Coreのポリシー実装を使用する必要があります。それはあなたのサービスに固有の何かをしていない限り、”独自の"ポリシーを作成しようとしないでください。既存のポリシーに別のオプションが必要な場合は、[アーキテクチャーボード] に連絡してオプションを追加してください。

# 環境変数による設定

{% include requirement/MUST id="golang-envvars-prefix" %} Azure固有の環境変数の接頭辞に `AZURE_` を付けます。

{% include requirement/MAY id="golang-envvars-client-prefix" %} クライアントライブラリーへのパラメーターとして提供されるポータル構成の設定には、クライアントライブラリー固有の環境変数を使用できます。これには通常、資格情報と接続の詳細が含まれます。たとえば、Service Busは次の環境変数をサポートできます:

* `AZURE_SERVICEBUS_CONNECTION_STRING`
* `AZURE_SERVICEBUS_NAMESPACE`
* `AZURE_SERVICEBUS_ISSUER`
* `AZURE_SERVICEBUS_ACCESS_KEY`

ストレージは以下をサポートできます:

* `AZURE_STORAGE_ACCOUNT`
* `AZURE_STORAGE_ACCESS_KEY`
* `AZURE_STORAGE_DNS_SUFFIX`
* `AZURE_STORAGE_CONNECTION_STRING`

{% include requirement/MUST id="golang-envvars-approval" %} すべての新しい環境変数について、[アーキテクチャーボード] から承認を得てください。

{% include requirement/MUST id="golang-envvars-syntax" %} 特定のAzureサービスに固有の環境変数には、次の構文を使用してください。

* `AZURE_<サービス名>_<構成キー>`

ここで、_サービス名_ はスペースを含まない正規の短縮名であり、_構成キー_ はそのクライアントライブラリーのネストされていない構成キーを指します。

{% include requirement/MUSTNOT id="golang-envvars-posix-compliance" %} アンダースコアを除いて、環境変数名に英数字以外の文字を使用しないでください。これにより、幅広い相互運用性が保証されます。

{% include refs.md %}
{% include_relative refs.md %}