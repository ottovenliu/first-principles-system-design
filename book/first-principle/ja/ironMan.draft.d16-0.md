# Day 16 | Dev / Staging / Prod マルチ環境ガバナンスとアーキテクチャ戦略: AWSマルチ環境設定管理とデプロイメント戦略

これまでの2つのトピック、<Infrastructure as Code: Terraformでインフラストラクチャをコード化しバージョン管理する>と<完全なCI/CD自動化実装 - GitHub Actions × CodePipeline × CodeBuild>で、**「インフラストラクチャを予測可能、反復可能、バージョン管理可能にすること」**の重要性と、**「既存のビジネスロジックがコード変更によって破壊されるのをアクティブに保護すること」**について、それぞれ議論しました。両方とも`テストとデプロイメントのための安定した環境と固定されたビジネスロジック検証プロセスを持たない`という問題を解決することを目的としており、私たちの**`ビジネスロジック検証`**が安定した土壌の中でそびえ立つ木のように力強く成長できることを保証します(台湾の某銀行の話ではありません)。環境の重要性と対応する痛点への解決策について簡単に話した後、今回は議論します - **`どのような環境が必要なのか?`**

最近、友人とリージェント台北のImpromptu by Paul Leeというレストランを訪れ、2025年夏の味を楽しむ機会がありました。オープンキッチンと客席エリアを組み合わせたユニークなデザインで、シェフとそのチームが食材、スパイス、厳選されたワインを使って2025年夏の美しく壮大なタペストリーをどのように提示するかをシンプルかつ透明に見ることができました。私はこのメニューデザインが大好きでした。明確で、鮮明で、率直なエレガンスを感じることができました。

しかし話は変わりますが、ミシュランスターの料理はどのようにデザインされるのでしょうか？新しいコンセプトはすべてのグルメにとってエキサイティングなニュースですが、もしカウンターで作ってお客様に直接提供し、それが失敗したらどうなるでしょうか？塩が多すぎたり、調理時間が間違っていたり - 良くても評判を失い、最悪の場合、お客様が腹痛を起こし、翌日にはあなたのレストランの名前が見出しで台無しになります。

では一歩下がって考えましょう。公式リリース前に、レストランの「メインキッチン」バックステージの片隅で試作してみるのはどうでしょうか？

これは良い方法です。お客様が実験的な料理をすぐに味わうことを避けられます。しかし、お客様には見えませんが、既存のキッチンの正常な運営を妨害する可能性があります。調理準備に使用されているまな板を占領したり、必要なときにソースが別のシェフによって使い切られていることに気づいたり、または私たちの実験からの煙が隣の慎重に調理された料理に影響を与える可能性すらあります。

つまり、資金とスペースが許すなら、完全に独立した「テストキッチン」で私たちの新しいコンセプトを実現しようとするのが最適な解決策のようです。ここでは、私たち自身の完全な鍋、フライパン、食材、オーブンのセットがあり、外部のレストラン運営に影響を与える心配なく、自由に実験、失敗、再試行できます。

これが**開発環境(Dev)**です。完全に分離されたサンドボックスであり、すべての開発者が何かを「壊す」ことや誰かに影響を与えることを心配せずに、自由に自分のコードを書き、修正し、テストできます。**「メインキッチン」バックステージ(共有環境)**で他の機能をテストしている同僚に影響を与えることを防ぎ、その逆も同様です。最も重要なことは、本番環境(Production / Prod)で直接コードを書くことを最大限避けることです。不正なデータベースコマンドのような小さなミスでも、すべてのユーザーのサービス中断やデータ損失につながる可能性があります。**`これは絶対に、絶対に禁止された行為です。`**

正式に提供される前に、私たち自身の**R&Dキッチン(開発環境)**で新しい料理を作り、それは天国の味がします。次は何でしょうか？直接メニューに載せて、最も重要な食評論家に販売しますか？

シェフの帽子の上にマスターシェフが隠れていなくて、私たちが提供しているのがラタトゥイユである場合(Ratatouilleは素晴らしい映画です、時間があれば見てください)、これは非常に危険な動きです。

正式なローンチの前に、**「最終リハーサル」**のための場所が必要です。この場所を「ステージングキッチン」と呼びます。その設定、ストーブ、ワークフロー、さらには皿のブランドまで、正式な「メインキッチン」と同一でなければなりません。ここで、実際の提供プロセスをシミュレートする必要があります：ウェイターは注文から提供までの全プロセスを通さなければならず、シェフチームはピーク時のプレッシャーの下で意図されたプロセスに従って料理を安定して作れるかシミュレートし、最後にこの新しい料理が他のクラシック料理と一緒に提供されたときにプロセスと競合しないかテストする必要があります。

これが**Staging / プリプロダクション環境**の重要性です。私たちは、ビジネスロジックが要件と品質基準を満たしていることを確認するために、さまざまなシナリオで包括的にテストするための環境を持たなければなりません。

今、ソフトウェア開発ライフサイクルを明確な「進行」に抽象化できます：

```python
Dev => Staging => Prod
```

- **開発環境(Dev): テストキッチン**
  - **目的**: 機能開発、ユニットテスト、迅速な反復。
  - **特性**: 混沌は常態です。頻繁な変更、デプロイメント、完全な解体さえ許可されます。開発者はより高い権限を持ちます。データは通常、偽物または匿名化されています。
  - **コアフォーカス**: 開発効率。

- **Staging環境: ステージングキッチン**
  - **目的**: 統合テスト、ユーザー受け入れテスト(UAT)、パフォーマンスストレステスト。すべての実際のシナリオをシミュレート。
  - **特性**: 環境は本番環境に極めて近くなければなりません。設定、OS、ネットワークルール、データベースバージョンなどは一貫している必要があります。デプロイメントプロセスも正式なローンチプロセスと同一である必要があります。データは本番環境のレプリカ(脱感作後)である可能性があります。
  - **コアフォーカス**: 安定性と一貫性。これが本番前の品質の最後の防衛線です。

- **本番環境(Prod): ミシュランスターレストラン**
  - **目的**: 実際のユーザーにサービスを提供し、価値を創造。
  - **特性**: 極めて安定、安全、高性能。いかなる変更も厳格な承認と標準化されたプロセスを経る必要があります。アクセス制御が最も厳格で、通常、自動化ツールまたは非常に限られた数の承認された担当者のみがデプロイメントを実行できます。
  - **コアフォーカス**: 信頼性とセキュリティ。

この**`Dev => Staging => Prod`**の一方向フローは、ソフトウェア品質保証の`コアワークフロー`です。コードは水のように、下位環境から上位環境にのみ流れることができ、決して逆流してはいけません。各フローは**`「品質ゲート」`**でのチェックであり、**CI(継続的インテグレーション)**プロセスでの**`自動チェック`**によって**`既存のビジネスロジックがコード変更によって破壊されるのを保護する`**か、<バージョン管理戦略(PRレビュー戦略)>で述べられた**Peer Review Gate**、**Business Review Gate**、**Quality Review Gate**の3方向検証かにかかわらず。**`ビジネスロジックの実装条件が満たされていない`**場合は、直ちに緊急修理のために返却する必要があります。私たちは深く明確に理解しなければなりません - **`システムは抽象的なビジネスロジックの実装である`**。

現代のソフトウェア開発において、マルチ環境管理はアプリケーションの安定性と信頼性を確保する鍵です。次に、AWS上でDev/Staging/Prodマルチ環境アーキテクチャを構築し管理する方法を深く掘り下げ、Dockerコンテナ化、ECSクラスタ管理、EKS/K8sオーケストレーションなどの核心技術に焦点を当てます。

> 1. マルチ環境アーキテクチャ設計原則 - 環境分離戦略とIaC実践
> 2. Dockerコンテナ化戦略 - マルチステージビルドと環境別設定
> 3. Amazon ECSクラスタ管理 - サービス定義とオートスケーリング設定
> 4. Amazon EKSとKubernetes管理 - クラスタ設定、アプリケーションデプロイ、Ingressセットアップ
> 5. CI/CDパイプライン統合 - GitHub Actionsワークフローとブルー/グリーンデプロイメント
> 6. 設定管理とシークレット管理 - AWS Secrets ManagerとK8s ConfigMap/Secret
> 7. 監視とログ管理 - CloudWatchとPrometheus/Grafana統合
> 8. セキュリティベストプラクティス - ネットワークセキュリティとRBAC設定
> 9. コスト最適化戦略 - 自動化されたリソース管理とスポットインスタンス統合
> 10. 災害復旧とバックアップ - リージョン間バックアップとデータベースフェイルオーバー

## 1. マルチ環境アーキテクチャ設計原則

### 1.1 環境分離戦略

私たちは上記で、レストラン(私たちのシステム)を「R&Dキッチン(Dev)」、「ステージングキッチン」、「フラッグシップストア(Prod)」に分ける必要があることを議論しました。実際にどのように「境界を定める」か、そしてこれら3つの土地の「建築基準」が一貫していることを確保する方法について話しましょう。

