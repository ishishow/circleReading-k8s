# Kubernetes 環境での CI/CD

- 実際に運用を行う際には手動での kubectl コマンドの実行は可能な限り避けるべきです。

## GitOps

- GitOps とは、Git を用いた CI/CD 手法の一つです。
- Kubernetes 情にデプロイするアプリケーションを開発する際には下記のようなフローが発生します。
  - アプリケーションのソースコードを変更する。
  - アプリケーションのテストを行う
  - コンテナイメージを作成する。
  - コンテナイメージをコンテナレジストリにプッシュする
  - Deployment などのマニフェストを書き換える
  - kubectl apply 相当の処理を実行してクラスタに反映する

## GitOps に適した CI ツール

- GitOps を実現する CI ツールはアプリケーションのテスト/Docker イメージのビルド/ナブフェスとリポジトリの更新など基本的な処理が行えるものであればどのようなものでも構いません
- Kubernetes と親和性が高いツールをあげると Kubernetes の CustomResource として CI パイプラインを定義可能な Tekton と Kubernetes 上でコンテナをビルドする Kaniko を組み合わせるといった選択肢があります。

## Kubeval

- Kubeval はマニフェストファイルの YAML の構造が特定の API のバージョンに準拠しているかを検証できる OSS です。
- 例えばよくある間違いとして、Annotation の値には文字列か Null 値が必要にもかかわらず、数値を指定してしまう場合が挙げられます。こういった間違いは kkubeval によって型チェックが行われるため、事前に検出可能です。

## Conftest

- Conftest はマニフェストファイルのユニットテストを行う OSS です。OpenPolicyAgent で利用されている Rego 言語でポリシーを記述し、下記のようなテストを CI ツールに組み込むことが可能です。
  - 特権コンテナ
  - リソースに付与するラベルのルールを決めている場合に、必要なラベルをつけ忘れていないか
  - イメージのタグが latest などではないか
  - リソース制御に関する設定を定義してあるか

## Open Policy Agent

- Open Policy Agent は汎用的なポリシーチェックを行うプロダクトで、Kubernetes と連携する Gatekeeper と組み合わせることでマニフェストが適用された際にポリシーチェックを行うことが可能になります。

## GitOps に適した CD ツール

## ArgoCD

- ArgoCD は GitOps を実現するための CD ツールです。指定したリポジトリを監視し、Kubernetes クラスタに対してマニフェストを適用します。DeployAgent に該当します。

- ArgoCD では、Application リソースを作成することで特定のリポジトリの特定のパスにあるファイルを Kubernetes クラスタに適用する仕組みになっています。基本的には自動的に同期されるように設定しておくことが推奨されていますが、手動で管理することもできます。マニフェストの差分を表示することもできるため、手動で管理する場合には差分を確認して適用できます。ArgoCD には他にも、リポジトリから削除されたリソースを自動的に削除する Prune オプションや、自動回復させる selfHeal オプションなども用意されています。

- 今回は紹介しませんが、ArgoCD では生のマニフェスト以外にも Kustomize や Helm などのマニフェストも扱えるため、本書の執筆時点においては GitOps を始めるための最も完成形に近いプロダクトが ArgoCD であると言える。

-
