# Kubernetes における監視

- Saas の Datadog,OSS の Prometheus の二つが有名

## Datadog

- 通常 Datadog を Kubernetes 上にでデプロイする場合、DaemonSet を利用して DatadogAgent を各ノード上で起動します。（環境によってはサイドカーパターンでデプロイすることもあります。）
- デプロイされた Agent は各ホストの CPU 使用率やディスク使用率といったメトリクスじゃもちろんのこと、各ノード上のコンテナの CPU 使用率などのメトリクスも取得します。
- Datadog を Kubernetes 上で連携して起動する場合には、コンテナにいくつかの環境変数を渡す必要があります。また、クラスタレベルのメトリクスを収集するコンポーネント(kube-state-metrics を)を追加でインストールして連携することが推奨されています。
- kube-state-metrics と連携することで Deployment で起動している Pod すうや要求起動数などのメトリクスを扱うことができるようになります。他にも Job の成功数と失敗数や、ローリングアップデート時の Pod の起動数の推移を監視することも可能です。