```python
本番環境
├── 高可用性設定
├── オートスケーリング
├── 完全な監視とアラート
└── 厳格なデプロイメントプロセス

Staging環境
├── 本番環境のミラー
├── 完全な機能テスト
├── パフォーマンステスト環境
└── 統合テスト検証

開発環境
├── 迅速なデプロイメント
├── 開発者フレンドリー
├── リソースコスト最適化
└── 柔軟な設定調整
```

最もシンプルな方法は、もちろん各キッチンをロックし、対応する機能的ニーズを持つ人々に専用の鍵を渡すことです。まずドアをロックして、人々が歩き回るのを防ぎます。私たち自身の人々がうっかりパラメータを調整したり、サービス麻痺を引き起こす設定エラーを起こすだけでも十分悪いです。少なくとも、鍵を持たない泥棒や強盗が入ることさえできないことを確保しなければなりません。したがって、私たちの最初のコア原則はシンプルです：**`爆発半径を最小化する`**。これは、1つの環境で災害が発生した場合(例：ハッキングされた、または設定エラーによるサービス麻痺)、他の環境に絶対に影響を与えてはいけないことを意味します。

AWSで、この目標を達成するためのゴールドスタンダードは**`マルチアカウント戦略`**と呼ばれます。

**1. AWS Organizations: クラウド企業本社**

**`AWS Organizations`**は私たちの「企業本社」であり、これを通じて異なるAWSアカウントを管理し、基本的なポリシーを策定できます。グループが財務、営業、法務、HR、開発、保守のための異なる事業部門を持っているように、`DevOU`と`ProdOU`を事業部門の概念として作成できます。`Account-Dev`(部門)は`DevOU`(事業部門)に配置され、`Account-Staging`と`Account-Prod`は`ProdOU`に配置されます。次に、**`サービスコントロールポリシー(SCP)`**を設定して、その下のアカウントが**できること**または**できないこと**を制限できます。これはグループによって発行された「最高執行命令」のようなものです。事業部門ポリシーとグループ本社の間に矛盾がある場合、`SCP`の規制が優先的かつ強制的に従われます。そして`AWS Organizations`にはもう1つの利点があります：事業部門のすべての請求書とコストが本社で集約され、監視と管理が便利です。

`SCP`は人的エラーや悪意のある行動を根本から防ぎ、事後に修正するよりもはるかに効果的です。各環境の「行動憲法」と「物理法則」を定義します。例えば：

- DevOU下のすべてのアカウントに対して:
  - 外部ネットワークに接続されたデータベースの作成を禁止(RDS public access = false)。
  - CloudTrail(監査ログ)の無効化を禁止し、すべての操作が記録されることを保証。
  - リソース作成をアジア太平洋地域(例：東京、シンガポール)に制限し、誤って高価なヨーロッパやアメリカ地域にリソースを作成することを防ぐ。
- ProdOU下のすべてのアカウントに対して:
  - 特定の管理者ロールを除き、データベースやS3バケットの削除を禁止。

**2. IAM権限: 異なるアイデンティティのための「パス」**

アカウント分離後、人々はどのように仕事をするために入るのでしょうか？答えはIAM Rolesであり、各アカウントでユーザー名とパスワード(IAM User)のセットを作成することではありません。

- **概念**: Management AccountにIAM Usersを作成し、次に異なるIAM Roles(例：DeveloperRole、QARole、OperatorRole)を定義します。次に、これらのUsersに他のアカウントでタスクを実行するために「Assume Role」させます。
- **たとえ話**: 私たちは会社の従業員(IAM User)であり、会社は私たちに異なる作業バッジ(IAM Role)を与えます。
  - 「開発者バッジ」があれば、Account-Devのすべての部屋にスワイプで入れます。
  - 「QAバッジ」があれば、Account-Stagingに入れますが、観察と記録のみで、機器を変更できません。
  - 「フラッグシップストアマネージャーバッジ」を持つ者だけが、承認後にAccount-Prodに入って操作できます。

このアプローチは権限管理を一元化し、安全で明確にします。

### 1.2 Infrastructure as Code (IaC)

環境間に強固な壁があることを確保しましたが、「ステージングキッチン」と「フラッグシップストア」の内装、配管、ストーブが同一であることをどのように保証しますか？これが`Infrastructure as Code (IaC)`の力が発揮される場所です。`青写真(IaC)`に従って、同一のアーキテクチャと環境を迅速に構築できます。<Infrastructure as Code: Terraformでインフラストラクチャをコード化しバージョン管理する>で青写真の重要性について述べました。それはインフラストラクチャを`予測可能`、`反復可能`、`進化可能`にします。Terraformを例に続けましょう。

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   └── variables.tf
│   └── ec2_instance/
│       ├── main.tf
│       └── variables.tf
│
└── environments/
    ├── dev/
    │   ├── main.tf
    │   └── terraform.tfvars
    ├── staging/
    │   ├── main.tf
    │   └── terraform.tfvars
    └── prod/
        ├── main.tf
        └── terraform.tfvars
```

- `modules/`は標準化された建築モジュールを保存します。
- `environments/`は各環境の「建設指示」を保存します。

`environments/dev/main.tf`では、偽のコードで提示します：

```yaml
# VPCモジュールを参照
module "my_vpc" {
  source = "../../modules/vpc"
  cidr_block = var.vpc_cidr
}

# EC2モジュールを参照
module "web_server" {
  source      = "../../modules/ec2_instance"
  instance_type = var.instance_type
  vpc_id      = module.my_vpc.id
}
```

気づきましたか？この`main.tf`ファイルは、dev、staging、prodでほぼ同じに見えます。実際の違いは、その隣にある`terraform.tfvars`パラメータファイルにあります：

- `environments/dev/terraform.tfvars`
```
vpc_cidr      = "10.10.0.0/16"
instance_type = "t3.micro"  // dev環境で小型マシンを使用してコスト削減
```

- `environments/prod/terraform.tfvars`
```
vpc_cidr      = "10.20.0.0/16"
instance_type = "m5.large"  // 本番環境で大型マシンを使用してトラフィックを処理
```

Dev環境をデプロイしたいとき、`environments/dev`フォルダに入って`terraform apply`を実行するだけです。Terraformは自動的に`dev`パラメータを読み取り、共有モジュールに適用し、開発仕様を満たす環境を構築します。`Prod`のデプロイも同様です。

これは私たちのIaC目標を完璧に達成します：
1. **一貫性**: すべての環境の「アーキテクチャ」は同じ青写真(modules)から来ます。
2. **追跡可能性**: すべてのインフラストラクチャ変更はコードを変更することによって行われなければなりません。すべての変更はバージョン管理システム(Gitなど)に記録され、レビューと追跡が可能です。
3. **再現性**: 環境全体が破壊されても、同じコードで同一のものを迅速に再構築できます。

## 2. Dockerコンテナ化戦略

IaCを使用して3つのキッチンのアーキテクチャ青写真を既に描きました。今、私たちの「調理ツール」を標準化する必要があります。これがDockerコンテナ化戦略が解決しようとしている問題です。

私たちは皆、このクラシックなシナリオを聞いたことがあり、おそらく経験したことさえあります：
> 開発者A: 「この機能は私のマシンで正常に動作します！なぜデプロイメント後に壊れたのですか？」

この「私のマシンでは動作する」問題は、Dockerが治そうとしている慢性疾患です。問題の根本は、開発者のコンピュータ、テストサーバー、本番サーバーの「環境」に微妙だが致命的な違いがあることです：
- 異なるOSバージョン(Ubuntu 20.04 vs 22.04)
- 依存ライブラリの異なるバージョン(Python 3.8 vs 3.9、OpenSSL v1 vs v3)
- 異なる環境変数設定
- 異なるハードウェアパラメータ
- 異なるシステムパスやファイル権限

すべてが同じでも、1つはテストに合格し、もう1つは失敗する可能性があります。Windows(偏見はありませんが、ここが`根本的な違い`が最も発生しやすい場所です)のような同じOSでも、お供え、聖水、またはオムニシアを賞賛して機械の精霊がスムーズに実行するよう祈るしかありません。

**`Docker`**の核心的なアイデアは、アプリケーションを「実行環境全体」と一緒にパッケージ化して持ち運ぶことです。

シェフに「ブッフ・ブルギニョン」のレシピを渡して、自分のキッチンの鍋、ストーブ、調味料を使わせる(各キッチンの違いにより味が異なる可能性が高い)代わりに、半完成品、特別なソース、精密な温度制御を備えた「標準化された加熱ボックス」を渡した方がよいでしょう。彼がすることは、ボタンを押すだけです。

この「標準化された加熱ボックス」が**Dockerコンテナ**です。このボックスには以下が含まれます：
1. 私たちのアプリケーションコード(料理そのもの)
2. ランタイム、例：Node.js v18またはJava 17(特定の調理ソース)
3. すべてのシステムレベルのライブラリとツール(特別な塩と胡椒)

この「ボックス」を`Dev`環境の電子レンジや`Prod`環境の最高級オーブン(両方とも資格のあるDocker Hostsである限り)に入れても、加熱後に出てくる料理は全く同じ味であることが保証されます。これがコンテナ化がもたらす`ポータビリティ`と`一貫性`です。

### 2.1 マルチステージビルド最適化

Dockerfileビルドプロセスを2つのステージに分割します。組立ラインのようなものです：
- **ステージ1(ビルダーステージ):** 広々として設備の整った「処理工場」。
- **ステージ2(最終ステージ):** クリーンで軽量な「配送ボックス」。

```dockerfile
# Dockerfile.multi-stage
# ビルドステージ
FROM node:18-alpine AS builder
WORKDIR /app
# ビルドに必要なpackage.jsonのみをコピー
COPY package*.json ./
# 本番に必要な依存関係のみをインストール
RUN npm ci --only=production

