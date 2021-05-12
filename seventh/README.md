# Config & Storage APIs カテゴリの概要
- Config Storage APIsカテゴリに分類されるリソースは、コンテナに対して設定ファイル、パスワードなどの機密情報などをインジェクトしたり、永続化ボリュームを提供したりするためのリソースです。
    - Secret
    - ConfigMap
    - PersistentVolumeClaim
    
の３つをよく私たちは使います。

## 環境変数の利用
- Kubernetesでは、個別のコンテナに対する設定の内容は、環境変数やファイルが置かれた領域をマウントして渡すことが一般的です。
- Kubernetesで環境変数を渡す際にはPodテンプレートにenvまたはenvFromを指定します。大きく分けて、下記の５つの情報源から環境変数を埋め込むことが可能です。
    - 静的設定
    - Podの情報
    - コンテナの情報
    - Secretリソースの機密情報
    - ConfigMapリソースの設定値
    
## 静的設定
静的設定では、spec.containers[].envに静的な値として定義します。

例
```yaml
apiversion: v1
kind: Pod
metadata:
  name: sample-env
  labels:
    app: sample-app
spec:
  containers:
    - name nginx-container
      image: nginx:1.16
      env:
      - name: MAX_CONNECTION
          value "100"
```

デフォルトのタイムゾーンはUTCに設定されています。下記のような環境変数を指定しておくことでタイムゾーンをJSTに合わせることができます。
```yaml
env:
  - name: TZ
    value: Azia/Tokyo
```

## Podの情報
- どのノードで起動しているか、Pod自身のIPアドレス、起動時間などのPodに関する情報はfieldRefを使うことで参照できます。
- 参照可能な値に関しては、「kubectl get pods -o yaml」などで確認できます。一部抜粋すると、登録したマニフェストの情報とは別に、Pod
のIPアドレスやホストの情報と言った様々な情報が追加されていることが確認できます。
```yaml
env:
- name: K8S_NODE
  valueFrom:
    fieldRef:
      fieldPath: spec.nodeName
```
## コンテナの情報
- Podの情報と同様に、コンテナのリソースに関する情報はresourceFieldRefを使うことで参照することが可能です。
- Podには複数のコンテナの情報が含まれており、各コンテナに関する情報についてはfieldRefでは参照できないです。
```yaml
env:
- name: CPU_REQUESTS
  valueFrom:
    resourceFieldRef:
      containerName: nginx-container
      resource: limits.cpu
```

# Secret
- データベースなどに対して接続する際にはユーザー名やパスワードなどの機密情報が必要になります。そこで２つの方法がよく使用されています。
    - Dockerビルド時にコンテナイメージに埋め込んでおく
    - PodやDeploymentのマニフェストに記載して渡す
    
## Dockerビルド時にコンテナイメージに埋め込んでおく
- イメージビルド時、アプリケーション側やコンテナの環境変数及び実行引数などにパスワードなどを埋め込んでおくことができます。
- しかし、イメージ自体い機密情報が入ってしまうと、Dockerレジストリに機密情報ごとアップロードすることになってしまうため、セキュリティ上好ましくありません。
- また、機密情報の内容を変更するだけであっても再度イメージをビルドしなければならず、利便性も良くありません。

## PodやDeploymentのマニフェストに記載して渡す
- この場合もマニフェスト自体に機密情報が入ってしまうため、マニフェストの管理が困難になります。
- 例えばGitリポジトリのような外部のサーバで機密情報が入ったマニフェストを管理するとセキュリティリスクになります。
- また複数のアプリケーションから同じ機密情報を利用したい場合に、マニフェスト群の複数箇所にそれを記述することになるため、管理しにくくkなるという問題もあります。
