# Mizar 新機能仕様案：Algorithm & Procedural Tactics

## 1. 基本コンセプト：言語の統一 (Unified Language)

* **タクティクスとアルゴリズムの統合**:
証明を助けるためのスクリプト（Tactics）と、数学的な検証対象（Algorithm）を別々の言語にせず、**「実行可能なMizarコード」** として統一する。
* **リフレクション (Reflection)**:
定義・検証されたアルゴリズムは、証明中に `by computation(Algo)` として呼び出し、その計算結果を定理として利用できる。

## 2. 構文体系 (Procedural Syntax)

「教科書の擬似コード」のように書ける、手続き的な構文を導入する。

* **定義ブロック (`algorithm`)**:
`definition` ブロック内で `algorithm Name(args) -> Type` として定義。
* **定数・変数の宣言 (`const`, `var`)**:
    * `const` で宣言した変数は**不変（Immutable）**。キャプチャ機能として利用される。
    * `var` で宣言した変数は**可変（Mutable）**となり、再代入（`:=`）が可能。


* **制御構造**:
    * `if ... do ... else ... end` (elif 節による多分岐をサポート)
    * `while ... do ... end`
    * `return val`


* **パターンマッチ**:
数式処理やタクティクス記述のために、代数的なデータ構造分解（ASTマッチング）を提供する。
  * **構文**: `match 対象 with ... end`
  * **First-Match ルール**:
  記述された順（上から下）にパターンを評価し、最初に構造が一致した `case` ブロックのみを実行する。複雑な優先順位ルールは設けない。
  * **型付きパターン変数**:
  パターン内で使用する変数（メタ変数）は、事前に `let` または `reserve` で型定義されていなければならない。
  * **束縛 (Binding)**: 宣言済みの型と一致する部分構造がマッチした場合、その変数に値が束縛される（不変定数扱い）。
  * **一致 (Equality)**: 特定の定数やシンボル（例: `0`, `X`）は、値が等しい場合のみマッチする。


## 3. 検証システム (Verification System)

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



## 4. 拡張機能 (Extensions)

* **Forループ (`for`)**:
    * `for i = 1 to n do ... end` を導入。
    * 機能的には `while` の構文糖衣だが、**「停止性が自明（Termination check skip）」** である点において、Mizarでの実用性が極めて高い。


* **ドメイン特化機能 (Probabilistic/Quantum)**:
    * 構文拡張ではなく、**「ライブラリと属性（Attribute）」** で対応。
    * 例: 量子操作は `Unitary` 属性を持つ関数の呼び出しとして記述し、検証器がその意味論（ユニタリ変換）を適用する。



---
## コード例 1：数値計算アルゴリズム (GCD)

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

## コード例 2：多項式微分アルゴリズム (パターンマッチ) 

```mizar
definition
  let P, u, v be PolynomExpr;
  let n be Nat;
  let c be Real;

  algorithm Differentiate(P) -> PolynomExpr
  do
    match P with
      :: セミコロン区切り & end締め
      case u + v do
        return Differentiate(u) + Differentiate(v);
      end;

      case u * v do
        const du = Differentiate(u);
        const dv = Differentiate(v);
        return (du * v) + (u * dv);
      end;

      case u ^ n do
        :: if文の導入
        if n = 0 then
          return 0;
        elsif n = 1 then
          return Differentiate(u);
        else
          return n * (u ^ (n - 1)) * Differentiate(u);
        end;
      end;

      case X do
        return 1;
      end;

      case c do
        return 0;
      end;
    end;
  end;
end;
```

## 5 Reflection & Computation (リフレクションと計算)

定義・検証されたアルゴリズムを、証明のための「タクティクス」として利用する仕組み。

### 5.1 計算環境 (`requirements`)

* ディレクティブ `requirements ALGORITHMS;` を記述することで、その環境下でのアルゴリズムの自動評価が有効になる。

### 5.2 暗黙の計算 (Implicit Computation)

* 定数引数（Ground Terms）を与えられたアルゴリズム呼び出しは、検証器によって自動的にその結果値へと簡約（Reduce）される。
* ユーザーは `by computation` を明示する必要はない（オプションとして残す）。

### 5.3 定義の参照 (Implicit Reference)

* アルゴリズム呼び出しを含む命題は、そのアルゴリズムの `ensures` 節を公理として利用できる。

---

## 実装例：タクティクスとしてのアルゴリズム利用

この仕様に基づくと、ユーザー体験は以下のようになります。

### 1. アルゴリズムの定義

```mizar
definition
  let a, b be Integer;

  algorithm GCD(a, b) -> Integer
    requires a > 0 & b > 0
    ensures result = a gcd b  :: この仕様が証明で使われる
  do
    var x = a as Integer;
    var y = b as Integer;

    while y <> 0 do
      Inv:
      invariant x gcd y = a gcd b by LoopInvariant(x, y);
      
      const r = x mod y as Integer;
      x := y;
      y := r;
    end;
    return x;
  end;
  
  correctness
  proof
    thus result = a gcd b by Inv;
  end;
end;

```

### 2. 証明での利用 (リフレクション)

```mizar
:: 計算機能を有効化
requirements ALGORITHMS;

theorem
  12 gcd 18 = 6
proof
  :: Step 1: 定義への展開 (Expansion)
  :: "GCD(12, 18)" は数学的な "12 gcd 18" と等しいことが ensures から自動推論される
  thus 12 gcd 18 = GCD(12, 18)
  
  :: Step 2: 計算の実行 (Execution)
  :: 定数引数なので、検証器が裏でコードを実行し "6" に置き換える
  :: "by computation" は書かなくて良い
      .= 6;
end;

```

### 3. 判定タクティクスとしての利用

```mizar
definition
  let n be Nat;
  :: 素数判定アルゴリズム
  algorithm IsPrime(n) -> Boolean
    ensures result = TRUE iff n is prime
  do ... end;
end;

requirements ALGORITHMS;

theorem
  101 is prime
proof
  :: IsPrime(101) が TRUE になることは自動計算される。
  :: ensures により "TRUE iff 101 is prime" が導かれる。
  :: したがって、この1行だけで証明終了。
  thus 101 is prime by IsPrime(101); 
end;

```
