# Amazon OpenSearch Serverless による高度な検索
## 目的
- 2025/8/7 に発表された「[Amazon OpenSearch Serverless がハイブリッド検索、AIコネクター、自動化のサポートを追加](https://aws.amazon.com/jp/about-aws/whats-new/2025/08/amazon-opensearch-serverless-ai-connectors-hybrid-search/)」を試すことができる Notebook を提供する

## 環境
- EC2 上の Amazon Linux 2023 に VSCode の Remote Development で接続し、uv と ipykernel で作った Notebook 環境をクライアント、OpenSearch Serverless（以下 AOSS） と Bedrock をバックエンドにします。

## 環境構築
### EC2
- uv は、astral の [Installing uv](https://docs.astral.sh/uv/getting-started/installation/) に記載のとおり。
```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

- Project は、以下のように構築します。
```
uv init aoss-bedrock-workshop
cd aoss-bedorck-workshop
uv venv
uv add ipykernel
uv run ipython kernel install --user --name=aoss-bedrock-workshop
```

- VSCode には、以下の Extension を導入します。
    - [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
    - [Jupyter](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)

### IAM
- 以下の IAM Role に AOSS と Bedrock のアクセスを付与します。
    - クライアントとなる EC2
    - 管理コンソールを操作するユーザー

- AOSS が Bedrock を操作するためのロールを作成します。
    - 信頼するサービス： `ml.opensearchservice.amazonaws.com`
        - [ここ](https://docs.aws.amazon.com/ja_jp/opensearch-service/latest/developerguide/ml-amazon-connector.html) にしか書いてないので、見落とすと途方にくれる
    - ポリシー：InvokeModel ができること

### AOSS
- まず、コレクションの名前だけを決める。
    - 検索コレクション：workshop-search
    - ベクトル検索コレクション：workshop-vector
- 管理コンソールで、OpenSearch＞Serverless＞Security にそれぞれ作る
    - Encryption policy
        - リソース：workshop*　（のようにコレクション名をカバーする）
        - 暗号化：AWS所有キーを使用する（バラバラにするとOCUもバラバラになる）
    - Network policy
        - workshop* のエンドポイントとアクセスを有効にする
        - この workshop だけならパブリックでも可
    - Data access policy
        - プリンシパルは、上の IAM でアクセスできるようにした EC2 とユーザー
        - リソースは、以下の３つ
            - collection：collection/workshop*
            - index：index/workshop*/*
            - model：model/workshop*/*
        - トラップ
            - 画面が小さいと model が見えない。スクロールも出ない
            - １ルールに設定できる許可が２つまでになっているので、ルールを追加しないといけない

### Bedrock
- 以下のモデルが使えない場合は、管理コンソールで Bedrock＞モデルアクセス　で使えるように変更する
    - Amazon Titan Text Embeddings V2

## Workshop
### 元ネタ
- [Amazon OpenSearch Service による検索ワークショップ(日本語版)のご紹介](https://aws.amazon.com/jp/blogs/news/introduction-to-amazon-opensearch-service-workshop-jp/)
    - [Amazon OpenSearch Service 検索ワークショップ](https://catalog.us-east-1.prod.workshops.aws/workshops/26c005b2-b387-454a-b201-9b8f37f92f92/ja-JP)
        - 本当の元ネタの Notebook は、以下のようにダウンロードします。
            ```
            curl "https://static.us-east-1.prod.workshops.aws/public/620273a4-2710-402b-903b-ec23011a465b/static/notebooks.tar.gz" --output notebooks.tar.gz
            ```

### 1-basic-search.ipynb