# 開発ステージ
FROM builder AS development
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# 本番ステージ
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

これを行う利点は明らかです：
- **非常に小さな最終イメージ:** `node_modules`(本番)、`dist`、`package.json`のみが含まれます。ソースコードも開発ツールもありません。
- **より高いセキュリティ:** 攻撃面が最小化されます。パッケージが攻撃されたり、トロイの木馬が埋め込まれたりするニュースは一般的ではありませんが、聞いたことがあります。コンパイルツール(gccやnpmなど)や開発ライブラリは、ハッカーの脆弱性になる可能性があります。したがって、本番環境には`「最低限必要な」`実行コンポーネントのみが含まれるべきです。
- **明確なビルドプロセス:** 「どのようにビルドするか」と「どのように実行するか」の関心事を分離します。

### 2.2 環境別設定管理

1つのボックスを異なるキッチンにどのように適応させますか？

標準化された「加熱ボックス」は既にありますが、今新しい問題があります：
> Dev環境では、この料理は`dev-database`に接続する必要があります。Prod環境では、`prod-database`に接続する必要があります。
> 
> しかし、私たちは1つのボックス(Docker Image)しかないので、どうすればいいですか？

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"

# docker-compose.prod.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: production
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
    restart: unless-stopped
    ports:
      - "80:3000"
```

まず、鉄の掟、絶対的なルールを守ることを覚えておく必要があります：
> #### **一度ビルドし、どこでも実行**

異なる環境のニーズのために、封印されたお弁当箱を開けてはいけません。これは**`イメージの不変性`**を完全に違反します。異なる環境用に異なる`Images`をビルドしてはいけません(例：`my-app:dev`、`my-app:prod`)。これは、私たちが確立したすべての**`一貫性保証`**を破壊します。

```
Devでテストに合格したImageは、StagingとProdにデプロイされる全く同じ、変更されていないImageでなければなりません。
```

しかし、`外部注入`のニーズが本当にある場合はどうすればよいでしょうか？

したがって、私たちの「標準加熱ボックス」にはいくつかのスロットが予約されています。「データベース接続スロット」や「APIキースロット」のようなものです。ボックス自体はこれらのものを運びません。代わりに、特定のキッチンの指定された場所に置かれたとき、キッチンの「自動アーム」(コンテナオーケストレーションシステム)が対応するプラグを差し込みます。

または、一部の日本のインスタントラーメンのように、お湯の入口と水の出口は異なる穴であり、それぞれ独自の目的があります。私たちはお湯、熱いお茶(これも良い、試しました)、またはホットミルクティー(???、見たことがありますが尊重します)を注ぎ、排水できますが、中のメインボディ(麺と調味料)は変わりません。

YAMLの例を振り返ると、いくつかの手がかりが見つかります：

```yaml
# docker-compose.dev.yml
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug

