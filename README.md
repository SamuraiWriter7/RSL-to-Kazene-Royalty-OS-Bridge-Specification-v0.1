# RSL-to-Kazene-Royalty-OS-Bridge-Specification-v0.1
Bridge specification connecting RSL 1.0 external licensing with Kazene Royalty OS v3.0 access, trace, and royalty layers.

RSL 1.0 の外部ライセンス表現を、Kazene Royalty OS v3.0 の Access / Trace / Royalty 構造へ接続するためのブリッジ仕様です。  
このリポジトリは、**RSL の外側の許諾・課金** と、**印税OS の内側の寄与ログ・分配** をつなぐための、機械可読な仕様・スキーマ・サンプルを提供します。  
目的は、AI時代の権利処理を **透明性・監査性・還元性** の3点から実装可能な形に落とし込むことです。

---

## Overview

RSL は、AI クローラーや AI 利用に対する外部ライセンス条件を、機械可読な形で提示するための標準です。  
一方、Kazene Royalty OS は、AI 内部での参照・利用・寄与を Trace Layer で記録し、Royalty Layer で分配へ接続する構造です。

このリポジトリでは、両者を競合関係ではなく **階層の異なる補完構造** として扱います。

- **RSL Layer**: 外部アクセスの許諾・課金
- **Access Layer**: 外部ライセンスの内部権限化
- **Trace Layer**: 内部利用ログの記録
- **Royalty Layer**: 寄与ベースの分配計算

つまり、  
**RSL が入口の条件を定め、Kazene Royalty OS が内部の流れを監査・配分する**  
という二層ロイヤリティ基盤を目指しています。

---

## Scope

この仕様が扱うもの:

- RSL 1.0 のライセンス宣言を内部アクセス権限へ正規化すること
- 外部課金イベントを内部寄与分配へ接続すること
- Access / Trace / Royalty の3層を、監査可能なデータ構造として定義すること
- JSON Schema とサンプルデータによって、実装の検証可能性を高めること

この仕様が扱わないもの:

- RSL 本体仕様の変更
- 法的助言
- 最終的な価格モデルの確定
- 実運用での強制執行やプラットフォーム契約の詳細

---

## Design Principles

### 1. Preserve origin
起源情報を失わないこと。  
外部ライセンス文書、内部スナップショット、トレースイベント、課金イベント、分配記録を連結可能に保ちます。

### 2. Separate external fee and internal allocation
外部課金と内部寄与分配を混同しないこと。  
外部で支払われた金額と、内部で計算される Q 使用量は別レイヤーとして保持します。

### 3. Unknown should remain unknown
RSL 標準語彙に存在しない権限は、勝手に `allow` にしません。  
原則は `unknown` とし、必要なら拡張として保持します。

### 4. Extensions must survive
未知の拡張語彙や非標準要素は、破棄せず保持します。  
将来の拡張互換性を確保するためです。

### 5. Auditability first
すべての変換・課金・分配は、監査可能な ID と根拠を持つべきです。  
仕様は「読める」だけでなく、「追える」ことを重視します。

---

## Repository Structure

