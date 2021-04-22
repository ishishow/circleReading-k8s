# 第 4 回　 WorklosdAPI 「DaemonSet〜」

# DaemonSet

- [公式リンク](https://kubernetes.io/ja/docs/concepts/workloads/controllers/daemonset/)

## 公式による説明

> DaemonSet は全て(またはいくつか)の Node が単一の Pod のコピーを稼働させることを保証します。Node がクラスターに追加されるとき、Pod が Node 上に追加されます。Node がクラスターから削除されたとき、それらの Pod はガーベージコレクターにより除去されます。DaemonSet の削除により、DaemonSet が作成した Pod もクリーンアップします。

## DaemonSet の必要性

- ある種のエージェントやデーモンなどを、Kuberenetes クラスタを構成する各ノード上で動作させたいというユースケースがある
- 特定ノードへの割り当てを制限したり、Pod が同じノード上に配置されないようにパラメータを調整することを ReplicaSet でおこなうことは推奨されない。
- このようなユースケースの際に利用できるのが DaemonSet。

## ReplicaSet と DaemonSet

- ReplicaSet は各 k8s Node 上に合計で N 個の Pod を Node のリソース状況に合わせて配置する
  - そのため各 Node 上の Pod の数が等しくなるとは限らないし、全ての Node 常に確実に配置される保証もない
- DaemonSet は各 Node に Pod を一つずつ配置するリソースである

  - そのためレプリカ数は指定できないし、１つの Node に 2Pod ずつ配置するといったこともできない
  - ただし、Pod を配置したくないノードがある場合は nodeSelector や Node Anti-Affinity を使ったスケジューリングで除外できる。
  - Kubernetes Node を増やした際も、DaemonSet の Pod は自動的に増えたノードで起動する。

## ユースケース

DaemonSet のユースケースとして、全 Node 上で必ず動作させたいプロセスのために利用することが有用である。例えば以下が挙げられる。

- 各 Pod が出力するログをホスト単位で収集する Fluentd
- 各 Pod のリソース状況やノードの状態をモニタリングする Datadog

## DaemonSet のアップデート戦略

- Ondelete
- RollingUpdate

## DaemonSet の代替案

- Init スクリプト

  - Node 上で直接起動することにより(例: init、upstartd、systemd を使用する)、デーモンプロセスを稼働することが可能です。この方法は非常に良いですが、このようなプロセスを DaemonSet を介して起動することはいくつかの利点があります。
    - アプリケーションと同じ方法でデーモンの監視とログの管理ができる。
    - デーモンとアプリケーションで同じ設定用の言語とツール(例: Pod テンプレート、kubectl)を使える。
    - リソースリミットを使ったコンテナ内でデーモンを稼働させることにより、デーモンとアプリケーションコンテナの分離を促進します。しかし、これは Pod 内でなく、コンテナ内でデーモンを稼働させることにより可能です(Docker を介して直接起動する)。

- Bare Pods

  - 特定のノードを指定して実行する Pod を直接作成することは可能です。しかし、DaemonSet は、ノードの故障やカーネル・アップグレードなどの破壊的なノード・メンテナンスの場合など、何らかの理由で削除されたり終了したりする Pod を置き換えるものです。このような理由から、個々の Pod を作成するよりも DaemonSet を使用することをお勧めします。

- 静的 Pod

  - Kubelet によって監視されているディレクトリに対してファイルを書き込むことによって、Pod を作成することが可能です。これは静的 Pod と呼ばれます。DaemonSet と違い、静的 Pod は kubectl や他の Kubernetes API クライアントで管理できません。静的 Pod は ApiServer に依存しておらず、クラスターの自立起動時に最適です。また、静的 Pod は将来的には廃止される予定です。

# StatefulSet

- [公式リンク](https://kubernetes.io/ja/docs/concepts/workloads/controllers/statefulset/)

## 公式による説明

> StatefulSet はステートフルなアプリケーションを管理するためのワークロード API です。

    StatefulSetはDeploymentとPodのセットのスケーリングを管理し、それらのPodの順序と一意性を保証 します。
    Deploymentのように、StatefulSetは指定したコンテナのspecに基づいてPodを管理します。Deploymentとは異なり、StatefulSetは各Podにおいて管理が大変な同一性を維持します。これらのPodは同一のspecから作成されますが、それらは交換可能ではなく、リスケジュール処理をまたいで維持される永続的な識別子を持ちます。
    ワークロードに永続性を持たせるためにストレージボリュームを使いたい場合は、解決策の1つとしてStatefulSetが利用できます。StatefulSet内の個々のPodは障害の影響を受けやすいですが、永続化したPodの識別子は既存のボリュームと障害によって置換された新しいPodの紐付けを簡単にします。

## 使用パターン

- StatefulSet は、Compute Engine の永続ディスクなどの永続ストレージにデータを保存するステートフル アプリケーションやクラスタ化されたアプリケーションをデプロイするように設計されています。
- StatefulSet は、Kafka、MySQL、Redis、ZooKeeper などの一意で永続的な ID と固有のホスト名が必要なアプリケーションのデプロイに適しています。ステートレス アプリケーションには、Deployment を使用します。

---

## 本書による説明

- StatefulSet も DaemonSet 同様に ReplicaSet の特殊な形ともいえるリソースで、データベースなどのステートフルなワークロードに対応するために存在する

- ReplicaSet との違いとして、
  - 作成される Pod 名のサフィックスは数字のインデックスが付与されたものになる
    - sample-statefulset-0, sample-statefulset-1, sample-statefulset-2
    - Pod 名が変わらない
  - データを永続化するための仕組みを有している
    - Persistent Volume(永続化領域)を使っている場合は　 Pod の再起動時に同じディスクが利用される

## あとで追加する

- [関連リンク]（https://cstoku.dev/posts/2018/k8sdojo-13/）

# Job

## Job オブジェクトの必要性

- ReplicaSet, DaemonSet などは、データベースや Web アプリケーション、監視エージェントなど、長時間（基本的にはずっと・アップグレードするか不要になるまで）動き続けるプロセスに焦点を当てたもの。
- 一方で、Job オブジェクトは、バッチ処理など、短時間だけ動かすタスクを扱うもの。
  Job は、処理が正常終了する（終了コード 0 での終了など）まで動く Pod を作成する。

# CronJob
