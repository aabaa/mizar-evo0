# Sample Codes

## Directory structure

```bash
mml/
    function/
    algebra/
        structure/
        group/
        ring/
        field/
        module/
        vector/
    number/
        natural/
        integer/
        rational/
        real/
        complex/
```

## /mml/function

- function.miz

    ```mizar
    definition
      let X,Y be set;
      mode Function of X,Y is quasi_total PartFunc of X,Y;
    end
    ```

- binop.miz

    ```mizar
    from .function thruport *;

    definition
      let S be 1-sorted;
      :: Dot operator is introduced
      mode BinOp of S is Function of [: S.carrier, S.carrier :], S.carrier;
    end
    ```

## /mml/algebra/structure

- sorted.miz

    ```mizar
    from mml.function.{function,binop} thruport *;

    definition
      :: Inherit all functors and attributes from set.
      :: This grammar rule is specialized for set.
      struct 1-sorted extends set
      begin
        field carrier inherits set -> set;
      end
    end

    definition
      struct UnitStr extends 1-sorted
      begin
        field carrier -> set;
        property unit -> Element of the carrier;
      end
    end

    definition
      struct ZeroStr extends UnitStr
      begin
        field carrier -> set;
        property zero inherits unit -> Element of the carrier;
      end
    end

    definition
      struct OneStr extends UnitStr
      begin
        field carrier -> set;
        property one inherits unit -> Element of the carrier;
      end
    end

    definition
      struct ZeroOneStr extends ZeroStr, OneStr
      begin
        field carrier -> set;
        property zero -> Element of the carrier;
        property one -> Element of the carrier;
      end
    end

    definition
      struct 2-sorted extends 1-sorted
      begin
        field carrier -> set;
        field carrier' -> set;
      end
    end
    ```

- magma.miz

    ```mizar
    from .sorted thruport *

    definition
      :: Inherit all functors and attributes from 1-sorted.
      struct Magma extends 1-sorted
      begin
        field carrier -> set;
        field binop -> BinOp of the carrier;
      end
    end

    definition
      struct AddMagma extends Magma
      begin
        field carrier -> set;
        field add inherits binop -> BinOp of the carrier;
      end
    end

    definition
      struct MulMagma extends Magma
      begin
        field carrier -> set;
        field mul inherits binop -> BinOp of the carrier;
      end
    end

    ```

- loopstr.miz

    ```mizar
    from .magma throughport *

    definition
      struct LoopStr extends Magma
      begin
        field carrier -> set;
        field binop -> BinOp of the carrier;
        property unit -> Element of the carrier;
      end

      ::=
        The Mizar analyzer must check the consistency for diamond inheritance.
        It means the both following paths introduce the same fields or properties:
        Path 1: AddLoopStr.add -> LoopStr.binop -> Magma.binop
        Path 2: AddLoopStr.add -> AddMagma.add -> Magma.binop
      =::
      struct AddLoopStr extends LoopStr, AddMagma
      begin
        field carrier -> set;
        field add inherits binop -> BinOp of the carrier;
        property one inherits unit -> Element of the carrier;
      end

      struct MulLoopStr extends LoopStr, MulMagma
      begin
        field carrier -> set;
        field mul inherits binop -> BinOp of the carrier;
        property zero inherits unit -> Element of the carrier;
      end

      struct DoubleLoopStr extends AddLoopStr, MulLoopStr
      begin
        field carrier -> set;
        field add -> BinOp of the carrier;
        field mul -> BinOp of the carrier;
        property zero -> Element of the carrier;
        property one -> Element of the carrier;
      end;
    end

    notation
      let A be AddLoopStr;
      let x,y be Element of AddLoopStr;
      synonym x+y for M.binop(x,y);
    end;

    notation
      let M be MulLoopStr;
      let x,y be Element of MulLoopStr;
      synonym x*y for M.binop(x,y);
    end;

    ```

## /mml/algebra/group

- /mml/algebra/group/Group.miz

    ```mizar
    from ..structure.loopstr throughport *;

    definition
      let M be Magma;

      attr M is associative means
        for x,y,z be Element of M holds
        M.binop(M.binop(x,y),z) = M.binop(x,M.binop(y,z));

      attr M is unital means
        ex e be Element of M st
        for x be Element of M holds
        M.binop(x,e) = x & M.binop(e,x) = x;

      func id. M -> Element of M means
        for x be Element of M holds
        M.binop(x, it) = x & M.binop(it, x) = x;
      existence;
      uniqueness;

      attr M is commutative means
        for x,y be Element of M holds
        M.binop(x,y) = M.binop(y,x);
    end

    definition
      let M be LoopStr;

      redefine attr M is unital means
        (M qua Magma) is unital & M.unit = id. M;

      attr M is invertible means
        for x be Element of M
        ex y be Element of M
        st M.binop(x,y) = M.unit
         & M.binop(y,x) = M.unit;

      mode M is Group means
      M is non empty associative invertible unital;
    end
    ```

## /mml/algebra/ring

- /mml/algebra/ring/Ring.miz

    ```mizar
    from ..group.group throughport *;

    definition
      let R be DoubleLoopStr;

      mode R is Ring means
      (R qua AddLoopStr) is commutative Group &
      (R qua MulLoopStr) is associative unital &
      R.zero <> R.one &
      for x,y,z be Element of R holds
      x * (y+z) = x*y + x*z;
    end

    definition
      let R be Ring;

      ::=
        If `R is commutative` is stated without declaring the following `attr`,
        the analyzer returns an error because it cannot determine
        whether the `commutative` is associated with `AddLoopStr` or `MulLoopStr`.
      =::
      attr R is commutative means
      (R qua MulLoopStr) is commutative;
    end
    ```
