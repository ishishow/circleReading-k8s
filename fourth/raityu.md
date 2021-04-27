# TeamCNCF-beginner 4/26

[TOC]

## DaemonSet

- Replicaset の特殊な形とも言えるリソース

> Replicaset は、各 k8sNode 上に合計で N 個の Pod を Node のリソース状況に合わせて配置
> → そのため各 Node 上の Pod の数が等しくなるとも限らない
> DaemonSet は各 Node に Pod を 1 つづつ配置
> → そのためレプリカ数を指定できない
> 　 → ただし Pod を配置したくない Node がある場合除外することも可能

```yaml=
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: sample-ds
spec:
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.16
```

### アップデート戦略

- OnDelete
  - DeamonSet のマニフェストが更新されえた際に Pod 更新はせず別の要因で Pod が再作成されたときに新しい定義で Pod を作成
- RollingUpdate
  - Deployment と同様即時更新
  - デフォルト
  - Deployment と違い 1Node に複数 Pod 作成できないため maxSurge(超過可能数)を設定できない
    - MaxUnvaliable(一度に停止可能数)のみ設定する
    - デフォルトは 1 で MaxUnvaliable を 0 にすることはできない

```yaml=
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: sample-ds-rollingupdate
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2 # 一度に可能停止数
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.16

```

## StatefulSet

- DaemonSet 同様 Replicaset の特殊系リソース

### ReplicaSet との違い

- 作成される Pod 名のサフィックスに数字のインデックスが付与
  - Pod 名は変わらない
- データを永続化するための仕組みがある
  - StatefulSet によって作成される各 Pod に対して、PersistentVolumeClaim を設定できる
    - クラスタ外のネットワーク越しに提供される PersistentVolume(永続化領域)を Pod にアタッチできる
    - 1 つの Pod が専有することも・複数の Pod で共有することもできる

```yaml=
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sample-stateful-set
spec:
  serviceName: sample-stateful-set
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.16
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1G
```

### スケーリング

- Replicaset 同様、[kubectl apply -f]または[kubectl scale]でスケール可能
- Replicaset・DaemonSet などと違いデフォルトでは Pod を同時に 1 つずつ作成/削除するため少しだけ時間がかかる傾向にある
  - スケールアウトする際は、インデックスが一番小さいものから 1 つずつ Pod を作成し、作られた Pod が Ready になってから次の Pod を作成
  - スケールインする際は、インデックスが一番大きいものを削除していく
  - Replicaset はランダムに Pod が削除されていく
    - 特定の Pod がマスターとなるような APP は向かない

### ライフサイクル

- ReplicaSet などと異なり、複数 Pod を並行して作成しない（1 つずつ作成）
  - spec.podManegementPolicy を Parallel に設定することで同時起動させることも可能
  - デフォルトは OrderedReady が設定されている

### アップデート戦略

- DaemonSet と同様に OnDelete と RollingUpdate

#### OnDelete

- type 以外に指定可能な設定項目はない

```yaml=
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sample-statefulset-onedelete
spec:
  updateStrategy:
    type: OnDelete
  serviceName: sample-statefulset-onedelete
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.16

```

#### RollingUpdate

- StatefulSet では永続データがあるため、Deployment とは異なり余分なデータを作成してのローリングアップデートはできない
  - 一度に停止可能な Pod 数を指定することも出来ず、1 つの Pod ごとに Ready になってから行う
  - spec.PodManagementPolicy が Parallel の場合でも 1 つずつ行う
  - partition を設定することで、全体の Pod のうちどの Pod まで更新するかを指定
    - partition の値よりも小さいインデックスを持ち Pod はアップデートされない

```yaml=
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sample-statefulset-rollingupdate
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 3 # 0,1,2のPodはアップデートされない
  serviceName: sample-statefulset-rollingupdate
  replicas: 5 # 0~5のPodが作成される
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.16
```

#### 領域確認&解放

```shell=
# StatefulSetに紐づくPersistentVolumeの確認
$ kubectl get persistentvolumes

# StatefulSetが確保した永続化領域の解放
$ kubectl delete persistentvolumes www-sample-statefulset-{0..2}
```

## Job

- コンテナを利用して一度限りの処理を実行させるリソース
  - N 並列で実行しながら、指定した回数のコンテナの実行を保証するリソース

### ReplicaSet との違い

- 起動する Pod が停止することを前提にして作られているか
  - Replicaset などでは Pod の停止は予期せぬエラー

> rsync や S3 などへアップロードなどに用いる

### 挙動の違い

- spec.template.spec.restartPolicy に OnFailure または Never を指定
  - Never
    - Pod 障害時には新規 Pod を作成
  - Onfailure
    - 再度同一の Pod を利用し Job を再開

### タスク型とワークキュー型

- 成功回数を指定する: spec.completions
  - デフォルト 1
- 並列度を指定する: spec.parallelism
  - デフォルト 1
- 失敗を許容する回数: spec.backoffLimit
  - デフォルト 6

成功回数は後から変更することができないが、並列度と失敗許容回数は途中から変更可能
→ マニフェストを書き換えて apply

```yaml=
apiVersion: batch/v1
kind: Job
metadata:
  name: sample-job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 10
  template:
    spec:
      containers:
        - name: tools-container
          image: amsy810/tools:v2.0
          command: ["sleep"]
          args: ["60"]
      restartPolicy: Never

```

## CronJob

- Cron のようにスケジュールされた時間に Job を実行する
- CronJob が Job を管理＞ Jon が Pod を管理

### 作成

- 作成した直後は、Job は作成されておらず指定の時間になると Job が作成される

### 一時停止

- メンテナンスなどで Job が作成されてほしくない場合、suspend(一時停止)可能
  - spec.suspend が true にされているものはスケジュール対象から外れる

### 同時実行

- Job の実行が意図した間隔内で正常してる場合、指定せずとも同時実行されることなく新たな Job が実行される
  - 一方、古い Job がまだ実行してる際に新しく Job を作成するかの制御

spec.concurrencyPolicy に指定、デフォルトは Allow

- Allow
  - 同時実行に対して制限を行わない
- Forbid
  - 前の Job が終了していない場合、次の Job は実行しない
- Replace
  - 前の Job をキャンセルし、Job を開始する

### 実行開始期限

- 指定した時間になると Kubernetes Master が Job を作成
  - Kubernetes Master が一時的にダウンしていた場合など、開始時間が遅れた場合の許容秒数を(spec.startingDeadlineSeconds)に指定可能

### CronJob の履歴

- spec.successfulJobHistoryLimit
  - 成功した Job を保存する数
- spec.failedJobHistoryLimit
  - 失敗した Job を保存する数

Job に紐づく Pod の[Complete],[Error]がステータスで残っているため
kubectl logs で確認可能

```yaml=
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: sample-cronjob
spec:
  schedule: "*/1 * * * *"       # cron指定
  concurrencyPolicy: Allow      #同時実行の有無
  startingDeadlineSeconds: 30   # 開始時間が遅れた場合の許容秒数
  successfulJobsHistoryLimit: 5 # 成功ログ保持数
  failedJobsHistoryLimit: 3     # 失敗ログ保持数
  suspend: false                # 一時停止
  jobTemplate:
    spec:
      completions: 1
      parallelism: 1
      backoffLimit: 0
      template:
        spec:
          containers:
            - name: tools-container
              image: amsy810/random-exit:v2.0
          restartPolicy: Never
```