# docker-compose.prod.yml
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
```

**`環境変数`**を宣言することで、異なるインターフェースを設定できます。これは最も標準的な方法であり、12-Factor Appの哲学に最も適合します - **アプリケーションコードは設定をハードコードせず、環境変数から読み取る必要があります**。

コンテナが起動するとき(ECS/Kubernetesによって管理される)、次のように実行します：
- Dev環境で: `docker run -e DATABASE_HOST=dev.db.internal ... my-app`
- Prod環境で: `docker run -e DATABASE_HOST=prod.db.internal ... my-app`

より複雑な設定(例：完全な`settings.json`)の場合、コンテナオーケストレーションシステムは、起動時に対応する環境の設定ファイルをコンテナ内の特定のパス(例：`/etc/config/settings.json`)に「マウント」できます。私たちのアプリケーションは、この固定パスからファイルを読み取るだけです。

最後に、より高度で安全な方法があります：**`起動時に設定サービスから取得する`**。コンテナが起動すると、アプリケーションは割り当てられたアイデンティティ(例：AWS ECS Task Role)を使用して、外部サービス(AWS Secrets ManagerやParameter Storeなど)を呼び出し、必要なデータベースパスワード、APIキーなどを取得します。こうすることで、環境変数にも平文で表示されず、最高レベルのセキュリティです。

## 3. Amazon ECSクラスタ管理

標準化されたキッチン(IaC)を既に構築し、標準化された「半完成食事キット」(Docker Images)を設計しました。今、バックエンド全体の運営を管理するための知的で効率的な「ヘッドシェフシステム」が必要です。このシステムは以下を知る必要があります：
- 料理(私たちのアプリケーション)の標準調理手順は何ですか？
- ピーク時(高トラフィック)にこの料理を作るために、何台のストーブを同時に動作させるべきですか？
- ストーブが故障した場合(サーバー障害)、調理を続けるために自動的に新しいものと交換する方法は？

この「ヘッドシェフシステム」が**`Amazon ECS (Elastic Container Service)`**です。

### 3.1 ECSサービス定義

ECSを理解するために、まずその4つのコアコンポーネントを知る必要があります：`ECSクラスタ - 「キッチン自体」`、`タスク定義 - 「標準作業手順(SOP)カード」`、`タスク - 「強火ストーブ上の中華鍋」`、`サービス - 「疲れ知らずのヘッドシェフ」`。

1. **ECSクラスタ - 「キッチン自体」**
- **コア概念:** すべてのコンテナ化されたアプリケーションを含むために使用される論理的なグループ化。`Dev Kitchen Worktop`、`Staging Kitchen Worktop`、`Prod Kitchen Worktop`と考えることができます。何平方フィートのキッチン作業エリアがあるかを人々に伝えるようなもので、内部の実際のレイアウトには触れません。それ自体はコストがかからず、単なる管理境界です。
- **重要なポイント:** クラスタが動作するには「計算能力」が必要です。キッチンにはストーブが必要なのと同じです。計算能力のソースには2つのモードがあります：
  - **EC2モード:** 私たち自身でバッチの「ストーブ」(EC2クラウドサーバー)を購入し、キッチンに登録します。ストーブのブランドとモデルを完全に制御でき、自由に調整できますが、メンテナンスと管理にも責任があります。
  - **Fargateモード(サーバーレス - 業界の主流トレンド):** ストーブを購入しません。調理が必要なときに、AWSに伝えるだけです：「500Wの電力と1平方メートルのエリアを持つストーブが必要です。」AWSは自動的に、その広大なリソースプールから要件を満たすストーブを出現させ、終わったら消えます。
  - 大多数の新しいアプリケーションにとって、Fargateが優先選択です。面倒なサーバー管理から解放され、アプリケーション自体により集中できます。

2. **タスク定義 - 「標準作業手順(SOP)カード」**
- **概念:** これはECSのコアブループリントです。「料理の作り方」を詳細に定義するJSON形式の取扱説明書です。
- **重要なポイント:** シェフのための完全なSOPカードのようなもので、正確に記載されています：
  - `image:` どのバージョンの「半完成食事キット」を使用するか(Docker Image URL)。
  - `cpu & memory:` このタスクにどれだけの「火力」と「スペース」を割り当てるか。
  - `environment:` どのような「調味料」を注入するか(環境変数、例：DATABASE_URL)。
  - `secrets:` 「金庫」(AWS Secrets Manager)からどの「秘密のレシピ」を取得するか(APIキー、データベースパスワード)。
  - `logConfiguration:` 調理プロセスの記録(ログ)をどこに送信するか(例：CloudWatch Logsへ)。
  - `taskRoleArn:` このシェフにどのような「権限カード」(IAM Role)を与えるか、他のAWSサービスとのやり取りを許可する(例：S3からファイルを読む)。

3. **タスク - 「強火ストーブ上の中華鍋」**
- **概念:** タスク定義の「実行インスタンス」。ECSが実際にSOPカードに基づいてコンテナを起動すると、その実行中のコンテナがTaskです。Tasksの数を設定することで、同時にいくつの中華鍋が炒めているかを決定できます。

4. **サービス - 「疲れ知らずのヘッドシェフ」**
- **概念:** ServiceはECSの頭脳と執事です。その義務は、私たちが望む状態を維持することです。
- **重要なポイント:** キッチンのすべてが私たちが望む実行順序に従って動作することを確保します。
  - `desiredCount: { N }`: 「常に`{ N }`人前の『ブッフ・ブルギニョン』が火の上で調理されているようにしたい、多くも少なくもない。」
  - `Health Check`を通じて、ヘッドシェフはすべてのストーブが正常かどうか常にパトロールします。欠陥のあるストーブのために1つが失敗した(`HC fail`)ことがわかった場合、すぐにそれを捨てて新しいストーブを起動し、アクティブなストーブの数が常に3であることを確保します。
  - **ロードバランシング:** ストーブが点火されると(新しいTaskが起動)、ヘッドシェフは自動的にフロントのウェイター(Application Load Balancer)に通知し、今追加の提供オープニングがあることを伝え、顧客の注文(トラフィック)を送ることができます。

**実装例**

```json
{
  "family": "app-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "your-registry/app:${ENVIRONMENT}",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "${ENVIRONMENT}"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:${ENVIRONMENT}/database-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/${ENVIRONMENT}/app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### マルチ環境でのECS実践

ECSの意味を簡単に説明した後、私たちの`Dev Kitchen Worktop`、`Staging Kitchen Worktop`、`Prod Kitchen Worktop`戦略はECSレベルでどのように現れるのでしょうか？

- **クラスタ分離:** マルチアカウント戦略に従って、`Account-Dev`に`dev-cluster`を、`Account-Prod`に`prod-cluster`を作成します。これが最も徹底した分離です。
- **SOPカード(タスク定義)管理:** タスク定義の管理にはまだIaC(Terraform)を使用します。共有の`task-definition.tf`テンプレートを持ち、異なる環境用に異なるパラメータを渡します。
  - `dev.tfvars`: `cpu = 256 (1/4 vCPU)`、`memory = 512 (MB)`、`API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:DEV_ACCOUNT_ID...`
  - `prod.tfvars`: `cpu = 1024 (1 vCPU)`、`memory = 2048 (MB)`、`API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:PROD_ACCOUNT_ID...`
  - 私たちの`image`パラメータは**同じDocker Image**を指しますが、注入される`CPU`、`memory`、`Secrets`は依然として異なる環境設定から来ており、**`一貫性`**と**`不変性`**を維持していることに注意してください。
- **ヘッドシェフコマンド(サービス)管理:** IaCを使用して定義します。
  - `dev`環境のServiceは`desiredCount`が`1`だけで、コスト削減のために自動スケーリングが設定されていない可能性があります。
  - `prod`環境のServiceは`desiredCount`が`3`から始まり、次に説明する自動スケーリング戦略が設定されます。

### 3.2 ECS自動スケーリング設定

私たちのレストランは3人のシェフだけに頼ることはできません。時には予測可能で、火曜日の午後には1人で十分ですが、金曜日の夜には10人が必要になり、途切れない注文に対処する可能性があります。時には、群衆が突然押し寄せ、20人のシェフが必要になる場合もあります。

この時点で手動でシェフを追加または削除するのは遅すぎて非現実的です。したがって、「客流量」に基づいてスタッフを自動的に増減し、コストを最適化できる**`「顧客フロー - ストーブ監視連動システム」(ECS Service Auto Scaling)`**を設定する必要があります。

- **コアアイデア:** メトリックを監視し、このメトリックの変化に基づいてServiceの`desiredCount`を自動的に調整します。
- **最も一般的なメトリック:**
  - **CPU使用率:** 「シェフの平均ビジネスレベルが70%を超えた場合、スタッフを追加する。」これが最も一般的なメトリックです。
  - **メモリ使用率:** 「キッチンの平均スペース占有率が75%を超えた場合、キッチンを拡張する(Tasksを追加)。」
  - **ALB Request Count Per Target:** これは実際のユーザープレッシャーを最もよく反映するメトリックです。「各提供オープニングが毎分処理する必要がある平均注文数が500を超えた場合、すぐに提供オープニングを追加する。」
- **しきい値を設定:** この`ALBRequestCountPerTarget`メトリックを500前後に保ちたい。
- **境界を設定:**
  - `min_capacity = 2`: 何があっても、キッチンには少なくとも2人のシェフが待機している必要があります。
  - `max_capacity = 20`: 最大は20です。それ以上だと、コストが爆発するか、バックエンドデータベースが対応できなくなる可能性があります。

運用フローは大まかに次のとおりです:
- **スケールアウト:** 金曜日の夜7時にトラフィックが急増すると、平均リクエスト数が800に急上昇します。Auto Scalingはこれを検出し、すぐにServiceの`desiredCount`を増やします(例：2->3、3->5...)。ECSは新しいTasksを起動します。新しいTasksは自動的にALBに登録され、トラフィックを共有し、平均リクエスト数が500前後に戻るまで続きます。
- **スケールイン:** 夜10時、お客様が徐々に去ると、平均リクエスト数が100に下がります。Auto Scalingはこれを検出し、徐々に`desiredCount`を減らします。ECSは過剰なTasksを優雅に終了し、リソースを解放してコストを節約します。
- **スケジュール(期間):** 毎週金曜日の夜7時に**固定トラフィック急増**があり、**夜11時**までにはお客様がほとんどいなくなることが既にわかっている場合、`19:00-23:00`期間に固定数のTasks `{ N }`を設定してトラフィックを処理できます。これによりコストをさらに削減できます - 監視にもコストがかかります。

```yaml
# ecs-auto-scaling.yml
Resources:
  ServiceScalingTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    Properties:
      MaxCapacity: !Ref MaxCapacity
      MinCapacity: !Ref MinCapacity
      ResourceId: !Sub "service/${ClusterName}/${ServiceName}"
      RoleARN: !Sub "arn:aws:iam::${AWS::AccountId}:role/aws-service-role/ecs.application-autoscaling.amazonaws.com/AWSServiceRoleForApplicationAutoScaling_ECSService"
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs

  ServiceScalingPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: !Sub "${ServiceName}-scaling-policy"
      PolicyType: TargetTrackingScaling
      ScalingTargetId: !Ref ServiceScalingTarget
      TargetTrackingScalingPolicyConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ECSServiceAverageCPUUtilization
        TargetValue: 70.0
```

## 4. Amazon EKSとKubernetes管理

人的リソースには時として限界があり、味覚は疲れることがあります。時には、より多くの人を引き付けるためにより多くの**ミシュランレストラン**を開くことが必ずしも良い選択とは限りません。おそらく味の問題かもしれないし、予算の問題かもしれません。私たちのサービスがビジネスロジックを拡大し、同じレストランに新しい料理を継続的に追加すると、管理コストと複雑さが徐々に増加します。同時に、新しい店舗を開いた後、誰もが特定の料理(ミシュランシェフのチーズバーガーや食べた後に自動的に爆発するレストランのような)を指し、他のサービスは全く使用されない可能性もあります - これはベンダーの過ちではなく、間違いなくコストの無駄です。

私たちの次の目標が、異なる顧客グループのために数十の異なるブランド(マイクロサービス)を持つレストラングループを開き、GoogleやMicrosoftのモール(マルチクラウド)に支店を開く可能性さえある場合、業界標準の標準化された「グループオペレーティングシステム」が必要です。このシステムがKubernetes(しばしばK8sと略されます)です。

Amazon EKS(Elastic Kubernetes Service)は、この複雑なオペレーティングシステムの最も核心的で頭を悩ませる「グループ本社管理オフィス」(コントロールプレーン)の管理を支援するためにAWSが提供するサービスです。

### 4.1 EKSクラスタ設定 — フードコートの構築

ECS Clusterが独立したレストランのバックキッチンであるなら、EKS Clusterはフードコート全体の会場と基本インフラストラクチャです。最も重要な部分は**「フードコート管理オフィス」(Control Plane)**と**「屋台スペース」(Data Plane / Worker Nodes)**です。

- **Control Plane - 「フードコート管理オフィス」**
  - **概念:** これはKubernetesの頭脳です。指示を受け取り、すべての屋台(アプリケーション)の開店場所をスケジュールし、すべての屋台の運営状況を監視し、広場全体のレイアウトを保存することなどを担当します。この部分は非常に複雑で重要であり、24時間365日高可用性でなければなりません。
  - **EKSの価値:** 私たちはこの管理オフィスを自分で構築する必要も、警備員やITスタッフを雇って維持する必要もありません。AWS EKSは、高可用性で安全な、自動的にアップグレードされる管理オフィスを提供してくれます。これが私たちがEKSに支払う主な理由です。私たちは屋台の運営に集中するだけです。

- **Data Plane / Worker Nodes - 「屋台スペース」**
  - **概念:** これが実際に「調理」が行われる場所、つまり私たちのアプリケーションコンテナが実際に実行されるサーバーです。
  - EKSには主に2つの形式があります：
    - **EC2 Managed Node Groups:** ECSのEC2モードに似ています。広場管理から一列の「標準屋台」(EC2インスタンス)を借ります。屋台の仕様(例：`m5.large`)を選択できますが、屋台の電力、ネットワーク、防火などはAWSが管理します(自動化されたノードのアップグレードと交換など)。これが最も一般的なモードです。
    - **Fargate Profiles:** ECS Fargateモードと同じです。屋台を借りる必要さえありません。管理オフィスに伝えるだけです：「たこ焼きの臨時屋台を開きたいので、1vCPUと2GBのメモリを持つスペースが必要です。」EKSは自動的に広場の片隅に要件を満たすスペースを作成し、終わったらそれを持ち去ります。ステートレスアプリケーションや突然のトラフィック急増に対処する必要があるシナリオに非常に適しています。

**マルチ環境戦略:** 原則は変わりません。`Account-Dev`に`dev-eks-cluster`を、`Account-Prod`に`prod-eks-cluster`を作成します。IaC(Terraform)は依然としてこれらのクラスタ設定を標準化して定義するためのツールです。

```yaml
# eks-cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: app-cluster-${ENVIRONMENT}
  region: us-west-2
  version: "1.28"

nodeGroups:
  - name: worker-nodes
    instanceType: ${INSTANCE_TYPE}
    minSize: ${MIN_SIZE}
    maxSize: ${MAX_SIZE}
    desiredCapacity: ${DESIRED_SIZE}
    privateNetworking: true
    labels:
      environment: ${ENVIRONMENT}
    tags:
      Environment: ${ENVIRONMENT}
      Project: "multi-env-demo"

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-load-balancer-controller
```

### 4.2 Kubernetesアプリケーションデプロイメント

Kubernetesの世界では、「屋台の運営方法」を説明するための標準的な語彙と文書セット(YAMLファイル)があります。これはECSのTask Definitionよりも詳細で強力です。

1. **Pod - 「独立した調理ワークステーション」**
   - **概念:** Kubernetesでデプロイ可能な最小単位。1つまたは複数の密接に結合されたコンテナを含むことができます。
   - **たとえ話:** Podは鍋(Container)だけではありません。鍋の隣の小さな作業台、独立したシンクとコンセント(独立したネットワークIPアドレス)、専用の小さな冷蔵庫(ストレージVolume)も含まれます。Pod内のコンテナはこの小さな環境を共有できます。
   - **重要な特徴:** Podは一時的です。いつでも破壊され再作成される可能性があります。特定のPodの存在に直接依存してはいけません。

2. **Deployment - 「フランチャイズハンドブック」**
   - **概念:** Deploymentは高レベルのコントローラーで、その核心的な責任は、指定された数の同一Pod replicasが実行されていることを確保することです。
   - **たとえ話:** 私たちは「A+フライドチキン」のブランドオーナーです。私たちのDeployment YAMLはフランチャイズハンドブックであり、次のように記載されています：
     - `replicas: 3`: フードコートには常に3つの「A+フライドチキン」調理ワークステーション(Pods)が運営されている必要があります。
     - `template`: ハンドブックには標準ワークステーションの仕様が詳しく記載されています(どのDocker Imageを使用するか、どれだけのCPU/Memoryが必要かなど)。
     - **自己修復:** Deploymentは2つのワークステーションしか実行されていないことを発見した場合、すぐに3つ目を作成します。
     - **ローリングアップデート:** 新しいフレーバーをローンチしたい場合(Docker Imageを更新)、Deploymentは非常にスマートです：まず新しいワークステーションを開設 -> 新しいステーションの準備ができるのを待つ -> 次に古いものを1つ閉鎖します。この段階的な交換により、屋台のサービスが中断されないことが保証されます。

3. **Service - 「屋台の呼び出し番号と内線」**
   - **概念:** Podsは常に作成され破壊されており、そのIPアドレスは常に変化しています。Serviceは、機能的に同一のPodsのグループを表す安定した不変の内部ネットワークエンドポイントを提供します。
   - **たとえ話:** 私たちの3つのフライドチキンワークステーション(Pods)はフードコートの異なる場所にあり、いつでも位置が変わる可能性があります。お客様は追いかけることはできません。したがって、フードコートのサービスデスクで固定の屋台番号「B-52」(Service)を登録します。
   - **仕組み:** フードコート内の他の屋台(例：「Good Drinks」Pod)がフライドチキンを注文したい場合、どのフライドチキンPodの実際の場所も知る必要はありません。内線「B-52」に電話するだけで、Kubernetesネットワークシステムが自動的にリクエストを現在健康なフライドチキンPodに転送します。これによりサービス間通信の安定性が確保され、マイクロサービスアーキテクチャの基礎となります。

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: ${ENVIRONMENT}
  labels:
    app: demo-app
    environment: ${ENVIRONMENT}
spec:
  replicas: ${REPLICA_COUNT}
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
        environment: ${ENVIRONMENT}
    spec:
      containers:
        - name: app
          image: your-registry/app:${IMAGE_TAG}
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: ${ENVIRONMENT}
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: database-secret
                  key: url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: ${ENVIRONMENT}
spec:
  selector:
    app: demo-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

### 4.3 IngressとLoad Balancer設定

屋台が開店し、内部スタッフが互いに注文できるようになりました。最も重要な質問は：外部のお客様はどのように入って私たちの屋台を見つけるのでしょうか？

シンプルで粗野な方法は、各屋台に外部への直接入口を与えることです(Service type LoadBalancer)。これにより、各サービスに独立したAWS Load Balancerが作成されますが、数十のサービスがあると信じられないほど高価になります。

より優雅で経済的なアプローチは**`Ingress`**を使用することです。`Ingress`はクラスタに入る外部トラフィックを管理するAPIオブジェクトです。HTTP/HTTPSベースのルーティングルールを提供します。フードコート全体に1つのメインエントランスしかなく、エントランスに巨大な電子ディレクトリボードがあるようなものです：
- 「A+フライドチキン」へ行くには(`host: food.com, path: /chicken`) -> エリアB、No. 52へお進みください(`chicken-service`に誘導)
- 「C-ピザハウス」へ行くには(`host: food.com, path: /pizza`) -> エリアC、No. 03へお進みください(`pizza-service`に誘導)

私たちの入口(Ingress)システムを構成する2つの主要コンポーネントは：
1. **Ingress Controller - 「エントランスの警備員とガイド」**: クラスタ内で実際に実行されるプログラムです(それ自体もPod)。私たちが定義するIngressルールをリッスンし、実際にメインエントランスのAWS Application Load Balancer(ALB)を設定する責任があります。EKS環境では、最もよく使用されるのはAWS Load Balancer Controllerです。
2. **Ingress Resource - 「私たちが書いたディレクトリマップルール」**: これはYAMLファイルです。このファイルをKubernetesに提出して、Ingress Controllerに私たちが望むルーティングルールを伝えます。

Ingressの大きな利点は、1つのALBを数十または数百のバックエンドサービスのトラフィックエントリポイントとして使用でき、異なるドメイン名またはURLパスを通じてトラフィックを対応するServiceに正確にルーティングできることです。これによりコストが大幅に節約され、外部トラフィックの管理が簡素化されます。

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ${ENVIRONMENT}
  annotations:
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: ${CERTIFICATE_ARN}
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  rules:
    - host: ${ENVIRONMENT}.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

### ECS vs. EKS

今日(?)、マクロ的なクラスタ構築の視点からミクロ的なアプリケーションデプロイメントとトラフィック入口管理の視点まで、Kubernetesのコアパスを完全に歩きました。このシステムは複雑ですが、今日の大規模マイクロサービスアプリケーションを支える骨格です。

最後に、この2つの違いを簡単に比較しましょう：
> ### ECSはよく組織された高級レストランです：
> AWSは私たちのために多くのルールを定義してくれました。私たちは料理(私たちのアプリケーション)に集中するだけです。高い統合性と緩やかな学習曲線があります。

> ### EKS/Kubernetesは活気ある国際フードコートです：
> 非常に強力で、柔軟で、ベンダーニュートラルな「運営基準」のセットを提供します。その全用語と概念のセット(Pod、Deployment、Service、Ingress...)を学ぶ必要がありますが、見返りとして比類のない柔軟性、スケーラビリティ、活気あるエコシステムがあります。

## 5. CI/CDパイプライン統合

<完全なCI/CD自動化実装 - GitHub Actions × CodePipeline × CodeBuild>で既にCI/CDの統合とプロセスの概念について話しましたので、ここではプロセスを簡単に説明し、断片化されたジョブを整理してから、**`ブルー/グリーンデプロイメント`**セクションに移ります。これは<Infrastructure as Code: Terraformベースのインフラストラクチャコード化とバージョン管理>と<完全なCI/CD自動化実装 - GitHub Actions × CodePipeline × CodeBuild>のマルチ環境アプリケーションを組み合わせたものです。

### 5.1 GitHub Actionsワークフロー

復習すると、CI/CDプロセスの最も重要な概念は`安定した固定のビジネスロジック検証と配信プロセスを確立すること`であり、**`既存のビジネスロジックがコード変更によって破壊されるのをアクティブに保護する`**ことです。

コードがコードリポジトリ(例：GitHub)にプッシュされるたびに、パイプラインが自動的に起動します。最新のコードを取得し、コンパイル、ユニットテスト、コード品質スキャンなどの一連の「品質チェック」を実行します。これは食品工場の「品質管理」部分のようなものです。新しい原材料が到着するたびに、品質検査官がすぐにサンプルを採取して、この原材料バッチが問題なく、生産ライン全体を汚染しないことを確保します - CIの目標は**問題をできるだけ早く発見すること**です。すべてのCI品質管理ステップが合格すると、パイプラインは次のステップに進みます：パッケージング(Docker Imageを作成)、リポジトリへのプッシュ(ECR)、次にDev -> Staging -> Prod環境への自動デプロイメントをトリガーします。CDの目標は、リリースを**日常的**、**安定的**、**追跡可能**なイベントにすることです。品質承認された原材料が自動的に生産ラインに送られ、処理、調理、真空パック、最後にコンベアベルトでトラックに直接配送され、主要な店舗に配布されるのと同じです。

```yaml
# .github/workflows/deploy.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to Development
        run: |
          # ECS deployment
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST

  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          # Kubernetes deployment
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Blue-Green deployment
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

そして、<完全なCI/CD自動化実装 - GitHub Actions × CodePipeline × CodeBuild>、<エンタープライズレベルJobモジュール化とクロスドメイン参照>で述べたように、デプロイメントが必要な場合、共通プロセス(Jobs)を抽出してパイプライン化し、現在のブランチの動作に基づいて自動デプロイメントを実行できます。

```yaml
# build
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT
```

```yaml
#  deploy-dev
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to Development
        run: |
          # ECS deployment
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST
```

```yaml
#  deploy-staging
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          # Kubernetes deployment
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging
```

```yaml
#  deploy-prod
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Blue-Green deployment
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

抽出されたプロセスは次のようになります：
```yaml
# .github/workflows/deploy.yml
name: Multi-Environment Deployment

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  - template: jobs/build.yml@templates
  - template: jobs/deploy-dev.yml@templates
  - template: jobs/deploy-staging.yml@templates
    parameters:
      Version: ${{ variables.Version }}
      env: $(env)
      hasPreview: true
```

### 5.2 ブルー/グリーンデプロイメント：モノリシックインスタンス(ECS/EC2)とクラスター(EKS)

#### モノリシックインスタンス(ECS/EC2)のブルー/グリーンデプロイメント
CI/CDプロセスと`Jobs`の独立分割を簡単に復習した後、マルチ環境デプロイメントの実践的応用 - **`ブルー/グリーンデプロイメント`**を理解しましょう。

現在のシステム開発には主に2つの戦略があります。1つは**ローリングアップデート**で、`トラフィックが完全に冷却されるまで削減された後`にデプロイメントを行います。もう1つは`ゼロダウンタイム`リリース戦略 - **ブルー/グリーンデプロイメント**です。ローリングアップデートはF1レースのようなもので、車がピットストップに入り完全に静止した時(`トラフィックがサービスが完全に冷却されるまで削減`)にのみ`迅速な修理とアップデート`が許可されます。ピットストップでは、カウントダウンと競争しなければなりません。ミスがあればピットストップからの退出時間が遅れます。さらに悪いことに、トラックに戻って新しいタイヤに問題があることがわかった場合、車は既に不安定な状態にあり、調整のために再度ピットストップに入らなければなりません - さらに悪い状況は**`高速で走行中のトラック上で緊急修理を実施する`**ことです。

**`ブルー/グリーンデプロイメント`**は、より安全でエレガントな哲学を提供します。私たちは動いているレースカー(`ブルー環境`)で作業しません。サイドトラックで、新しいタイヤを装着した同一の新しい車(`グリーン環境`)を準備します。この新しい車を始動させ、ウォームアップし、すべてのダッシュボードをチェックします。新しい車が完璧であることを100%確信したとき(もちろん、これは理想的な状況です、笑)、徐々にプレッシャーを新しい車に切り替えます。古い車は徐々にサイドで退役します。新しい車に問題があれば、**`即座に切り戻す`**ことができます。

- **実行ステップ:**
  1. **初期状態:** 現在の本番環境がV1であると仮定し、これをブルー環境と呼びます。すべてのユーザートラフィックはApplication Load Balancer(ALB)を通じてブルー環境のTarget Groupに向けられます。
  2. **グリーン環境のデプロイ:**
     - CI/CDパイプラインが起動します。
     - 完全に新しい独立したインフラストラクチャセット(例：新しいECS ServiceまたはK8s Deployment)を作成し、その上にV2バージョンのソフトウェアをデプロイします。これがグリーン環境です。この時点で、グリーン環境は実行されていますが、実際のユーザートラフィックはありません。
  3. **グリーン環境の検証:**
     - これがブルーグリーンデプロイメントの最も重要な利点です。トラフィックを切り替える前に、グリーン環境で完全で分離された自動テストを実行できます。例えば、パイプライン内からグリーン環境の内部エンドポイントにシミュレートリクエストを送信して、そのコア機能が正常でAPIが正しく返されることを検証します。
  4. **トラフィック切り替え(スイッチ):**
     - すべてのテストが合格したとき(通常「手動承認」ステップがあります)、CI/CDパイプラインは単一のアトミック操作を実行します：ALB Listenerのルールを変更して、トラフィックの**カスタムパーセンテージ**(ビジネスニーズによりますが、20%が一般的な比率です)をブルーTarget GroupからグリーンTarget Groupにリダイレクトします。この切り替えはユーザーにとって瞬時であり、サービス中断を感じません。
  5. **スタンバイと廃止:**
     - 切り替え後、グリーン環境が徐々に引き継ぎ、正式に新しい本番環境になります。
     - 古いブルー環境はすぐには破棄されません。「ホットバックアップ」としてスタンバイのままです。V2バージョンが本番稼働後数分または数時間後に監視システムが深刻な問題を検出した場合、「ワンクリックロールバック」を実行できます - つまり、もう一度トラフィック切り替えを実行して、トラフィックをブルー環境に戻し、秒レベルの災害復旧を達成します。
     - 新しいバージョンが一定期間安定して実行されていることを確認した後、CI/CDパイプラインは最終的なクリーンアップ作業を実行し、ブルー環境を破棄してコストを節約します。

AWS CodeDeployはECSおよびEC2と深く統合されており、上記で説明した複雑なブルーグリーンデプロイメントプロセスを自動化できます。簡単な設定を行うだけです。

先ほど説明した**`「IaCで環境状態を管理し、CI/CDでデプロイメントライフサイクルを管理し、環境の違いを段階的に検証する」`**分業と協力モデルは、現在Cloud-Native分野で最も語られ、認識されている**最適実装**および**ゴールドスタンダード**です。

「最適」は「最も簡単」または「最も速く構築できる」ソリューションを意味するのではなく、いくつかの主要で、しばしば矛盾するエンジニアリング目標の間で最もエレガントなバランスを達成することを意味します。まず、**「安定性」**と**「機敏性」**のバランスを取ります。過去、システムを安定させるには変更を減らすことを意味し、デプロイメントサイクルが数ヶ月になる可能性がありました。迅速な反復(アジャイル)を望むことは、しばしば安定性を犠牲にし、頻繁なオンラインエラーにつながりました。現代のクラウドネイティブアプローチはIaCを使用して「安定性」の基礎を保証します。私たちのインフラストラクチャはもはやブラックボックスではありません。すべての変更には記録があり、レビューされ(Code Review)、追跡可能です。これにより、「手動エラー」または「環境の不整合」によって引き起こされるほとんどの安定性問題が排除されます - インフラストラクチャは岩のように堅固になります。CI/CDは「機敏性」のエンジンを提供します。この堅固な基盤の上で、自動化されたパイプラインはアプリケーションデプロイメントを非常に速く低リスクにします。開発者は基盤インフラストラクチャをクラッシュさせる心配なく、1日に数十回デプロイできます。ブルーグリーンデプロイメントなどの戦略は、この高速反復に最高レベルの安全網を提供します。

次に、要求チーム(Require)と運用チーム(Maintain)の間には、しばしば溝があります。要求者は新機能を迅速に提供したいのに対し、保守者は既存機能を安定させたいと考えます。相互に責任を追及し、非効率性が常態です。しかし`「IaCで環境状態を管理し、CI/CDでデプロイメントライフサイクルを管理し、環境の違いを段階的に検証する」`は、**「明確な責任」**と**「チーム協力」**の間でバランスを取ります。明確な境界を通じて、私たちの分業モデルは非常に明確な「契約」を提供します：
- **プラットフォーム/Opsチーム:** そのコアアウトプットは、安定した信頼性の高い標準化されたインフラストラクチャプラットフォーム(Terraformで定義)です。彼らは都市計画者のようなもので、水道、電気、ガスパイプライン、標準化された建物の基礎を構築する責任があります。
- **アプリケーション/Devチーム:** そのコアアウトプットは高品質のアプリケーション(Dockerにカプセル化)と安全で信頼性の高いデプロイメントプロセス(GitHub Actions Workflowで定義)です。彼らは標準化された基礎の上に家を建てる建設チームのようなもので、家自体にのみ集中し、水と電気がどこから来るかを心配する必要はありません。

プラットフォームチームが安定したプラットフォームを提供すると、アプリケーションチームはCI/CDパイプラインを通じて「セルフサービスデプロイメント」を実行でき、毎回プラットフォームチームを煩わせる必要がなく、開発の自律性と効率を大幅に向上させます。

最後に、**「制御可能性」**と**「複雑性」**の間でバランスが取られます。現代のクラウドシステムは確かに非常に複雑で、数百のリソースと設定が関与することがよくあります。学習曲線はありますが、IaCはこの複雑性を管理する唯一の効果的な方法です。広大なクラウドリソースを人間が読め、マシンが実行可能な構造化されたテキストに変換します。ソフトウェアと同様に、インフラストラクチャをレビュー、テスト、モジュール化できます。同時に、CI/CDは複雑性を隠します。ほとんどの開発者にとって、ブルーグリーンデプロイメントの背後にあるALBルール切り替えの複雑な詳細を理解する必要はありません。「コードをメインブランチにマージしてPipelineの『承認』ボタンをクリックすると、新機能が安全にオンラインでデプロイされる」ことを知っているだけです - CI/CDパイプラインは複雑なデプロイメントプロセスをシンプルで信頼性の高い自動プロセスにカプセル化します。

#### クラスター(EKS)のブルー/グリーンデプロイメント
ブルーグリーンデプロイメントの「哲学」とそのECSおよびEC2レベルでの応用を知った今、Kubernetesという「フードコートマネージャー」が、そのツールを使用してこの精密なトラフィック切り替え操作をどのように実行するかを見てみましょう。

ECSとは異なり、Kubernetesには`blue-green-deployment`という既製のリソースはありません。しかし、レゴを組み立てるように、より柔軟な強力な基本コンポーネント(プリミティブと呼ばれます)のシリーズを提供し、AWS CodeDeployよりも柔軟なブルーグリーンデプロイメントプロセスを構築できます。業界で最も主流でKubernetesの設計哲学を体現する2つの実装方法に焦点を当てます：**`Serviceレイヤースイッチング`**と**`Ingressレイヤースイッチング`**。

復習すると、Kubernetesの重要なアイデアは：**`Deployments/Pods(アプリケーションインスタンス)`**は実際に実行されている「屋台」であり、**変更可能**で**一時的**です。そして**`Services/Ingresses(サービスエントランス)`**は内部および外部顧客に提供される「固定屋台番号」または「フードコートエントランス」であり、**安定して不変**である必要があります。K8sブルーグリーンデプロイメントの本質は、「サービスエントランス」を変更せず、それが指す「アプリケーションインスタンス」のみを変更することです。

##### Service Selector スイッチング(最もクラシックなK8sアプローチ)
これは、Kubernetesのコア概念を最もよく体現する方法であり、Serviceのselectorを変更して瞬時にトラフィックを切り替えます。先ほどEKSを議論するときに使用したフードコートの事例によれば、フードコートの「B-52呼び出しデバイス」(Service)自体は動きません。私たちはそのバックエンドシステムで、サービススタッフを「青いユニフォームチーム」から「緑のユニフォームチーム」に瞬時に切り替えるだけです。

説明的ステップ:
1. **初期状態(ブルー環境実行中):**
   - `v1`バージョンPodsを管理する`Deployment-Blue`があります。これらのPodsはすべて`version: blue`のラベルが付けられています。
   ```yaml
   # deployment-blue.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-blue
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           app: myapp
           version: blue # 重要なラベル
       spec:
         containers:
         - name: myapp
           image: my-app:v1
   ```
   - 安定した`Service`があります。その`selector`は`version: blue`ラベルを持つ`Pods`のみを見つけ、トラフィックをそれらに転送します。
   ```yaml
   # service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-service # これは安定した不変のエントリポイント
   spec:
     selector:
       app: myapp
       version: blue # 現在blueを指す
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
   ```
   この時点で、すべてのトラフィックは`v1` Podsに流れます。

2. **グリーン環境のデプロイ:**
   - CI/CDパイプラインは、`v2`バージョン`Pods`を管理する全く新しい`Deployment-Green`を作成し、これらの`Pods`には`version: green`のラベルが付けられます。
   ```yaml
   # deployment-green.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-green
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           app: myapp
           version: green # 重要なラベル
       spec:
         containers:
         - name: myapp
           image: my-app:v2
   ```
   - デプロイメント完了後、v2 Podsはクラスタ内で健全に実行されていますが、`myapp-service` selectorがまだ`version: blue`であるため、グリーン環境は公式トラフィックを全く受け取りません。

3. **グリーン環境の検証:**
   - この時点で、内部専用テストServiceを作成できます。例えば`myapp-green-test-service`、そのselectorは`version: green`を指します。CI/CDパイプラインはこの内部Serviceのアドレスを使用して、グリーン環境で完全な自動テストを実行できます。

4. **トラフィック切り替えの実行(スイッチ):**
   - すべての準備ができたら、CI/CDパイプラインは単一のアトミックコマンドを実行します:
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'
   ```
   - Kubernetesのコアネットワーキングは即座にルーティングルールを更新します。安定した`myapp-service`エントリポイントへのすべての後続トラフィックは、即座に`version: green`を持つ`Pods`に向けられます。切り替え完了!

5. **ロールバックとクリーンアップ:**
   - `v2`バージョンに問題が発生した場合、ロールバック操作も同様にシンプルで高速です:
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'
   ```
   - `v2`が安定して実行されていることを確認した後、CI/CDパイプラインは古いブルー環境を安全に削除できます:
   ```bash
   kubectl delete deployment myapp-blue
   ```

##### Ingressレイヤースイッチング(マイクロサービスとカナリアリリースにより適しています)
この方法はさらに一歩進みます。Serviceに触れず、Ingressルーティングルールを変更します。今回、「青チーム」と「緑チーム」にはそれぞれ独立した内部呼び出しデバイス(`service-blue`と`service-green`)があります。フードコートのメインエントランスのディレクトリボード(Ingress)を変更し、「A+フライドチキン」の方向を「青カウンター」から「緑カウンター」に変更します。

説明的ステップ:
1. **初期状態:**
   - `deployment-blue`と`deployment-green`があります。
   - 2つの`Services`があります:`service-blue`は`version: blue`を持つ`Pods`を指し、`service-green`は`version: green`を持つ`Pods`を指します。
   - `Ingress`リソースがあり、そのルールは`service-blue`を指します。
   ```yaml
   # ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress
   spec:
     rules:
     - host: myapp.example.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: service-blue # blueサービスを指す
               port:
                 number: 80
   ```

2. **トラフィック切り替え:**
   - CI/CDパイプラインはコマンドを実行して`Ingress`リソースを変更し、`backend.service.name`を`service-blue`から`service-green`に変更します。
   ```bash
   # 簡単な例、実際にはkubectl patchまたはapply -fを使用します
   # ingress.yamlのサービス名を変更してから実行
   kubectl apply -f ingress.yaml
   ```
   - Ingress Controller(例：AWS Load Balancer Controller)は変更を検出し、ALBの転送ルールを更新して、トラフィックを`service-green`にルーティングします。

このモデルはカナリアリリースを実装する基礎です。Ingressを設定して、トラフィックの95%を`service-blue`に、5%を`service-green`に送信し、小規模なグレースケール検証を達成できます。また、Serviceレイヤーでの変更が少ないため、「安定したエントリポイント」としての位置づけにより適合し、責任がより明確になります。

私たちが今マスターしたメンタルモデルは、`IaC、コンテナ化、CI/CD、高度なデプロイメント戦略`を組み合わせたもので、単なる一連の技術操作手順ではありません。それは**現代ソフトウェアエンジニアリングの哲学**です。これは、Google、Netflix、Amazonなどのトップテック企業が週に数千回のデプロイメントを実行しながら、高度に安定したシステムを維持できるコアシークレットです。なぜなら、単に技術的問題を解決するのではなく、システム的なエンジニアリング問題を解決するからです。ソフトウェア配信は単にコードを書くだけでなく、テスト、デプロイメント、運用、チーム協力、リスク管理など、一連のリンクも含むことを深く理解しています。

あなたの将来のキャリアにおいて、ツールがどのように進化しても(TerraformからOpenTofu、GitHub ActionsからGitLab CIまで)、`状態とプロセスの分離`、そして`宣言的と命令的な協力`の背後にあるコアアイデアは長期間適用され続け、高品質のソフトウェアシステムを構築するための堅固な基盤となります。ツールは時代とともに進化しますが、設計哲学は論理的推論と弁証法のプロセスであり、ちょうど次のように:
> 荀子、「儒効」：「千曲万化、其道一なり。」

または私が好むように、荘子から:
> 荘子、「天下」：「宗を離れず、天人と謂う。」

## 6. 設定管理とシークレット管理
最後に、今日(?)は、`環境パラメータ`、`認証公開鍵`、`接続ポート`の管理方法について話しましょう。

これまで、私たちの自動化されたフードコートは非常に先進的です：標準化された屋台の青写真(IaC)、標準化された食事キット(Docker)、効率的なマネージャー(ECS/EKS)、完全に自動化された配送パイプライン(CI/CD)があります。

しかし今、私たちは非常に敏感で重要な問題に対処しなければなりません：**各屋台の「独占秘密レシピ」(Secrets)をどこに置くか？**

`Dockerfile`や`Git`リポジトリの壁に「A+フライドチキン」の秘密マリネレシピを書いた紙を貼ることはできませんよね？それはセキュリティ災害になります。したがって、次にMr. Krabsの精神を学び、機密データをしっかりと保持し、「設定」と「シークレット」をアプリケーションコードから分離し、必要なときに正しいアプリケーションに「注入」する必要があります。

**K8sネイティブソリューション — ConfigMap & Secret**
Kubernetesはこの問題を処理するための2つのネイティブリソースを提供します。

1. **ConfigMap - 「公開メニューとお知らせ」**
   - **目的:** 非機密設定データを保存するために使用されます。
   - **たとえ話:** フードコートのお知らせボードを想像してください。あらゆる種類の公開情報が掲示されています：
     - `API_URL: https://api.internal.mycompany.com`(バックエンドAPIのアドレス)
     - `FEATURE_FLAG_NEW_UI: "true"`(新しいUIを有効にする機能フラグ)
     - `LOG_LEVEL: "info"`(ロギングの詳細レベル)
   - **特性:**
     - キーバリューペアで平文データを保存します。
     - 非常に柔軟で、Podsで使用する主な2つの方法があります：
       1. **環境変数として注入:** これが最も一般的な方法です。
       2. **ファイルとしてマウント:** 設定内容が大きいか、複雑な形式(例：`nginx.conf`ファイル)の場合、ConfigMap全体をPodのファイルシステムにマウントできます。
   - **重要なポイント:** ConfigMapは平文であり、機密情報を保存するために絶対に使用してはいけません。

2. **Secret - 「普通の引き出しに鍵をかけた秘密レシピ」**
   - **目的:** 機密データを保存するために使用されます。
   - **たとえ話:** これは屋台マネージャーがオフィスの普通の引き出しに保管している「秘密レシピノート」のようなものです。次のようなことが書かれているかもしれません：
     - `DATABASE_PASSWORD: "mypassword123"`
     - `STRIPE_API_KEY: "sk_live_..."`
   - **特性:**
     - キーバリューペアでデータを保存します。
     - ConfigMapとの最大の違いは、KubernetesがSecretの値を保存するときに、Base64エンコードすることです。
       - **Base64は暗号化ではありません!** 単なるエンコード方法です。エンコードされた文字列を取得した人は、簡単に元のテキストにデコードできます。
       1. **環境変数として注入:** これが最も一般的な方法です。
       2. **ファイルとしてマウント:** 設定内容が大きいか、複雑な形式(例：`nginx.conf`ファイル)の場合、Secret全体をPodのファイルシステムにマウントできます。
   - **重要なポイント:** Kubernetes Secretのセキュリティは主にRBAC(Role-Based Access Control)に依存します。厳格な権限を設定して、特定のPodsまたは管理者のみがこのSecretオブジェクトを「読み取る」権利を持つように指定できます。君子を防ぐが小人を防がず、主に偶発的な情報漏洩を防ぎます。

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ${ENVIRONMENT}
data:
  NODE_ENV: ${ENVIRONMENT}
  LOG_LEVEL: ${LOG_LEVEL}
  API_BASE_URL: ${API_BASE_URL}
---
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
  namespace: ${ENVIRONMENT}
type: Opaque
data:
  url: ${DATABASE_URL_BASE64}
```

しかし、本格的な本番環境では、K8s Secretを使うだけでは不十分です。先ほど述べたように、`君子を防ぐが小人を防がず、主に偶発的な情報漏洩を防ぎます。`秘密レシピを誰でも簡単にこじ開けられる引き出しに鍵をかけるようなものです。私たちはまだ欠けています：
- **保管時の暗号化:** データベース(etcd)内の秘密はエンコードされているだけで、暗号化されていません。
- **監査証跡:** 誰がいつ秘密レシピを覗いたか？(誰かが覗いたか？)知る方法がありません。
- **自動ローテーション:** セキュリティのため、パスワードは定期的に変更する必要があります。手動ローテーションは面倒でエラーが発生しやすいです。

**AWSのトップレベル金庫 — Secrets Manager**
上記の問題を解決するために、クラウドサービスプロバイダーは専用の「シークレット管理サービス」を提供しています。AWSでは、このサービスはAWS Secrets Managerです。

- **目的:** すべての機密シークレットを一元的に保存、管理、監査、ローテーションします。
- **たとえ話:** これはもはや普通の引き出しではなく、銀行レベルのトップレベル金庫です。
- **コアの利点:**
  1. **強力な暗号化:** すべてのシークレットは、保存時にAWS KMS(Key Management Service)を使用して強力に暗号化されます。
  2. **きめ細かいIAM権限制御:** IAM Policiesを使用して、「どのアプリケーション(どのIAM Role)がどのシークレットのどのバージョンのみを読み取れるか」を正確に指定できます。
  3. **完全な監査証跡:** CloudTrailと深く統合されています。シークレットのすべての読み取り、変更、削除が詳細に記録され、セキュリティチームによる監査が可能です。
  4. **自動ローテーション:** これがSecrets Managerの切り札です。RDSなどのサービスと統合して、データベースパスワードの完全自動定期ローテーションを達成でき、アプリケーションはコードを変更したり再起動したりする必要がありません。

```yaml
# terraform/secrets.tf
resource "aws_secretsmanager_secret" "database_url" {
  name = "${var.environment}/database-url"
  description = "Database connection string for ${var.environment}"

  tags = {
    Environment = var.environment
    Project     = "multi-env-demo"
  }
}

resource "aws_secretsmanager_secret_version" "database_url" {
  secret_id     = aws_secretsmanager_secret.database_url.id
  secret_string = jsonencode({
    url = var.database_url
  })
}
```

**ベストプラクティス: K8s PodsがAWS金庫から直接取得させる**
今、K8sアプリケーション(Pods)とAWS金庫(Secrets Manager)があります。問題は、Podが信頼性の低い手動伝達を通じてではなく、金庫から秘密レシピを安全に取得する方法です。

これには重要な「橋渡し」技術が必要です：IAM Roles for Service Accounts(IRSA)、そして重要なツール：AWS Secrets and Configuration Provider(ASCP)。

屋台のシェフ(Pod)に金庫のパスワードを教えないのと同じです。代わりに、このシェフに特別な**「認証された従業員カード」(IAM Roleを持つService Account)**を発行します。シェフが秘密レシピを必要とするとき、このカードを使って銀行の特定のゲート(ASCP)をスワイプします。銀行システムがカードの権限を確認した後、今日必要な秘密レシピのページを専用の鍵付きボックスに自動的かつ一時的に入れます(Podにファイルとしてマウント)。シェフが終わった後、ボックスは破棄されます。

**運用フロー:**
1. **IRSAをセットアップ:**
   - AWS IAMでRoleを作成し、`my-app/db-password` Secretを読み取る権限を与えます。
   - EKSでKubernetes ServiceAccountを作成します。
   - これら2つを「バインド」し、EKSに伝えます：「このServiceAccountを使用するすべてのPodは、そのIAM Roleのアイデンティティを引き受けることができます。」
2. **DeploymentでServiceAccountを指定:**
   - `deployment.yaml`で、`spec.template.spec.serviceAccountName`を作成したServiceAccountとして明示的に指定します。
3. **ASCPを使用してSecretsをマウント:**
   1. まず、EKSクラスタにAWS Secrets and Configuration Providerアドオンをインストールする必要があります。
   2. 次に、`deployment.yaml`で`Volume`を定義し、ASCPに伝えます：
      1. シークレットをマウントしたい。
      2. シークレットの名前は`my-app/db-password`(Secrets Managerの名前)です。
      3. Podの`/etc/secrets/db-password`パスにマウントしてください。
4. **アプリケーションがシークレットを読み取る:**
   - アプリケーションが起動した後、環境変数からパスワードを読み取るのではなく、ローカルファイル`/etc/secrets/db-password`から直接読み取ります。

このソリューションの大きな利点:
- **最高のセキュリティ:** シークレットはKubernetesリソース(YAML、環境変数)で平文で表示されることはありません。AWS Secrets Managerから暗号化されて送信され、Podのメモリファイルシステムに直接マウントされます。
- **ゼロコード結合:** アプリケーションはシークレットがAWSから来ているか他の場所から来ているかを知る必要も気にする必要もありません。固定ファイルパスから読み取るだけです。
- **動的更新:** Secrets Managerでシークレットの値を更新すると、ASCPはPod内のマウントされたファイル内容をほぼリアルタイムで更新でき、アプリケーションは最新の値を動的に読み取ることができます。

最後に、私たちは階層化された設定とシークレット管理戦略を作成しました：
> **ConfigMap:** 公開アプリケーション設定を保存するために使用されます。
> **K8s Secret:** 基本的なシークレット管理ツールとして、主にRBACで保護され、一部のあまり機密ではないシナリオに適しています。
**AWS Secrets Manager + IRSA + ASCP**は、EKS上で高度に機密なシークレットを管理するためのゴールドスタンダードでありベストプラクティスです。エンドツーエンドの暗号化、監査、自動ローテーション機能を提供し、安全で信頼性の高いシステムを構築するために必要な部分です。

今のところここで打ち切りにしましょう。**監視とログ管理**、**セキュリティベストプラクティス**、**災害復旧とバックアップ**の3つのトピックについては、将来の**リリースと運用監視(セキュリティとコンプライアンス)**段階で詳しく説明します。今日(?)主に議論したのは**ソフトウェア開発と継続的インテグレーション - Dev / Staging / Prod マルチ環境ガバナンスとアーキテクチャ戦略**です。そして明日、**ソフトウェア開発と継続的インテグレーション**の最後の内容 - 開発者エクスペリエンス(DX)最適化：内部ツールとトラブルシューティング設計について議論します。