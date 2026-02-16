# Architecture: Reasoning Boundary

## Purpose

Mizar Evo における**推論責務の切り分け**を定義する。
どの推論をMizar側（カーネル・型チェッカ）が担当し、どの推論を外部ATP側に委譲するかの境界線を明確にする。

## Context

- [doc/spec/16.theorems_and_proofs.md](../../spec/16.theorems_and_proofs.md) — 証明構文と `by` justification
- [doc/spec/17.clusters_and_registrations.md](../../spec/17.clusters_and_registrations.md) — クラスタ登録
- [doc/idea/property_verification_and_atp.md](../../idea/property_verification_and_atp.md) — 最小カーネル設計の初期アイデア
- Spec Index Ch.23 §23.7–23.8 — Property-based Verification Pipeline / ATP Integration

## Design Decisions

### Mizar側（カーネル / 型チェッカ）が担う推論

<!-- TODO: 各項目の詳細な仕様を記述 -->

- **型チェック**: 型の整合性、サブタイプ関係の検証
- **クラスタ推論**: existential / conditional / functorial registration の適用
- **sethood 判定**: Fraenkel式使用時の sethood 登録確認（コンパイル時拒否）
- **等式展開**: definitional expansion（定義展開）
- **構文糖の脱糖**: 言語構文からコア論理形式への変換

### ATP側に委譲する推論

<!-- TODO: 各項目の詳細な仕様を記述 -->

- **一階述語論理の推論**: `by` justification に対する推論ステップの探索
- **等式推論**: 公理に基づく等式チェーン生成
- **Property活用**: commutativity, symmetry 等の性質を公理/AC宣言として渡す
- **反駁証明**: 仮定の否定から矛盾を導出

### Alternatives Considered

<!-- TODO: 比較テーブルを記述 -->

1. **全て組み込み推論器**（従来のMizar方式）
2. **全てATP委譲**（Sledgehammer方式）
3. **ハイブリッド: 型レベルはカーネル、論理推論はATP**（採用方針）

### Adopted Approach

ハイブリッド方式を採用。根拠:
- カーネルのTCB（信頼すべきコードベース）を最小化
- 型チェックはATPに渡すまでもなくローカルに決定可能
- 論理推論はATPの方が圧倒的に高性能

## Interface Definitions

<!-- TODO: カーネル → ATP への問題生成インターフェースを定義 -->

```
カーネル（型チェック完了）
   ↓ 局所的ゴール + 利用可能な前提の集合
問題生成器（Translator）
   ↓ TPTP / SMT-LIB 形式の問題
ATP バックエンド
   ↓ 証明証明書
カーネル（証明書検証）
```

## Affected Modules

- `doc/design/mizar-kernel/checker.md` — 型チェック・クラスタ推論
- `doc/design/mizar-atp/translator.md` — 問題変換
- `doc/design/mizar-atp/backend.md` — ATP呼び出し
- → [atp_interface_protocol.md](./atp_interface_protocol.md)
- → [atp_backend_integration.md](./atp_backend_integration.md)

## Constraints and Assumptions

- カーネルはATPの結果を盲目的に信頼しない（de Bruijn criterion）
- 証明証明書の独立検証が必須
- ATP非可用時でもカーネル単体で型チェック・クラスタ推論は動作すること
