### Mizar 新機能仕様案：Algorithm & Procedural Tactics

#### 1. 基本コンセプト：言語の統一 (Unified Language)

* **タクティクスとアルゴリズムの統合**:
証明を助けるためのスクリプト（Tactics）と、数学的な検証対象（Algorithm）を別々の言語にせず、**「実行可能なMizarコード」** として統一する。
* **リフレクション (Reflection)**:
定義・検証されたアルゴリズムは、証明中に `by computation(Algo)` として呼び出し、その計算結果を定理として利用できる。

#### 2. 構文体系 (Procedural Syntax)

「教科書の擬似コード」のように書ける、手続き的な構文を導入する。

* **定義ブロック (`algorithm`)**:
`definition` ブロック内で `algorithm Name(args) -> Type` として定義。
* **定数・変数の宣言 (`let`, `var`)**:
    * `let` で宣言した変数は**不変（Immutable）**。キャプチャ機能として利用される。
    * `var` で宣言した変数は**可変（Mutable）**となり、再代入（`:=`）が可能。


* **制御構造**:
    * `if ... then ... end`
    * `while ... do ... end`
    * `return val`


* **パターンマッチ**:
    * 複雑な優先順位ルールは排し、**AST構造に対する単純な「上から順（First-match）」** のマッチングを採用。
    * 表記の揺らぎはパーサやマッチングで解決せず、`synonym` 等の定義機能で吸収する。



#### 3. 検証システム (Verification System)

ホーア論理に基づく検証を行うが、証明の汚れを防ぐ工夫を取り入れる。

* **契約 (Contract)**:
ヘッダに `requires` (事前条件) と `ensures` (事後条件) を記述。
* **不変条件 (`invariant`)**:
`while` ループの先頭に必須。検証器はこれを帰納法の仮定として扱う。
* **アサーション (`assert`)**:
計算の途中経過を確認するために任意の位置で使用可能。
* **正当化 (Justification) の階層**:
1. **自動展開 (Auto-expansion)**: 代入や分岐は検証器が自動追跡。
2. **参照 (Reference)**: `invariant P by Lemma(x, y)` のように、外部で証明した定理を引数付きで参照することを推奨（コードを汚さないため）。
3. **インライン証明 (Inline Proof)**: 自動証明できない場合、その場に `proof ... end` を書いて手動証明することを許容（エスケープハッチ）。



#### 4. 拡張機能 (Extensions)

* **Forループ (`for`)**:
    * `for i = 1 to n do ... end` を導入。
    * 機能的には `while` の構文糖衣だが、**「停止性が自明（Termination check skip）」** である点において、Mizarでの実用性が極めて高い。


* **ドメイン特化機能 (Probabilistic/Quantum)**:
    * 構文拡張ではなく、**「ライブラリと属性（Attribute）」** で対応。
    * 例: 量子操作は `Unitary` 属性を持つ関数の呼び出しとして記述し、検証器がその意味論（ユニタリ変換）を適用する。



---
#### コードイメージ

```mizar
definition
  :: アルゴリズム定義
  let a, b be Integer;
  algorithm GCD(a, b) -> Integer
    requires a > 0 & b > 0
    ensures result = a gcd b
  do
    var x = a as Integer;
    var y = b as Integer;

    :: ループ処理
    while y <> 0 do
      :: 不変条件 (外部定理 LoopInvariant を参照)
      Inv:
      invariant x gcd y = a gcd b by LoopInvariant(x, y);

      const r = x mod y as Integer;
      x := y;
      y := r;
    end;

    return x;
  end;

  :: アルゴリズム全体の正当性証明 (事後条件の検証)
  correctness
  proof
    thus result = a gcd b by Inv;
  end;
end;
```