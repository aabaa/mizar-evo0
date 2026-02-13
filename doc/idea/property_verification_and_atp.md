# Property-based Verification via ATP

## 設計方針

Mizar Evo は**最小カーネル**設計を採用する：
- **カーネル**: 証明証明書 (proof certificate) の検証のみ。組み込み推論器は持たない
- **推論**: 全て外部ATP (Vampire, E, CVC5等) に委譲
- **Property**: ATPに渡す公理/宣言としてエンコードし、ATP側の推論を強化

## 1. Property の ATP エンコーディング

### 1.1 TPTP形式 (Vampire, E)

**公理として渡す（汎用）:**
```tptp
fof(comm_union, axiom, ![A, B]: (union(A,B) = union(B,A))).
fof(sym_equiv, axiom, ![A, B]: (equiv(A,B) => equiv(B,A))).
fof(refl_subset, axiom, ![A]: (subset(A,A))).
fof(irrefl_lt, axiom, ![A]: (~less(A,A))).
```

**AC宣言（Vampire/Eネイティブ機能、遥かに効率的）:**
- Vampire: `--ac_reasoning on` で commutativity を組み込み推論として処理
- E-prover: AC完備化をネイティブサポート
- 公理として渡すと探索空間が爆発するが、ネイティブAC処理では項をAC正規化するため桁違いに高速

### 1.2 SMT-LIB形式 (CVC5, Z3)

```smt2
(assert (forall ((a T) (b T)) (= (F a b) (F b a))))  ; commutativity
(assert (forall ((a T)) (P a a)))                       ; reflexivity
```

CVC5/Z3 の合同閉包に公理を追加するだけで等式推論が自動強化。

### 1.3 推奨エンコーディング戦略

| Property | ATP渡し方 | 理由 |
|---|---|---|
| `commutativity` | **AC宣言** (Vampire/E) | AC公理より数百倍高速 |
| `idempotence` | 等式公理 | ネイティブ機能なし |
| `involutiveness` | 等式公理 | ネイティブ機能なし |
| `projectivity` | 等式公理 | ネイティブ機能なし |
| `symmetry` | 含意公理 | 単純だがATPに効果的 |
| `reflexivity` | 全称公理 | ATP等式推論に有効 |
| `asymmetry` | 含意公理 | |
| `connectedness` | 含意公理 | |
| `irreflexivity` | 否定公理 | |

## 2. アーキテクチャ

```
┌────────────────────────────────────┐
│            ユーザー証明              │
│ (proof ... end / by references)    │
└──────────────┬─────────────────────┘
               │
       ┌───────▼───────┐
       │  問題生成器     │  ← property を公理/AC宣言としてエンコード
       │  (Translator)  │  ← sethood → Fraenkel 許可判定（型チェック段階）
       └───────┬───────┘
               │ TPTP / SMT-LIB
       ┌───────▼───────┐
       │  ATP バックエンド │  Vampire / E / CVC5
       │  (外部プロセス)   │
       └───────┬───────┘
               │ 証明証明書
       ┌───────▼───────┐
       │  最小カーネル    │  ← 証明書検証のみ
       │  (Checker)     │  ← 推論は行わない
       └───────────────┘
```

- **問題生成器**: Mizar の `by` justification + property 公理 を TPTP/SMT-LIB に変換
- **ATP**: 推論を実行し証明を発見
- **カーネル**: ATPが返した証明証明書を検証（信頼すべきコードを最小化）

## 3. `sethood` の扱い

`sethood` は推論ではなく**型チェック**レベルで処理可能：
- Fraenkel式 `{ f(x) where x is T : P(x) }` の使用時に T の `sethood` 登録を確認
- 登録なし → コンパイルエラー（ATPに渡す前に拒否）
- これはカーネルの型チェック機能の範囲内

## 4. 証明証明書の形式

ATPが返す証明を最小カーネルで検証するための形式：
- **TSTP** (Vampire/E): 推論ステップの列として返却
- **Proof log** (CVC5): LFSC/Alethe 形式の証明証明書
- カーネルはこれらを独立検証し、ATPを信頼しない

> [!IMPORTANT]
> カーネルがATPの結果を盲目的に信頼しないことが、最小カーネル設計の核心。
> ATPはあくまで「推論ヒント生成器」であり、最終検証はカーネルが行う (de Bruijn criterion)。

## 関連

- Ch.5 §5.8.1: `sethood`
- Ch.9 §9.5.1: predicate properties
- Ch.10 §10.6.1: functor properties
- Ch.16 §16.5.4: `consistency`
- Ch.23 §23.7–23.8: 文書化予定
