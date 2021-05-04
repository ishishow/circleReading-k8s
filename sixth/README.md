# Service のその他の機能

## セッションアフィニティ

- セッションアフィニティーはスティッキーセッションとも言われ、サーバープロセス群の手前に位置し、ルーティング/負荷分散機能を提供するものが、背後のサーバープロセス群のうち、リクエスト元のクライアントのセッションに属する処理を以前していたサーバープロセスに対してルーティングする機能のことです。

- 「サーバー プロセス群の手前に位置するもの」とは、具体的にはロード バランサーや、(サーバー プロセスと Web サーバーが分離している場合は) Apache や IIS などの Web サーバーが考えられます。

- これらは、サーバー プロセスが返すセッション ID Cookie に、どのサーバー プロセスかを示す追加情報を追加したり、独自の Cookie を別途設定したり、セッション ID とサーバー プロセスのマッピング情報を保持したりすることで、適切なサーバー プロセスへのルーティングを実現します。

- セッションに属さないリクエストや、セッション内の初回のリクエストに対しては、セッション アフィニティのルーティングは行われず、ラウンド ロビンなどの指定されたアルゴリズムで、負荷分散を行います。

- Service にはセッションアフィニティを有効化することができます。
- 例えば、ClusterIP Service で有効にした場合、Pod から ClusterIP に向けて送られたトラフィックは Service に紐づくいずれか一つの Pod に対して転送されたあと、その後のトラフィックもずっと同じ Pod に送られるようになります。

- NodePort Service, LB でもセッションアフィニティを有効かすることができますが、どの k8sNode に転送するかはわからないので注意が必要であまり使われません。

## ラウンドロビン

- ラウンドロビンとは、再現のあるリソースを順繰りに割り振ってゆく方式
- DNS のサーバーにおいて用いる DNS ラウンドロビン方式を指す場合が多い。

## ネットワークにおける負荷分散の技法である DNS ラウンドロビン

- インターネット通信においてドメイン名と IP アドレスを相互変換する DNS において、一つのドメイン名に対して複数の IP アドレスをあらかじめ割り当てるというものである。
- クライアントが同じドメイン名を用いてアクセスする度にホストは異なる IP を持つサーバーにアクセス先を順番に割り振ってゆくことでドメイン名の同一性は保持されたままでサーバーコンピューターのアクセス集中によるトラブルを防ぐことができる。

## ノード間通信の排除と送信元 IP アドレスの保持

- NodePort Service と LoadBalancer Service では Kuberneetes Node 上に到達したリクエストはさらにノードを跨いだ Pod へもロードバランシングされるようになっていて二段階ロードバランシングが行われています。

- 二段階ロードバランシングによって均一にリクエストが分散しやすい反面、不要なレイテンシのオーバーヘッドが生じてしまう他、バランシングが行われる際に NAT されるため送信元の IP アドレスが消失するという特徴もあります。

## Headless Service

- Headless Service は対象となる個々の Pod の IP アドレスが直接帰ってくる Service です。今までに紹介した他の Service で負荷分散のために提供されるエンドポイントは仮想 IP によるロードバランシングの動作をするような複数 Pod 宛のエンドポイントでした。

- 一方で Headless Service はロードバランシングするための IP アドレスは提供されず、DNS RoundRobin を使ったエンドポイントを提供します。

## Headless Service の作成

- Headless Service を作成するには、下記の 2 つの条件を満たす必要があります。また、StatefulSet と組み合わせて利用する場合、特定の条件下で Pod 名で名前解決を行うこともできます。

- Service の sspec.type が ClusterIP であること
- Service の spec.clusterIP が None であること
- StatefulSSset で作成された Pod 名でディスカバリする場合、Service の metadata.name が StatefulSet の spec.serviceName と同じであること

## Ingress

- Service が L4 ロードバランシングを提供するリソースであることに対し、Ingress は L7 ロードバランシングを提供するリソースです。そういった意味で Service とは位置付けが大きく異なるリソースであるため、Service の Type の１つとしてではなく独立したリソースとして実装されています。

- Ingress を利用するときは「kind: Service」タイプのリソースではなく「kind: Ingress」タイプのリソースを指定します。

- HTTP や HTTPS の外部アクセスを制御するオブジェクトで、バーチャルホストパスベースのロードバランシングや SSL ターミネーションなどの機能を提供します。

## Why Ingress

- kubernetes は pods, deployments, services の基本要素で構成されており、標準的には、service にロードバランサーを設定して IP を晒し、外部インターネットからアクセスできるようにします。
- しかし、HTTPS にリダイレクトするには内部で nginx-proxy を持つ必要がありますし、service 毎に IP アドレスが生成されてしまうという問題があります。

- ingress は HTTPS レイヤーのロードバランサーであり以下のメリットがあります。
  - IP 管理などを個別の service ではなく ingress で管理できる
  - Google が推奨している
- Ingress を利用しなくても、Service の機能でアプリケーションを公開することはできるが、SSL の設定や、その他メリットも多いので、基本的には利用したほうがよいと思います。

## SSL アクセラレーション

- SSL(/TLS)アクセラレーションとは、LB や firewall などで SSL 通信を暗号化・復号化する処理のこと

## リソースとコントローラ