```text
.
├─ README.md
├─ schema/
│  ├─ rsl-royalty-bridge-v0.1.schema.json
│  ├─ access_snapshot.schema.json
│  ├─ trace_event.schema.json
│  ├─ royalty_settlement.schema.json
│  └─ external_charge_event.schema.json
├─ examples/
│  ├─ rsl-royalty-bridge-v0.1.sample.json
│  ├─ access_snapshot.sample.json
│  ├─ trace_event.sample.json
│  ├─ royalty_settlement.sample.json
│  └─ external_charge_event.sample.json
└─ .github/
   └─ workflows/
      └─ validate-specs.yml

Start Here

初めて読む場合は、以下の順で見るのがおすすめです。

schema/rsl-royalty-bridge-v0.1.schema.json
このリポジトリ全体のブリッジ仕様を検証するメタスキーマです。
examples/rsl-royalty-bridge-v0.1.sample.json
ブリッジ仕様全体のサンプルです。全体像をつかむのに向いています。
schema/access_snapshot.schema.json
RSL のライセンス情報を内部アクセス権限へ落とした結果を表すスキーマです。
schema/trace_event.schema.json
実際の AI 利用ログを表すスキーマです。
schema/external_charge_event.schema.json
外部課金イベントを表すスキーマです。
schema/royalty_settlement.schema.json
内部 Q 計算と外部課金配分を束ねる分配記録のスキーマです。
Core Data Flow

この仕様の基本的な流れは、次の5段階です。

1. Discover

RSL ドキュメントを発見する。
発見元、URL、方式を監査用に記録します。

2. Parse

RSL XML を解析し、content / license / payment などを抽出します。

3. Normalize

RSL の表現を、内部アクセス権限と課金ポリシーへ正規化します。
ここで Access Snapshot が生成されます。

4. Trace

AI が実際に行った参照・検索・要約・応答などを Trace Event として記録します。

5. Settle

外部課金イベントと内部利用ログを照合し、Royalty Settlement を生成します。

Main Objects
Access Snapshot

RSL のライセンス条件を内部利用権限へ落としたスナップショットです。

主な役割:

RSL 原文と内部権限の橋渡し
allow / prohibit / unknown の明示
課金要否の判断
トレースや分配時の参照元になること
Trace Event

AI の実際の利用行為を記録するイベントです。

主な役割:

誰が
どの AI が
どの作品に
どの行為を行い
どの程度利用したか

を、後から追える形で保持することです。

External Charge Event

外部ライセンス基盤や CDN / 決済基盤などで発生した課金イベントです。

主な役割:

RSL ベースの外部課金結果を保存すること
課金タイミングと課金モードを保持すること
内部寄与分配の原資を明示すること
Royalty Settlement

内部利用ログと外部課金を結びつけ、最終的な分配に接続する記録です。

主な役割:

Q 使用量の記録
外部金額の配分根拠の保持
監査可能な provenance の固定
Validation Targets

このリポジトリでは、現在以下の 5組 を自動検証対象としています。

schema/rsl-royalty-bridge-v0.1.schema.json
↔ examples/rsl-royalty-bridge-v0.1.sample.json
schema/access_snapshot.schema.json
↔ examples/access_snapshot.sample.json
schema/trace_event.schema.json
↔ examples/trace_event.sample.json
schema/royalty_settlement.schema.json
↔ examples/royalty_settlement.sample.json
schema/external_charge_event.schema.json
↔ examples/external_charge_event.sample.json
Schema Usage
GitHub Actions での自動検証

このリポジトリでは、.github/workflows/validate-specs.yml により、上記 5組の Schema / Sample を自動検証します。

対象:

rsl-royalty-bridge-v0.1
access_snapshot
trace_event
royalty_settlement
external_charge_event
ローカルでの検証準備

まず jsonschema をインストールします。

python -m pip install --upgrade pip
python -m pip install jsonschema
ローカルで 5組すべてを検証する
python <<'PY'
import json
from pathlib import Path
from jsonschema import Draft202012Validator, FormatChecker

pairs = [
    (
        "schema/rsl-royalty-bridge-v0.1.schema.json",
        "examples/rsl-royalty-bridge-v0.1.sample.json",
    ),
    (
        "schema/access_snapshot.schema.json",
        "examples/access_snapshot.sample.json",
    ),
    (
        "schema/trace_event.schema.json",
        "examples/trace_event.sample.json",
    ),
    (
        "schema/royalty_settlement.schema.json",
        "examples/royalty_settlement.sample.json",
    ),
    (
        "schema/external_charge_event.schema.json",
        "examples/external_charge_event.sample.json",
    ),
]

def load_json(path_str):
    path = Path(path_str)
    with path.open("r", encoding="utf-8") as f:
        return json.load(f)

for schema_path, sample_path in pairs:
    print(f"\n=== Validating {sample_path} against {schema_path} ===")
    schema = load_json(schema_path)
    sample = load_json(sample_path)

    validator = Draft202012Validator(schema, format_checker=FormatChecker())
    errors = sorted(validator.iter_errors(sample), key=lambda e: list(e.path))

    if errors:
        print(f"Validation failed for {sample_path}")
        for err in errors:
            location = ".".join(str(x) for x in err.path) or "<root>"
            print(f"  - {location}: {err.message}")
        raise SystemExit(1)

    print(f"OK: {sample_path}")

print("\nAll schema validations passed.")
PY
個別に見るべきファイル
全体仕様を確認したい場合
schema/rsl-royalty-bridge-v0.1.schema.json
アクセス権限の内部化を見たい場合
schema/access_snapshot.schema.json
AI 利用ログの形を見たい場合
schema/trace_event.schema.json
外部課金イベントを見たい場合
schema/external_charge_event.schema.json
分配結果を見たい場合
schema/royalty_settlement.schema.json
Interpretation Rules
Prohibits override permits

prohibits は permits より優先します。
一見単純ですが、ここが曖昧だと全体が崩れます。

Unknown is safer than accidental allow

明示されていない権限は、原則 unknown とします。
勝手に許可へ倒さないことが重要です。

External fee and Q allocation are different things

外部課金額と内部 Q 使用量は別物です。
外部金額は現金系、Q は内部精算系であり、混ぜると話が濁ります。

Extensions are preserved

未認識の語彙や拡張要素は extensions に退避させ、変換の継続性を保ちます。

Current Status

このリポジトリは v0.1 / proposed 段階です。
つまり、思想としての宣言ではなく、実装へ進むための検証可能な下書き にあたります。

現在含まれているもの:

ブリッジ全体のメタ仕様
主要データオブジェクトの実体スキーマ
各サンプル JSON
GitHub Actions による自動検証

今後の自然な拡張候補:

変換ロジックの参照実装
YAML / XML → JSON 変換ツール
より細かな action taxonomy
分配ロジックの高度化
監査証跡の署名や改ざん検知
Why this repo exists

AI 時代の権利処理は、
「使ったか / 使っていないか」だけでは、もう十分ではありません。

必要なのは、

どの条件でアクセスしたか
内部で何が起きたか
どのように分配へ接続されるか

を、機械可読かつ監査可能な形で扱うこと です。

このリポジトリは、そのための最小構造を提示するものです。
派手さはありませんが、骨組みがないと城は立ちません。

License / Notice

このリポジトリは、RSL 本体仕様そのものではなく、
RSL と Kazene Royalty OS を接続するための独自ブリッジ仕様案 です。

RSL の公式仕様を上書きするものではありません
実際の利用にあたっては法務・実務要件の確認が必要です
本仕様は実装検討・議論・検証の土台として提供されます
Contributing

Issue / Discussion / Pull Request による改善提案を歓迎します。
特に歓迎するのは、以下のような改善です。

変換ルールの明確化
スキーマの過不足の指摘
サンプルデータの改善
監査性・可読性・互換性の向上
実装しやすさの向上

ただし、仕様の核は以下を守る前提です。

起源を消さない
外部課金と内部配分を混同しない
未定義を勝手に許可へ倒さない
監査可能性を損なわない
Final Note

この仕様は、
RSL の外部ライセンス世界 と
Kazene Royalty OS の内部循環世界 を
一本の橋としてつなぐ試みです。

橋は目立たない。
だが、橋がなければ向こう岸へは渡れない。
