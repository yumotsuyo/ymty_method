# 対象理解モデル

機能一覧を作る前に使用する。図の見た目ではなく、対象機能の境界、処理契機、値・状態の流れを再現する。

## Mermaid例：Bluetoothヘッドセットのボリュームコントロール

仕様書の複数箇所に分散した、入力判断、状態別の対象選択、音量変換、上下限通知、一時保持、永続化、初期化との関係を再構成した例を示す。

```mermaid
flowchart LR
    user["利用者"]
    hfContext["着信音あり／通話中"]
    avContext["ストリーミング中"]
    invalidContext["非該当状態（例: 通話待受）"]
    power["電源OFF遷移"]
    reset["ソフトウェアリセット"]
    init["初期化"]
    multipoint["電源ON時の接続モード設定"]

    subgraph device["Bluetoothヘッドセット"]
        input["VOL操作受付・押下判定"]
        route["動作状態による対象系統選択"]
        change["音量step変換・範囲維持"]
        hf["HF音量 0..15（既定9）"]
        av["AV音量 0..20（既定7）"]
        memory["変更音量のメモリ保持"]
        pskey["HF／AV音量PSKEY"]
        beepJudge["上下限ビープ判定"]
        beep["ビープ出力（音量12）"]
        ignore["操作無視"]
    end

    user -->|"VOL+／VOL- 短押し・長押し"| input
    input -->|"短押し1step、800msで1step、以後300msごと"| route
    input -->|"312.5ms以内の後続操作"| ignore
    input -->|"過負荷時はキーコードを2000ms保持"| route
    hfContext -->|"HF操作を有効化"| route
    avContext -->|"AV操作を有効化"| route
    invalidContext -->|"非該当状態"| ignore
    route -->|"HF系を選択"| change
    route -->|"AV系を選択"| change
    change -->|"0..15内の変更"| hf
    change -->|"0..20内の変更"| av
    hf -->|"変更値、AVは不変"| memory
    av -->|"変更値、HFは不変"| memory
    change -->|"上下限到達／上下限外向き操作"| beepJudge
    beepJudge -->|"短押し、または長押し認識時1回"| beep
    power -->|"メモリ値を反映"| pskey
    memory -->|"電源OFF時の反映元"| pskey
    init -->|"HF=9、AV=7へ復帰"| hf
    init -->|"HF=9、AV=7へ復帰"| av
    reset -->|"HF=9、AV=7へ復帰（要確認）"| pskey
    user -->|"VOLを押しながら電源ON"| multipoint

    classDef target fill:#4fa72d,color:#fff,stroke:#28701a,stroke-width:2px;
    classDef related fill:#fff,color:#111,stroke:#28789a;
    class input,route,change,hf,av,memory,pskey,beepJudge,beep,ignore target;
    class user,hfContext,avContext,invalidContext,power,reset,init,multipoint related;
    style device fill:#9fb2c2,stroke:#28789a,stroke-width:2px
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
