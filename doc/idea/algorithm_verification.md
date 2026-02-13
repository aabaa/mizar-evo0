# Mizar Algorithm Specification

## 1. 基本コンセプト (Concept)

* **言語の統一**: 証明支援（Tactics）と計算対象（Algorithm）を、単一の「実行可能なMizarコード」として統合する。
* **部分的正当性 (Partial Correctness)**:
  * デフォルトでは停止性を保証しない（無限ループを許容する）。
  * 計算可能性（構成的か否か）を定義時には問わない。

* **明示的実行 (Explicit Execution)**:
  * 暗黙の型変換（集合の自動リスト化など）は行わない。
  * 非決定的な操作（Pick）を許容する。

## 2. 構文体系 (Procedural Syntax)

### 2.1 定義と変数

* **定義ブロック (`algorithm`)**:
`definition` ブロック内で `algorithm Name(args) -> Type` として定義。
* **定数・変数の宣言 (`const`, `var`)**:
    * `const` で宣言した変数は**不変（Immutable）**。キャプチャ機能として利用される。
    * `var` で宣言した変数は**可変（Mutable）**となり、再代入（`:=`）が可能。


### 2.2 制御構造 (Control Flow)

構造化プログラミングに加え、実用的なジャンプ命令をサポートする。

* **分岐**:
  * `if ... do ... else ... end`
  * `match ... with case ... do ... end` (パターンマッチ)

* **ループ**:
  * `while ... do ... end`
  * `for i = 1 to n do ... end` (停止性が自明なループ)
    * `i` はループ内でimmutableとし、ループ不変条件を維持する。

* **ジャンプ (Jump Statements)**:
  * **`break`**: 最も内側のループを脱出する。
    * *検証義務*: 脱出時点の状態が、ループ終了後の不変条件と整合すること。
  * **`continue`**: 次の反復へスキップする。
    * *検証義務*: スキップ時点の状態が、ループ不変条件を維持していること。

* **`return val`**: 値を返して関数を終了する。

### 2.3 非決定性操作 (Non-determinism)

* **Pick操作**: `const x = the Element of S;`
  * `S` が空集合でない限り、VMは任意の要素を一つ選択して返す。
  * 実行ごとに結果が異なる可能性があるが、論理的には「 を満たすある 」として正当化される。


## 3. 検証システム (Verification System)

ホーア論理に基づき、コード内に検証ポイントを埋め込むスタイル（Inline Assertion）を採用する。

### 3.1 契約とアサーション

* `requires P`: 事前条件。
* `ensures Q`: 事後条件。`return` 直前の状態で判定される。
* `assert P`: 任意の位置での表明。

### 3.2 ループ検証
* **`invariant P`**: ループ不変条件。`while` ループの先頭に必須。検証器はこれを帰納法の仮定として扱う。
* **`decreasing Variant [wrt Relation]`**: 停止性証明用の変量宣言。`while` ループの先頭に必須。検証器はこれを帰納法の仮定として扱う。
  * **基本形**: `decreasing n` (自然数  が減ることを宣言。標準順序  を使用)
  * **タプル形**: `decreasing m, n` (辞書式順序で減ることを宣言。複雑な再帰用)
  * **一般形**: `decreasing x wrt R` (任意の整礎関係  において  が遷移することを宣言)

## 4. 実行モデルと計算可能性 (Execution & Computability)

Mizarの厳密さとツールの利便性を両立させるため、「遅延評価」と「自己責任」の原則を採用する。

### 4.1 計算可能性 (Computability)

* **定義の自由**: アルゴリズム定義内では、非構成的（計算不能）な関数や無限集合を自由に使用してよい。
* **実行時の制約 (Runtime Check)**:
  * `by computation` 等で実際にVMがコードを実行する際、実装が存在しない操作（例: 無限集合からのPick、実装のない `func` の呼び出し）に遭遇した場合、**「実行時エラー (Runtime Verification Failure)」** となり、証明は失敗する。
  * コンパイルエラーにはしない（定義自体は数学的に正しい可能性があるため）。

### 4.2 停止性 (Termination)

* **デフォルト (Partial)**:
  * `decreasing` 記述がないループや再帰は、停止性を保証しない。
* VMは **Gas Limit（ステップ数制限）** を設け、超過した場合はタイムアウトとして処理を中断する。ユーザーの環境をフリーズさせない。

* **全域性の証明 (Total Correctness)**:
  * ユーザーがそのアルゴリズムを数学的な関数定義（`func`）として昇格させたい場合、停止性を証明する必要がある。

### 4.3 決定性 (Determinism)

* VMの実行結果は、メモリ配置や実装詳細により変動しうる（Pick操作など）。
* 証明における `by computation` は、「その実行パスにおいて条件を満たす値（Witness）が見つかった」ことをもって正当化とする。

---

## コード例：アッカーマン関数（再帰・タプル停止性）

```mizar
definition
  let m, n be Nat;

  :: 再帰的アルゴリズム
  :: デフォルトでは部分的正当性のみ（停止性証明は任意）
  algorithm Ackermann(m, n) -> Nat
  do
    :: 停止性を証明する場合の記述
    :: m, n のペアが辞書式順序で減っていくことを宣言
    decreasing m, n;

    if m = 0 do
      return n + 1;
    end;

    if n = 0 do
      return Ackermann(m - 1, 1);
    end;

    return Ackermann(m - 1, Ackermann(m, n - 1));
  end;
end;
```

## コード例：非決定性探索 (Pick & Break)

```mizar
definition
  let S be finite set;
  
  algorithm FindAnyEven(S) -> Integer
  do
    var current = S as set;
    
    while current <> {} do
      :: 非決定的な選択 (VMが適当に選ぶ)
      const x = the Element of current;
      
      if x mod 2 = 0 do
        return x; :: Break & Return
      end;
      
      current := current \ {x};
    end;
    
    return -1;
  end;
end;
```
