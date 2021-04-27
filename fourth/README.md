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
    - Persistent Volume(永続化領域)を使っている場合は　 Pod の再起動時に同じディスクが利用される U

## アップデート戦略

- Deployment や DaemonSet と同様に、StatefulSet のアップデートを行う際にも、アップデート戦略を選択することが可能
- OnDelete の場合には、StatefulSet のマニフェストが変更された際に Pod の更新はせず、別の要因で Pod が再起動されるときに新しい定義で Pod を作成します。
- もう一つの RollingUpdate は Deployment と同じく即時 Pod の更新を行なっていきます。StatefulSet のアップデート戦略も RollingUpdaqte がデフォルトです。

### OnDelete

- StatefulSet では、永続化領域を持つデータベースやクラスタなどに利用されることが多いため、手動でアップデートを行なっていきたい場合には Ondelete を使って任意のタイミングや次回再起動時に更新させられるようになっています。

### RollingUpdate

- Deployment と同じく、RollingUpdate を利用した StatefulSet のアップデートを行うことが可能です。しかし、StatefulSet は永続化データがあるため、Deployment とは異なり、余分な Pod を作成してのローリングアップデートはできません。また、一度に停止可能な Pod 数を指定してのローリングアップデートもできず、1 つの Pod ごとに Ready になったことを確認して行われていきます。spec.podMnagementPolicy が Parallel に設定されている場合でも、並列に処理されることはなく、１つずつ Pod のアップデートを行います。また、StatefulSet の RollingUpdate は partition という特有の値を設定することが可能です。

### StatefulSet の削除と PersistentVolume の削除

- StatefulSet を作成すると、Pod に対して PersistentVolumeClaim を設定可能なため、永続化領域も同時に確保できます。この確保した永続化領域は StatefulSet を削除しても同時に解放されることはないです。マネージドサービスの従量課金制の場合はお金がかかるので注意が必要です、

- [関連リンク]（https://cstoku.dev/posts/2018/k8sdojo-13/）

# Job

## 概要

Job はコンテナを利用して一度限りの処理を実行させるリソースです。より厳密な定義としては、N 並列で実行しながら、指定した回数のコンテナの実行を保証するリソースである。

## ReplicaSet との違いとユースケース

Job と ReplicaSet との大きな違いは「起動する Pod が停止することを前提にして作られているかどうか」。基本的に ReplicaSet などでは、Pod の停止は予期せぬエラーであり、一方 Job の場合は、Pod の停止が正常終了と期待される用途に向いている。例えば、「特定のサーバーとの Rsync」や「S3 などの Object Storage へのアップロード」などが挙げられます。

## Job オブジェクトの必要性

- ReplicaSet, DaemonSet などは、データベースや Web アプリケーション、監視エージェントなど、長時間（基本的にはずっと・アップグレードするか不要になるまで）動き続けるプロセスに焦点を当てたもの。
- 一方で、Job オブジェクトは、バッチ処理など、短時間だけ動かすタスクを扱うもの。
  Job は、処理が正常終了する（終了コード 0 での終了など）まで動く Pod を作成する。

# CronJob

CronJob と Job の関係は Deployment と ReplicaSet の関係に似ています。すなわち、CronJob が Job を管理し、Job が Pod を管理するような 3 そうの親子関係になっています。

## 構造ごとの守備範囲

### CronJob

- Cron フォーマットによる時間の管理
- 重複するジョブの制御など
- Job のステータス管理
- Job が正常に完了したかどうかをモニタリング
- 過去履歴の保持(成功/失敗それぞれ)
- データベースのバックアップを定期的に取る
- npm audit を定期的に実行する
- 放置されている issue/pull request を定期的に Slack に通知する

### Job

- Pod のコンフィグ管理
- Pod のステータス管理
- 失敗時のリトライ有無
- 失敗時のタイムアウト時間
- 並行実行数
- "成功"と判定するために必要な完了数
- Job は必ずしも CronJob と合わせて使う必要はなく、webhook から作成したりなどイベントドリブンな使い方も想定されている

### Pod

- Job が持つ Pod のコンフィグを継承、任意のイメージで任意のジョブを実行する(Deployment のように継続実行ではなく、ジョブを終えると exit することが期待される)

[メルカリの記事](https://engineering.mercari.com/blog/entry/k8s-cronjob-20200908/)

[Kubernetes の CronJob/Job の仕組みをひもとく](https://qiita.com/tmshn/items/aedf0d739a43a1d6423d)

[kubernetes / Helm 大量の CronJob に悩む貴方に送るプラクティス](https://tech.uzabase.com/entry/2019/10/04/200000)