- Kubernetes は分散システムであり、マニフェストで定義したリソースを Kubernetes に登録することから始まります。登録されただけでは何も処理は行われず、実際に処理を行うコントローラと呼ばれるシステムコンポーネントが必要です。
- 例えば Deployment には対応した ReplicaSet を作成したり、レプリカ数を変更しながらローリングアップデートを行う Deployment Controller と呼ばれるシステムコンポーネントが Kubernetes クラスタ上で動作しています。Deployment Controller がない場合、マニフェストで Deplouyment リソースを作成したとしても ReplicaSet は作られません。
- このように、各リソースは登録された後に様々なコントローラが実際に処理をすることでシステムが動作するようになっています。

## Ingress リソースと Ingress Controller

- Ingress リソースとはマニフェストで登録される API リソースのこと、Ingress Controller は Ingress リソースが Kubernetes に登録された際になんらかの処理を行うものになります。
- 処理の例としては GCP の GCLB を操作することによる L7 ロードバランサの設定や、Nginx の設定を変更してリロードを実施するなどが挙げられます。

## Ingress の種類

- Ingress の実装は複数ありその使い勝手も大きく異なるのもありますが、Ingress は２つに大別できます。
- クラスタ外のロードバランサを利用した Ingress
  - GKE Ingress
- クラスタ内に Ingress 用の Pod をデプロイする Ingress
  - Nginx Ingress

## クラスタ外のリードバランサを使用した Ingress

- GKE のようにクラスタ外のロードバランサを利用した Ingress の場合、Ingress リソースを作成するだけで LoadBalancer の仮想 IP が払い出されて利用可能になります。

- そのため、Ingress のトラフィックは GCP の GCLB がトラフィックを受信した後、GCLB で HTTPS 終端やパスベースルーティングを行い、NodePort へトラフィックを転送することで対象の Pod まで到達します。

## クラスタ内に Ingress 用の Pod を用意する Ingress

- 「クラスタ内に Ingress 用の Pod をデプロイする Ingress」パターンでは、Ingress リソースで定義した L7 相当のロードバランサ処理を行わせるための Ingress 用の Pod をクラスタ内に作成する必要があります。
- また、作成した Ingress 用の Pod に対してクラスタ外からアクセスできるように別途 Ingress 用の Pod が HTTPS の終端やパスベースルーティングなど L7 相当の処理を行うため負荷に応じて Pod のレプリカ数のオートスケーリングも考慮する必要があります。

## Ingress Controller のデプロイ

- Ingress を利用するにはいずれかの Ingress コントローラをデプロイする必要があります。
- GKE Ingress Controller のデプロイ
  - GKE の場合クラスタ作成時に HTTPLoadBalancing アドオンを有効化することでデプロイされます。
  - またこのアドオンはデフォルトで有効化されるようになっています。
- Nginx Ingress Controller のデプロイ
  - Nginx Ingress を利用する場合には Nginx Ingress COntroller をデプロイする必要があります。
  - Nginx Ingress では Ingress Controller 自体が「L7 相当の処理を行う Pod」にもなっており Controller という名前ですが実際の処理も行います。

[Kubernetes NodePort vs LoadBalancer vs Ingress? When should I use what?](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)

# 各 Service のユースケース

## ClusterIP

kubernetes Proxy を利用してあなたのサービスにアクセスするいくつかのシナリオがあります

- サービスをデバッグする、もしくは、直接何らかの理由でラップトップから直接接続を行う。
- 内部トラフィックを許可する、ダッシュボードを閲覧するなど。

しかしこの方法を用いて、kubectl を認証されたユーザで実行する必要があり、サービスをインターネットに公開したり、プロダクションサービスとして利用すべきではありません。

## NodePort

これらの方法には多くの弱点があります :

- 1 サービスあたり 1 つのポートしか利用できない

- 30000–32767 の間のポートだけが利用できます

- ノード/VM の IP を変更した場合、対処する必要がある

それらの理由から、その方法を利用してプロダクションのアプリケーションを公開することを私はおすすめしません。 いつも利用可能である必要がないサービスを実行している場合や、コストを気にする場合、この方法は役にたちます。 そのようなアプリケーションの良い例はデモアプリケーションや一時的なものの場合です。

## LoadBalancer

あなたが直接サービスを公開しようとしたばあい、これが標準の方法です。

指定したすべてのポートに対するトラフィックがサービスにフォワードされます。

フィルタリングされず、ルーティングなどもされません。

HTTP、TCP、UDP、Websocket,gRPC など様々なあらゆる種類のトラフィックを送ることができます。

大きな欠点として、LoadBalancer を利用して公開するそれぞれのサービスが IP アドレスを取得し、LoadBalancer を公開するサービスごとに支払いが発生するため、とても高価になります！

## Ingress

Ingress はあなたのサービスを外部に公開する場合に最もパワフルな方法ですが、最も難解な方法にすることもできます。

Ingress コントローラーの種類は、GCLB、Nginx、Contour、istio などあがります。

また、cert-manager のような Ingress コントローラー用のプラグインもあり、サービスの SSL 証明書を自動的にプロビジョニングできます。

Ingress は同じ IP アドレスで複数のサービスを公開吸う場合に非常に便利であり、これらのサービスはすべて同じ L7 プロトコルを利用します。（通常は HTTP）

ネイティブ GCP 統合を利用している場合には LoadBalancer 一台分を払うだけ、Ingress は "smart" なので、多くの機能をそのまま利用できます。（SSL や Auth，Routing など）

[GKE のベスト プラクティス: Ingress と Service を使用した GKE アプリケーションの公開](https://cloud.google.com/blog/ja/products/containers-kubernetes/exposing-services-on-gke)

[Kubernetes 上のコンテナを Ingress でインターネットに公開するまで
](https://thinkit.co.jp/article/18263)
