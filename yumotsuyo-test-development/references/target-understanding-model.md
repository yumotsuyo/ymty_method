# 対象理解モデル

機能一覧を作る前に使用する。図の見た目ではなく、対象機能の境界、処理契機、値・状態の流れを再現する。

## Mermaidテンプレート

```mermaid
flowchart LR
    registration["新規会員登録"]
    rank["ランク更新"]

    subgraph cartScreen["カート画面"]
        checkout["チェックアウト"]
    end

    subgraph registerScreen["レジ（お支払い、配送）画面"]
        coupon["クーポン利用"]
        pointUse["ポイント利用"]
        order["注文確定"]
    end

    subgraph historyScreen["注文履歴画面"]
        cancel["注文キャンセル"]
    end

    pointGrant["ポイント付与"]
    pointBalance["ポイント残高表示"]

    checkout --> registerScreen
    registration -->|"新規付与"| pointGrant
    pointUse -->|"ポイント割引額決定"| order
    order -->|"付与ポイント数－利用ポイント数"| pointGrant
    cancel -->|"利用ポイント返却"| pointGrant
    pointGrant -->|"ポイント数更新"| pointBalance
    pointGrant -->|"残高増減値"| rank

    classDef target fill:#4fa72d,color:#fff,stroke:#28701a,stroke-width:2px;
    classDef related fill:#fff,color:#111,stroke:#28789a;
    class pointUse,pointGrant,pointBalance target;
    class registration,rank,checkout,coupon,order,cancel related;
    style cartScreen fill:#9fb2c2,stroke:#28789a,stroke-width:2px
    style registerScreen fill:#9fb2c2,stroke:#28789a,stroke-width:2px
    style historyScreen fill:#9fb2c2,stroke:#28789a,stroke-width:2px
```

## 作成規則

- テスト対象候補を緑、画面をグレー、関連機能・処理・外部主体を白にして区別する。
- エッジの向きは処理、値、状態、結果の流れる向きにする。
- エッジへ契機または受け渡す具体物を書く。
- 更新するデータが同じでも契機と責務が異なる処理を無条件に統合しない。
- 図に現れない振る舞い断片を残さない。図へ載せない場合は理由を記録する。
- Mermaidが表示できない環境でもコードを成果物として残す。

## 照合

1. 緑の各ノードに機能一覧の項目があるか。
2. 機能一覧の各項目が図のノードへ戻れるか。
3. 各矢印に根拠となる振る舞い断片があるか。
4. 各断片が少なくとも一つの機能候補へ割り当てられているか。
5. 未割当・重複を解消した後、機能の統合・分割・名称を見直したか。
6. Mermaid図、振る舞い断片と機能候補の紐付け、未割当・重複・要確認をユーザーへ提示し、対象理解モデルの承認を得たか。
7. 承認前に機能一覧の確定、品質リスク分析、TADへ進んでいないか。
