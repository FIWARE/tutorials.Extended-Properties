<h1  align="center">
    <img src="https://fiware.github.io/tutorials.Step-by-Step/img/fiware-farm.png" />
    <img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"/>
    <br/>
    👨‍🌾 👩‍🌾 🐄 🐐 🐑 🐖 🐓 🌻 🥕 🌽
</h1>

## NGSI-LD Property サブクラス

[![FIWARE Core Context Management](https://fiware.github.io/catalogue/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/FIWARE/tutorials.Getting-Started.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

このチュートリアルでは、JSON-LD のキーワード構文トークンについて解説し、[Smart Farm example](https://github.com/FIWARE/tutorials.Getting-Started/tree/NGSI-LD)
のデータを再利用しながら、多言語対応と、好みの列挙名を扱うために NGSI-LD プロパティを拡張するカスタム・プロパティ・
タイプを紹介します。チュートリアルでは全体で [cUrl](https://ec.haxx.se/) コマンドを使用します。

[<img src="https://run.pstmn.io/button.svg" alt="Run In Postman" style="width: 128px; height: 32px;">](https://god.gw.postman.com/run-collection/217860-3b538d21-0f19-4c63-a9d6-e184ef829ca7?action=collection%2Ffork&source=rip_markdown&collection-url=entityId%3D217860-3b538d21-0f19-4c63-a9d6-e184ef829ca7%26entityType%3Dcollection%26workspaceId%3Db6e7fcf4-ff0c-47cb-ada4-e222ddeee5ac)
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/flopezag/tutorials.Multilanguage/tree/develop)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [JSON-LD `@keywords` の理解](#understanding-json-ld-keywords)
    -   [Farm Management Information System (FMIS) 内のエンティティ](#entities-within-a-farm-management-information-system-fmis)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker Engine](#docker-engine-)
-   [起動](#start-up)
    -   [`@context` ファイルの読み取り](#reading-context-files)
-   [NGSI-LD LanguageProperty](#ngsi-ld-languageproperty)
    -   [多言語プロパティの操作](#working-with-multilanguage-properties)
        -   [新しいデータ・エンティティを作成](#creating-a-new-data-entity)
        -   [正規化された形式での多言語データの読み取り](#reading-multilingual-data-in-normalised-format)
        -   [簡略化された形式での多言語データの読み取り](#reading-multilingual-data-in-simplified-format)
        -   [サポートされていない言語がリクエストされた場合のフォールバック](#fallbacks-when-requesting-data-for-an-unsupported-language)
        -   [多言語データのクエリ](#querying-for-multilingual-data)
-   [NGSI-LD VocabProperty](#ngsi-ld-VocabProperty)
    -   [列挙型と代替 `@context` の使用](#enumerations-and-using-an-alternative-context)
-   [次のステップ](#next-steps)

</details>

<a name="understanding-json-ld-keywords"></a>

# JSON-LD `@keywords` の理解

> "Я понять тебя хочу, Темный твой язык учу."<br/> > _"I want to understand you, I am studying your incomprehensible
> language."_
>
> — Alexander Pushkin (Verses, composed during a sleepless night)

[JSON-LD 構文](https://www.w3.org/TR/json-ld/#syntax-tokens-and-keywords) は、表示される JSON の構造を記述する
一連のキーワードを定義しています。**NGSI-LD** は **JSON-LD** の形式的に構造化された_拡張サブセット_にすぎないため、
**NGSI-LD** は JSON-LD が定義するすべての機能に対して、直接的または間接的に相当するものを提供できるはずです。

例として、JSON-LD はエンティティの一意な識別子を示すために `@id` を、エンティティの type を定義するために `@type`
を定義しています。NGSI-LD のコア `@context` はこれをさらに洗練させており、`id`/`@id` と `type`/`@type` は
相互に交換可能なものとして扱われます。

次の構文 (`@` の有無を問わず) は、いずれも NGSI-LD で受け入れられます:

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
}
```

```json
{
    "@id": "urn:ngsi-ld:Building:farm001",
    "@type": "Building",
    "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
}
```

JSON-LD で定義されているキーワードのうち、以下の用語は、JSON-LD データが供給された際にその意味を維持するために
NGSI-LD のコア `@context` 内で使用またはマップされています。

-   `@list` - 順序付けられたデータの集合を表すために使用されます
-   `@json` - 展開できない JSON オブジェクトに関連して使用されます
-   `@language` - 特定の文字列値または文字列配列の言語を指定するために使用されます
-   `@none` - 属性がインデックス化される機能を持たない場合のデフォルトのインデックス値として使用されます
-   `@value` - 特定のプロパティに関連付けられたデータを指定するために使用されます
-   `@vocab` - プロパティと値を展開するために使用されます

リレーションシップに関する文を記述する `@graph` などの他のキーワードも NGSI-LD で受け入れられますが、NGSI-LD
Context Broker によって直接処理されることはありません

例えば、コア `@context` を見ると、GeoProperty 属性 `coordinates` は次のように完全に定義されています:

```json
"coordinates": {
  "@container": "@list",
  "@id": "geojson:coordinates"
}
```

これにより、その配列内の値 (経度、緯度) の順序が常に維持されることが保証されます。

通常のすべての NGSI-LD **Property** (および **GeoProperty**) は `value` を持ち、これは JSON-LD の `@value`
に相当します - つまり、Property の `value` は、その特定のプロパティに関連付けられたデータそのものです。

しかし、NGSI-LD 仕様には最近の更新があり、この原則にさまざまな拡張やサブクラスが導入されました。これにより、
`@value` 以外の JSON-LD キーワードに直接準拠する NGSI-LD プロパティを作成できるようになっています。

-   NGSI-LD **LanguageProperty** は、国際化された文字列の集合を保持し、JSON-LD の `@language` キーワードを
    使用して定義されます。
-   NGSI-LD **VocabProperty** は、ユーザの `@context` 内で URI を値にマッピングするものであり、JSON-LD の
    `@vocab` キーワードを使用して定義されます。

いずれの場合も、結果として得られるペイロードの意味は標準の JSON-LD の定義に従って変化するため、出力される
NGSI-LD は完全に有効な JSON-LD のままです。

<a name="entities-within-a-farm-management-information-system-fmis"></a>

## Farm Management Information System (FMIS) 内のエンティティ

NGSI-LD をベースにした FMIS システム内で、いくつかの拡張された NGSI-LD プロパティを説明するために、以前に定義した
**Building** エンティティ・タイプを変更します。念のため、これは以下のように定義されていました

-   納屋のような building は、現実世界のレンガとモルタルによる建造物です。**Building** エンティティは、次の
    ようなプロパティを持ちます:
    -   building の名前 (例: "The Big Red Barn")
    -   building のカテゴリ (例: "barn")
    -   住所 "Friedrichstraße 44, 10969 Kreuzberg, Berlin"
    -   物理的な位置 (例: _52.5075 N, 13.3903 E_)
    -   充填レベル - building がどの程度満たされているかの度合い
    -   温度 (例: _21 °C_)
    -   building の所有者 (実在の人物) との関連付け
    -   ...等

最初の属性を取り上げると、Property の `name` は、例えば以下のように複数の言語にローカライズできます:

-   英語での **Big Red Barn**
-   ドイツ語での **Große Rote Scheune**
-   日本語での **大きな赤い納屋**

同様に、データ・スペース内のすべての参加者が `category` 内のさまざまな building タイプの列挙のために共通の URI に
合意できたとしても、各自のシステム内部では、これらの列挙をそれぞれのローカライズされた値で表示する必要がある
場合があります。

例えば、FMIS が openstreetmap.org で定義された URI に従っているとします。designated された _"barn"_ という
building は、実際には URI `https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn` によって定義されます。
JSON-LD `@context` は、これを必要に応じて短縮するために使用できます。

ユーザがシステム内部で `category` を `"barn"` と定義したい場合、次の JSON-LD `@context` を使用できます:

```json
{
    "@context": {
        "barn": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn"
    }
}
```

ユーザがシステム内部で `category` を `"scheune"` と定義したい場合、次の JSON-LD `@context` を使用できます:

```json
{
    "@context": {
        "scheune": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn"
    }
}
```

列挙の定義や再定義は、必ずしも言語のローカライゼーションだけの問題ではありません。FMIS が規制上の理由から、
別のコード・リストの値を使用したい場合もあります。例えば、農薬に含まれる成分の名称は法律によって規制されることが
あり、その製品が販売される市場によって必要な名称が異なる場合があります (例: `Water`, `H₂O`, `Hydrogen Hydroxide`,
`Oxygen Dihydride`, `Hydric Acid`)

<a name="architecture"></a>

# アーキテクチャ

デモ・アプリケーションは、準拠した context broker に対して NGSI-LD の呼び出しを送受信します。標準化された
NGSI-LD インタフェースは複数の context broker で利用可能ですが、ここでは1つだけを選択する必要があります -
例えば [Scorpio Broker](https://fiware-orion.readthedocs.io/en/latest/) です。したがって、アプリケーションは
1つの FIWARE コンポーネントのみを使用します。

現在、Orion Context Broker は、保持しているコンテキスト・データの現在の状態、およびサブスクリプションと
レジストレーションに関連する永続的な情報を保持するために、オープンソースの [MongoDB](https://www.mongodb.com/)
テクノロジに依存しています。Scorpio や Stellio などの他の Context Broker は、状態情報に
[PostgreSQL](https://www.postgresql.org/) を使用しています。

データ交換の相互運用性を促進するために、NGSI-LD context broker は、コンテキスト・エンティティ内に保持される
データを定義する [JSON-LD `@context` ファイル](https://json-ld.org/spec/latest/json-ld/#the-context) を明示的に
公開します。これにより、すべてのエンティティ・タイプとすべての属性に対して一意な URI が定義され、NGSI ドメイン外の
他のサービスが自身のデータ構造の名前を選択できるようになります。すべての `@context` ファイルはネットワーク上で
利用可能でなければなりません。今回の場合、チュートリアル・アプリケーションが一連の静的ファイルをホストするために
使用されます。

したがって、アーキテクチャは次の3つの要素で構成されます:

-   [Scorpio Context Broker](https://scorpio.readthedocs.io/) は、
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してリクエストを受信します
-   基盤となる [Postgres](https://www.postgresql.org/) データベース:
    -   Scorpio Context Broker が、データ・エンティティ、サブスクリプション、レジストレーションなどの
        コンテキスト・データ情報を保持するために使用します
-   HTTP **Web-Server** は、システム内のコンテキスト・エンティティを定義する静的な `@context` ファイルを提供します

3つの要素間のすべての相互作用は HTTP リクエストによって開始されるため、各要素をコンテナ化して、公開されたポートから
実行できます。

必要な構成情報は、関連する `scorpio.yml` ファイルの services セクションで確認できます:

```yaml
scorpio:
    labels:
        org.fiware: "tutorial"
    image: quay.io/fiware/scorpio:java-${SCORPIO_VERSION}
    hostname: scorpio
    container_name: fiware-scorpio
    networks:
        - default
    ports:
        - "1026:9090"
    depends_on:
        - postgres
```

```yaml
postgres:
    labels:
        org.fiware: "tutorial"
    image: postgis/postgis
    hostname: postgres
    container_name: db-postgres
    networks:
        - default
    ports:
        - "5432"
    environment:
        POSTGRES_USER: ngb
        POSTGRES_PASSWORD: ngb
        POSTGRES_DB: ngb
    logging:
        driver: none
    volumes:
        - postgres-db:/var/lib/postgresql/data
```

```yaml
ld-context:
    labels:
        org.fiware: "tutorial"
    image: httpd:alpine
    hostname: context
    container_name: fiware-ld-context
    ports:
        - "3004:80"
    volumes:
        - data-models:/usr/local/apache2/htdocs/
    healthcheck:
        test:
            (wget --server-response --spider --quiet  http://context/user-context.jsonld 2>&1 | awk 'NR==1{print
            $$2}'|  grep -q -e "200") || exit 1
```

すべてのコンテナは同じネットワーク上に存在します - Scorpio Context Broker は内部的にはポート `9090`、外部的には
`1026` でリッスンしており、PostGres はデフォルト・ポート `5432` でリッスンし、httpd Web サーバはポート `80` で
`@context` ファイルを提供しています。すべてのコンテナはポートを外部に公開してもいます - これは純粋にチュートリアル
用のアクセスのためであり、cUrl や Postman が同じネットワークの一部でなくてもアクセスできるようにするためです。
コマンドラインでの初期化は特に説明を要さないはずです。

<a name="prerequisites"></a>

# 前提条件

<a name="docker-engine-"></a>

### Docker Engine <img src="https://www.docker.com/favicon.ico" align="left"  height="30" width="30" style="border-right-style:solid; border-right-width:10px; border-color:transparent; background: transparent">

物事を単純にするために、すべてのコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。
**Docker** は、さまざまなコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Docker を Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の手順に
    従ってください
-   Docker を Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従って
    ください
-   Docker を Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAML file](/docker-compose/orionld.yml) は、アプリケーションに必要なサービスを構成するために使用します。つまり、
すべてのコンテナ・サービスは1つのコマンドで起動できます。

Compose V1 は廃止されましたが、Compose V2 がその後継となり、現在ではすべての最新の Docker Desktop バージョンに
統合されています。そのため、docker compose コマンドを実行するために docker engine への拡張機能をインストールする
必要はありません。

<a name="start-up"></a>

# 起動

リポジトリ内で提供される [services](/services) Bash script を実行することにより、コマンドラインからすべての
サービスを初期化できます。以下のコマンドを実行して、リポジトリのクローンを作成し、必要なイメージを作成してください:

```bash
git clone http://github.com/fiware/tutorials.Extended-Properties.git
cd tutorials.Extended-Properties

./services [start]
```

> [!NOTE]
>
> クリーンアップしてやり直す場合は、次のコマンドを実行してください:
>
> ```
> ./services stop
> ```

---

<a name="reading-context-files"></a>

## `@context` ファイルの読み取り

2つの `@context` ファイルが生成され、チュートリアル・アプリケーション上でホストされています。これらは、データ・
スペース内のさまざまな組織によって使用され、内部的には属性や列挙の名前をそれぞれ異なる方法で定義しています。

-   [`ngsi-context.jsonld`](http://localhost:3000/data-models/ngsi-context.jsonld) - この **NGSI-LD** `@context`
    は、context broker にデータを送信したりデータを取得したりする際にすべての属性を定義する役割を果たします。この
    `@context` は、すべての **NGSI-LD** から **NGSI-LD** への相互作用に使用する必要があります。

-   [`alternate-context.jsonld`](http://localhost:3000/data-models/alternate-context.jsonld) は、サードパーティが
    使用するデータ・モデルの属性の代替となる **JSON-LD** 定義です。この場合、すべての属性名と列挙をドイツ語で
    一般的に使用される用語を使って定義したいドイツ語話者の顧客がいるとします。実質的に、彼らの課金アプリケーション
    内部では、属性に対して別の短い名前のセットが使用されています。彼らの `@context` ファイルは、属性名間の
    合意されたマッピングを反映しています。

このチュートリアルで使用される **Building** エンティティの完全なデータ・モデルの説明は、標準の
[Smart Data Models definition](https://github.com/smart-data-models/dataModel.Building/tree/master/Building)
に基づいています。同じモデルの [Swagger Specification](https://petstore.swagger.io/?url=https://smart-data-models.github.io/dataModel.Building/Building/swagger.yaml)
も利用可能であり、完全なアプリケーションでコード・スタブを生成するために使用できます。

<a name="ngsi-ld-languageproperty"></a>

# NGSI-LD LanguageProperty

<a name="working-with-multilanguage-properties"></a>

## 多言語プロパティの操作

エンティティ・データの作成と消費において、異なる言語向けのバリエーションを提供するために文字列をローカライズする
ことが必要になる場合があります。これを行うためには、まず新しいデータ・タイプ `LanguageProperty` を定義する新しい
エンティティ・データを作成し、この属性の値の異なる言語での表現を保持するために (`value` ではなく) サブ属性
`LanguageMap` を使用する必要があります。

この `LanguageMap` は、[IETF RFC 5646](https://www.rfc-editor.org/info/rfc5646) の言語コードを表す JSON 文字列を
キーとする、一連の簡略化されたペアで構成される JSON オブジェクトに相当します。

<a name="creating-a-new-data-entity"></a>

### 新しいデータ・エンティティを作成

この例では、**LanguageProperty** と **VocabProperty** を持つエンティティを作成します。farm **Building**
エンティティを作成し、その `name` を _英語_、_ドイツ語_、_日本語_ の3つの異なる言語で利用できるようにします。
プロセスとしては、以下の情報を含む **POST** リクエストを Broker に送信します:

#### 1️⃣ リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Content-Type: application/ld+json' \
--data-raw '{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "category": {
        "type": "VocabProperty",
        "vocab": ["farm"]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Großer Stern 1",
            "addressRegion": "Berlin",
            "addressLocality": "Tiergarten",
            "postalCode": "10557"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
             "coordinates": [13.3505, 52.5144]
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
          "en": "Victory Farm",
          "de": "Bauernhof von Sieg",
          "ja": "ビクトリーファーム"
        }
    },
    "@context": "http://context/user-context.jsonld"
}'
```

#### レスポンス:

得られるレスポンスは (`Date` の値を除いて) 次のような内容になります:

```console
HTTP/1.1 201 Created
Date: Sat, 16 Dec 2023 08:39:32 GMT
Location: /ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001
Content-Length: 0
```

#### 2️⃣ リクエスト:

この例では、**LanguageProperty** と **VocabProperty** を持つ2つ目のエンティティを作成します。以降の各エンティティ
は、与えられた `type` に対して一意な `id` を持つ必要があります。`languageMap` 内で、`@none` の簡略化されたペアは、
未知の言語に対して表示されるデフォルトのフォールバック値を示すことに注意してください。

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Content-Type: application/json' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d '{
    "id": "urn:ngsi-ld:Building:barn002",
    "type": "Building",
    "category": {
        "type": "VocabProperty",
        "vocab": ["barn"]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Straße des 17. Juni",
            "addressRegion": "Berlin",
            "addressLocality": "Tiergarten",
            "postalCode": "10557"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
     "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
              "coordinates": [13.3698, 52.5163]
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
          "@none": "The Big Red Barn",
          "en": "Big Red Barn",
          "de": "Große Rote Scheune",
          "ja": "大きな赤い納屋"
        }
    }
}'
```

<a name="reading-multilingual-data-in-normalised-format"></a>

### 正規化された形式での多言語データの読み取り

この例では、**LanguageProperty** を正規化された形式で取得します。特定のエンティティ
(`urn:ngsi-ld:Building:farm001`) の `name` を、取得したい言語を指定せずに正規化された形式で取得したい場合は、
次のコマンドを実行します:

#### 3️⃣ リクエスト:

```console
curl -G -X  GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d 'pick=id,type,name'
```

得られるレスポンスは、さまざまな言語で定義されたすべての文字列値を含む `languageMap` 全体です:

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "en": "Victory Farm",
            "de": "Bauernhof von Sieg",
            "ja": "ビクトリーファーム"
        }
    }
}
```

一方、値 (または複数の値) を _ドイツ語_ のみで受け取りたいと決めた場合は、対応するクエリ・パラメータ `lang` を
`de` に指定する必要があります。

#### 4️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'  \
  -d 'pick=id,type,name' \
  -d 'lang=de'
```

この場合、レスポンスには選択された言語の詳細を示す新しいサブ属性 `lang` (`"lang": "de"`) が、対応する _ドイツ語_
の文字列の内容を持つサブ属性 `value` とともに提供されます。このレスポンスでは、`type` の値がすでに _Property_ に
なっており、`LanguageMap` ではなく `value` サブ属性になっていることに注意してください。

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "name": {
        "type": "Property",
        "lang": "de",
        "value": "Bauernhof von Sieg"
    }
}
```

<a name="reading-multilingual-data-in-simplified-format"></a>

### 簡略化された形式での多言語データの読み取り

この例では、**LanguageProperty** をキー・バリュー形式で取得します。レスポンスを簡略化された形式で取得したい場合は、
対応するリクエスト・パラメータ `format` を `simplified` に指定して送信する必要があります:

#### 5️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d 'pick=id,type,name' \
  -d 'format=simplified'
```

`format=simplified` は `option=simplified` として指定したり、エイリアス `keyValues` を使用して指定したりすることも
できることに注意してください。

#### レスポンス:

**Language Property** は `languageMap` 属性内で返されます。

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "name": {
        "languageMap": {
            "en": "Victory Farm",
            "de": "Bauernhof von Sieg",
            "ja": "ビクトリーファーム"
        }
    }
}
```

**英語** に対応する `name` の値だけを取得したい場合は、リクエストに `lang=en` パラメータを含める必要があります。

#### 6️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d 'pick=id,type,name' \
  -d 'format=simplified' \
  -d 'lang=en'
```

#### レスポンス:

この場合、**Language Property** は通常の **Property** として返され、_英語_ の文字列の値のみが返されます。簡略化
された形式では、サブ属性は返されません。

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "name": "Victory Farm"
}
```

<a name="fallbacks-when-requesting-data-for-an-unsupported-language"></a>

### サポートされていない言語がリクエストされた場合のフォールバック

すべての言語が必ずしも `languageMap` 内に存在するわけではありません。サポートされていない言語 (_フランス語_
`lang=fr` など) がリクエストされた場合、context broker は代わりに別の言語でデータを返すよう最善を尽くします。
優先されるデフォルトは `@none` 言語ですが、これが存在しない場合は、他の任意の言語が返されることがあります。

#### 7️⃣ リクエスト:

`urn:ngsi-ld:Building:barn002` について、`lang=fr` パラメータを追加して _フランス語_ でエンティティの名前を返します

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:barn002' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'  \
  -d 'type=Building' \
  -d 'pick=id,type,name' \
  -d 'lang=fr'
```

#### レスポンス:

**フランス語** はこのエンティティでサポートされている言語ではありませんが、(`@none` 属性で示されるように)
デフォルトの代替が存在するため、デフォルトの `@none` の値が返されます。**Language Property** は通常の **Property**
として返され、デフォルトの文字列の値のみが返されます。

```json
{
    "id": "urn:ngsi-ld:Building:barn002",
    "type": "Building",
    "name": {
        "type": "Property",
        "lang": "@none",
        "value": "The Big Red Barn"
    },
    "@context": [
        "http://context/user-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ]
}
```

#### 8️⃣ リクエスト:

`urn:ngsi-ld:Building:farm001` について、`lang=fr` パラメータを追加して _フランス語_ でエンティティの名前を返します

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'  \
  -d 'type=Building' \
  -d 'pick=id,type,name' \
  -d 'lang=fr'
```

#### レスポンス:

_フランス語_ はサポートされている言語ではなく、(`@none` 属性で示されるように) デフォルトの代替も存在しないため、
`"@lang": "en"` サブプロパティで示されるように、セット内の別の値、この場合は **英語** の文字列が返されます。
再び、**Language Property** は通常の **Property** として返され、_英語_ の文字列の値のみが返されます。

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "name": {
        "type": "Property",
        "lang": "en",
        "value": "Victory Farm"
    },
    "@context": [
        "http://context/user-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ]
}
```

<a name="querying-for-multilingual-data"></a>

### 多言語データのクエリ

`LanguageProperties` 内で個々の言語をクエリする際は、標準の Object 属性の角括弧 `[ ]` 記法を使用します。
例えば、_英語_ で名前が `Big Red Barn` と等しい Building を取得したい場合は、以下のようになります。

#### 9️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'  \
  -d 'type=Building' \
  -d 'pick=id,type,name' \
  -d 'q=name[en]==%22Big%20Red%20Barn%22'
```

#### レスポンス:

```json
[
    {
        "id": "urn:ngsi-ld:Building:barn002",
        "type": "Building",
        "name": {
            "type": "LanguageProperty",
            "languageMap": {
                "@none": "The Big Red Barn",
                "en": "Big Red Barn",
                "de": "Große Rote Scheune",
                "ja": "大きな赤い納屋"
            }
        },
        "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    }
]
```

次に、_任意の_ 言語で `Big Red Barn` に対応するレスポンスを受け取りたいとします。アスタリスク構文 `*` を使用して、
利用可能なすべての言語のデータを確認します。

#### 1️⃣0️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'  \
  -d 'type=Building' \
  -d 'pick=id,type,name' \
  -d 'q=name[*]==%22Big%20Red%20Barn%22'
```

#### レスポンス:

```json
[
    {
        "id": "urn:ngsi-ld:Building:barn002",
        "type": "Building",
        "name": {
            "type": "LanguageProperty",
            "languageMap": {
                "@none": "The Big Red Barn",
                "en": "Big Red Barn",
                "de": "Große Rote Scheune",
                "ja": "大きな赤い納屋"
            }
        },
        "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    }
]
```

<a name="ngsi-ld-VocabProperty"></a>

# NGSI-LD VocabProperty

<a name="enumerations-and-using-an-alternative-context"></a>

## 列挙型と代替 `@context` の使用

ユーザの `@context` は、URN をマッピングし、システム内に保持されているエンティティを定義するためのメカニズムです。
そのため、属性に対して異なる短縮名のセットを使用して、また **VocabProperty** の場合には属性自体の値に対しても
異なる短縮名を使用して、_同じデータ_ を取得することが可能です。これは、エンド・ユーザが別の参加者の context broker
に保持されているデータを完全に制御できない可能性がある、分散データ、フェデレーション、データ・スペースを
扱う際に特に有用です。

**Building** エンティティを作成する際、`ngsi-context.jsonld` という `@context` ファイルを使用しました。
`ngsi-context.jsonld` ファイル内では、すでに以下のように多くの用語がマップされています:

```json
{
    "@context": {
        "type": "@type",
        "id": "@id",
        "ngsi-ld": "https://uri.etsi.org/ngsi-ld/",
        "fiware": "https://uri.fiware.org/ns/dataModels#",
        "Building": "fiware:Building",
        "barn": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn",
        "category": "fiware:category",
        "farm": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dfarm"
    }
}
```

これは、ユーザ `@context` を追加せずにリクエストを行うことで証明できるように、内部的には `category` に対して
長い URI が使用されていることを意味します。

#### 1️⃣1️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Accept: application/ld+json'  \
  -d 'type=https://smartdatamodels.org/dataModel.Building/Building' \
  -d 'pick=id,type,https://smartdatamodels.org/dataModel.Building/category'
```

#### レスポンス:

ご覧のとおり、2つの Building エンティティが、すべての属性に対して長い名前で、また `vocab` の属性値についても
長い名前で返されます。コア・コンテキストで定義されている用語 (`id`、`type`、`vocab`、`VocabProperty` など) は、
コア・コンテキストがデフォルトとして暗黙的に適用されるため、展開されません。

```json
[
    {
        "id": "urn:ngsi-ld:Building:farm001",
        "type": "https://uri.fiware.org/ns/dataModels#Building",
        "https://uri.fiware.org/ns/dataModels#category": {
            "type": "VocabProperty",
            "vocab": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dfarm"
        },
        "@context": ["https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"]
    },
    {
        "id": "urn:ngsi-ld:Building:barn002",
        "type": "https://uri.fiware.org/ns/dataModels#Building",
        "https://uri.fiware.org/ns/dataModels#category": {
            "type": "VocabProperty",
            "vocab": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn"
        },
        "@context": ["https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"]
    }
]
```

#### 1️⃣2️⃣ リクエスト:

`ngsi-context.jsonld` `@context` がリクエストの `Link` ヘッダに含まれている場合、レスポンスはすべての属性名を
短縮名に変換し、**VocabProperty** の場合は値についても短縮名を使用します。

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Accept: application/ld+json'  \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d 'type=Building' \
  -d 'pick=id,type,category'
```

#### レスポンス:

レスポンスでは、カテゴリ `farm` と `barn` が使用されます。

```json
[
    {
        "id": "urn:ngsi-ld:Building:farm001",
        "type": "Building",
        "category": {
            "type": "VocabProperty",
            "vocab": "farm"
        },
        "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    },
    {
        "id": "urn:ngsi-ld:Building:barn002",
        "type": "Building",
        "category": {
            "type": "VocabProperty",
            "vocab": "barn"
        },
        "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    }
]
```

`alternate-context.jsonld` `@context` ファイルは、以下のようにすべての用語と列挙をドイツ語の名前にマップします:

```json
{
    "@context": {
        "type": "@type",
        "id": "@id",
        "ngsi-ld": "https://uri.etsi.org/ngsi-ld/",
        "fiware": "https://uri.fiware.org/ns/dataModels#",
        "Gebäude": "fiware:Building",
        "scheune": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn",
        "kategorie": "fiware:category",
        "bauernhof": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dfarm"
    }
}
```

#### 1️⃣3️⃣ リクエスト:

`alternate-context.jsonld` がリクエストの `Link` ヘッダに含まれている場合、レスポンスはすべての属性名を
`alternate-context.jsonld` で使用されている短縮名に変換し、**VocabProperty** の場合は値についても短縮名を
返します。

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Accept: application/ld+json'  \
  -H 'Link: <http://context/alternate-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d 'type=Geb%C3%A4ude' \
  -d 'pick=id,type,kategorie'
```

#### レスポンス:

レスポンスでは、category 属性は `kategorie` に名前が変更され、値には `bauernhof` と `scheune` が使用されます。
エンティティの `type` の短縮名も変更されています。

```json
[
    {
        "id": "urn:ngsi-ld:Building:farm001",
        "type": "Gebäude",
        "kategorie": {
            "type": "VocabProperty",
            "vocab": "bauernhof"
        },
        "@context": [
            "http://context/alternate-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    },
    {
        "id": "urn:ngsi-ld:Building:barn002",
        "type": "Gebäude",
        "kategorie": {
            "type": "VocabProperty",
            "vocab": "scheune"
        },
        "@context": [
            "http://context/alternate-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    }
]
```

#### 1️⃣4️⃣ リクエスト:

キー・バリューまたは簡略化されたリクエストを行うには、`format=simplified'` パラメータを含めます

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:barn002' \
  -H 'Accept: application/ld+json'  \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d 'pick=id,type,category' \
  -d 'format=simplified'
```

#### レスポンス:

簡略化されたレスポンスでは `vocab` 属性が保持されます (これは、`category` 属性の右辺を JSON-LD の `@vocab` を
使用して再展開できることを意味します)

```json
{
    "id": "urn:ngsi-ld:Building:barn002",
    "type": "Building",
    "category": {
        "vocab": "barn"
    },
    "@context": [
        "http://context/user-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ]
}
```

#### 1️⃣5️⃣ リクエスト:

`q` パラメータを使用してクエリを行う際は、クエリ内のどの属性が **VocabularyProperties** であるかを示すために
`expandValues` パラメータも含めます

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Accept: application/ld+json'  \
  -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/json"' \
  -d 'type=Building' \
  -d 'pick=id,type,category' \
  -d 'q=category==%22barn%22' \
  -d 'expandValues=category'
```

#### レスポンス:

```json
[
    {
        "id": "urn:ngsi-ld:Building:barn002",
        "type": "Building",
        "category": {
            "type": "VocabProperty",
            "vocab": "barn"
        },
        "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ]
    }
]
```

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか? このシリーズの
[他のチュートリアル](https://ngsi-ld-tutorials.rtfd.io)を読むことで見つけることができます

---

## License

[MIT](LICENSE) © 2020-2026 FIWARE Foundation e.V.
