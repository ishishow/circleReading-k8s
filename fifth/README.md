## 第５回　 ServiceAPIs カテゴリ〜LB まで

## はじめに

- 同一 Pod のコンテナへ通信を行う場合には localhost 当てに通信を行い、別 Pod のコンテナへ通信を行う場合には Pod の IP アドレス宛に通信を行います

- Kubernetes クラスタは内部ネットワークが自動構成されているため、Pod はサービスを利用せずとも Pod 間通信を行うことが可能です。しかし、Service を利用することによって大きなメリットが二つ得られます。

  - Pod 宛トラフィックのロードバランシング
  - サービスディスカバリとクラスタ内 DNS

- Service はいずれも L4 例やでのロードバランシング機能を提供するものであります。Service ににた機能として、Ingress というリソースもよくでてきますが、Ingress は L7 例やでのロードバランシングを提供します。

## Pod 宛トラフィックのロードバランシング

- Service は受信したトラフィックを複数の Pod にロードバランシングする機能を提供します
- 例えば、Deployment を使用して Pod を複数起動することができますが、Pod は起動するごとにそれぞれ異なる IP アドレスが割り当てられるため、ロードバランシングする仕組みを独自に実現しようとすると、各 Pod の IP を毎回調べたり転送先の宛先を設定したりする必要があります。
- これに対して Service を適用すると複数の Pod に対するロードバランシングを自動的に構成できます。また、Service はロードバランシングの接続口となるエンドポイントも提供します。
- エンドポイントには、外部のロードバランサが払い出す仮想 IP アドレス(VirtualIP)やクラスタ内でのみ利用可能な仮想 IP アドレス(ClusterIP)など様々な種類が提供されています。

## Service の種類一覧

- ClusterIP 一番基本となる Service,クラスタ内部で pod にアクセスを振り分ける
- NodePort 　外部から直接アクセスできる Service
- LoadBalancer 　クラウドプロバイダの LB と連携し、より可溶性の高いロードバランシングを実現するサービス
- ExternalName 外部ドメインへアクセスできるさ０尾す
- Headless Pod の IP アドレスを直接返してくれるサービス

## ClusterIP Service

- ClusterIP Service は、Kubernetes の最も基本となる「type: ClusterIP」の Service です
- 「type: ClusterIP」の Service を作成すると、Kubernetes クラスタ内からのみ疎通性がある仮想 IP が割り当てられます
- ClusterIP あての通信は各ノード上で実行しているシステムコンポーネントの kube-proxy が Pod 向けに転送を行います。
- クラスタ外からアクセスされない箇所などで、クラスタ内 LB として利用されることが多いです。

例

```
apiVersion: v1
kind Service
metadata:
  name: sample-clusterip
spec:
  type: ClusterIP
  ports:
  - name: "http-port"
    protocol: "TCP"
    port: 8080
    targetPort: 80
  selector:
    app: sample-app
```

## ClusterIP 仮想 IP の静的な指定

- 例えばアプリケーションからデータベースサーバを指定する場合、基本的に Service で登録されるクラスタ内 DNS レコードを用いてホストの指定を行うことが望ましいです。
- また一方で、IP アドレスで指定しなければならない場合には、ClusterIP を静的に指定することも可能です。
- ClusterIP を自動付与でなく、手動で指定する場合には spec.clusterIP を指定します。

## ExternalIP Service

- ExternalIP Service は特定の Kubernetes Node の IP アドレス Port で受診したトラフィックをコンテナに転送する形で外部疎通性を確立する Service です。
- しかし、特別な事情がなければ、ExterbnalIP Service の代わりに NodePort 　 Service の利用を検討した方が良いです。

## NodePort Service

- NodePort Service は ExternalIP Service に類似した Service です。
- ExternalIP は指定した Kurbernetes Node の IP アドレス:Port で受診したトラフィックをコンテナに転送する形で外部疎通性を確立していました。
- それに対して NodePort はすべての Kubernetes 　 Node の IP アドレス Port で受診したトラフィックをコンテナに転送する形で外部疎通性を確立します。
- 全 Kubernetes Node の IP アドレスで指定した Port を Listen するため、Port のバッティングを起こさないように注意する必要があります。
- NodePort とはクラスタ外からアクセスを行う上で必要なサービスで、特にサービスを外部に公開する際には必要です。

