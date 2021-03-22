## OCI とは?

Open Container Initiative（OCI）

コンテナのランタイムとイメージ関連のオープンな業界標準を作成するため、

2015 年 6 月に Docker 社などの複数の企業により設立された。

現在は Linux Foundation 傘下のオープンソース団体。

## docker イメージ

[軽量 Docker イメージに安易に Alpine を使うのはやめたほうがいいという話](https://blog.inductor.me/entry/alpine-not-recommended)

Distroless イメージは余分なものが少なく、運用コストが減るためアプリケーションの実行に特化したイメージ

## Dockerfile のベストプラクティス

[公式](https://man.plustar.jp/docker/engine/userguide/eng-image/dockerfile_best-practices.html)

### コンテナはエフェメラル(短命)であるべき

コンテナは、可能な限りエフェメラル（短命；ephemeral）にすべきです。

つまり、停止・破棄可能であり、また再構築して利用可能にするにも最小限の構成・設定さえすればよい状態であるべきということです。

## 不要なパッケージをインストールしない

複雑さ、依存関係、ファイルサイズ、構築時間をそれぞれ減らすためには、余分な、または必須ではない「あった方が良いだろう」程度のパッケージをインストールすべきではありません。

例えば、データベース・イメージであればテキストエディタは不要でしょう。

## .dockerignore ファイルの利用

ほとんどの場合、空のディレクトリに個々の Dockerfile を置くのがベストです。

その場合、そのディレクトリには Dockerfile の構築に必要なファイルだけを追加するようにします。

あるいは .dockerignore ファイルを追加することでファイルやディレクトリを除外でき、構築のパフォーマンスを高められます。

このファイルは .gitignore ファイルのような除外パターンに対応しています。

作成については、 .dockerignore ファイルをご覧ください。

## コンテナごとに１つのプロセスだけ実行

ほとんどの場合、１つのコンテナの中では１つのプロセスだけを実行すべきです。

アプリケーションを複数のコンテナに分離することで、水平スケールやコンテナの再利用が簡単になります。

サービス間に依存関係がある場合は、 コンテナのリンク を使います。

## k8s

![Kubernetes.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F275351%2Fab09b9fd-4a15-711f-863a-9e5e13126944.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&s=ffce2d11ddf60717f8af109ed65c7e4a)

## GKE EKS AKS

https://www.stackrox.com/post/2021/01/eks-vs-gke-vs-aks-jan2021/

https://www.toptal.com/kubernetes/k8s-aws-vs-gcp-vs-azure-aks-eks-gke

### CNCF が当てている３つの焦点

1. Consistency: The ability to interact consistently with any Kubernetes installation.

2. Timely updates: Vendors are required to keep versions updated, at least yearly.

3. Confirmability: Any end-user can verify the conformity using Sonobuoy.

> 1. 一貫性：Kubernetes インストールと一貫して対話する機能。
> 2. タイムリーな更新：ベンダーは、少なくとも年に 1 回、バージョンを更新し続ける必要があります。
> 3. 確認可能性：すべてのエンドユーザーは、Sonobuoy を使用して適合性を確認できます。

GKE の方が初心者は扱いやすいが、内部通信が若干ブラックボックス化している。

EKS は security group 周りなどから用意してあげる必要があり、拡張性が高いがより扱いにくい

AKS は GKE と EKS の間で GKE 寄りの位置に属する

## 基本を理解する

### コンテナオーケストレーションとは何ですか？

コンテナーオーケストレーションは、実行中のコンテナーを中心に展開するすべてのリソース（構成、リソース、スケーリング、監視、ネットワーキング、およびツール）の管理と抽象化です。Kubernetes は、業界で最も広く採用されているコンテナオーケストレーションツールの 1 つです。

### なぜコンテナオーケストレーションが必要なのですか？

サーバー上で実行されているコンテナーのフリートを効率的に管理および編成できるようにするには、コンテナーのオーケストレーションが必要です。コンテナオーケストレーションを使用すると、スケーラブルで復元力があり、強力なコンテナ中心のシステムを構築して、あらゆるアプリケーションをデプロイできます。

### Kubernetes を使用したコンテナオーケストレーションのメリットは何ですか？

Kubernetes でコンテナオーケストレーションを使用する利点は、サーバーの上に抽象化レイヤーを提供してコンテナを実行できることです。Kubernetes を使用すると、構成とリソースを効率的に管理し、必要に応じてインフラストラクチャを簡単に拡張できます。

### Kubernetes とは正確には何ですか？

Kubernetes は、Google プロジェクトである Borg に基づいて開発されたオープンソースツールです。これは、サーバー上に抽象化レイヤーを作成して、コンテナーのスケーリング、監視、リソース使用量、ネットワーキング、および構成を簡単に管理できるようにする、実稼働グレードのコンテナーオーケストレーションツールです。