## NodePort の注意点

- NodePort で利用できるポートの範囲は GKE を含む多くの Kubernetes 環境では 30000~32767 となっており、想定外の値を設定しようとするとエラーになる。
- NodePort では最終的にいずれかのノードに割り当てられた IP アドレス宛に通信を行うため、そのノードが単一障害店となりやすいですが、LoadBalancer はクラスター外のロードバランサーを利用するため障害に強いです。

## ClusterIP と NodePort の違い

- ClusterIP はポッドの代表 IP アドレスとして機能しますが、この違いは K8s 外からアクセスできるか否です。
- ClusterIP は各ポッド間の通信で利用するものですが、NodeIP は K8s クラスタ街からもアクセスが可能というわけです。
- よって、NodeIP は ClusterIP の機能は持ち合わせた上でさらに外部に公開用のポートを開いています。
- ちなみにサービスのマニフェストにタイプを指定しない場合は ClusterIP として処理されます。

## LoadBalancer Service

- LoadBalancer Service は、プロダクション環境でクラスタ外からトラフィックを受ける場合に、一番実用的で使い勝手が良いサービスです。
- クラウドプロバイダが提供する LB を利用してクラスタ外部に公開できる特徴があります。

  - アクセスの流れとして、
  - 外部から来たアクセスを外部 LB にて受信
  - 外部 LB と k8s LoadBalancer Service の連携により、外部アクセスが各 WorkerNode の NodePort を宛先としてバランシングされる。
  - NodePort から ClusterIP へ転送され、さらに Pod、コンテナへと転送されていく。

- LoadBalancer Service は Kubernetes クラスタ外のロードバランサに外部疎通性のある仮想 IP を払い出すことが可能です。
- 外部のロードバランサを利用するため、Kubenetes クラスタが構築されているインフラがこの仕組みに対応している必要があります。
- 現状では GCP/AWS/Azure/OpenStack をはじめとしたクラウドプロバイダが、LoadBalancer Service を利用できる環境の主流です。
- 最近では MetalLB なども登場し、クラウドプロバイダ以外での環境での選択肢も増えてきています。
- NodePort や ExternalIP では結局のところいずれかの KubernetesNode に割り当てられた IP アドレス宛に通信を行うため、そのノードが単一障害点(SPoF)になってしまいます。
- しかし、「type: LoadBalancer」では、Kubernetes Node 群とは別に外部のロードバランサを利用するのでノードの生涯に強いという特徴があります。
- イメージとしては NodePort Service を作り、クラスタ外の LoadBalancer から Kubernetes Node 宛にバランシングするような形です。

## LoadBalancer のファイアーウォールの設定

- LoadBalancer Service を作成すると、デフォルトではその Service は全世界に公開されることになります。
- NetworkProxy リソースや loadbalancersourcerange を使ってアクセス制御ができますが、スケーラビリティの低下やレイテンシに影響しやすいため、可能な限り外部のロードバランサが話でのアクセス制御にした方が良い。

## NodePort タイプと LoadBalancer タイプの違い

- NodePort タイプも、LoadBalancer タイプも外部からのアクセスを ClusterIP タイプのサービスに振り分けるという意味では同じである。
- NodePort タイプでは、WorkerNode に割り当てられた IP を宛先として、直接通信を行う。その WorkerNode に障害があった場合、他の WorkerNode が動作していても Service にアクセスできず、単一障害となってしまう。
- 対して LoadBalancer タイプであれば、他の WorkerNode が生きている限り、k8s のノードとは別に存在する外部 LB が生きている方にバランシングしてくれます。したがって全体としての可溶性を落とさずに済みます。
- LoadBalancer タイプの弱みとしては外部 LB を使うので費用がかかることが挙げられます。しかしメリットが大きいのでプロダクション環境では LoadBalancer タイプをきちんと使った方が良い場合が多いです。
