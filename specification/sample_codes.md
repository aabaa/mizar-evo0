# Sample Codes

## Directory structure

```bash
mml/
    function/
    algebra/
        structure/
    number/
        structure/
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
    import .function;
    import mml.algebra.structure.sorted;

    definition
      let S be 1-sorted;
      mode BinOp of S is Function of [: S.carrier, S.carrier :], S.carrier;
    end
    ```

## /mml/algebra/structure

- sorted.miz

    ```mizar
    import mml.function.function;

    definition
      :: This grammar rule is specialized for set.
      struct 1-sorted where
        field carrier -> set;
      end

      :: Inherit all functors and attributes from set.
      :: `it` means `set` itself.
      :: Sometimes a `field ... inherits ...` syntax requires type conversion.
      :: In the case, you have to prove the consistency of the type conversion
      :: with a `correctness` statement.
      :: `cluster` statements might be useful.
      extend 1-sorted inherits set where
        field carrier inherits it;
      end
    end

    definition
      struct UnitStr extends 1-sorted where
        field carrier -> set;
        property unit -> Element of the carrier;
      end

      :: If some fields or properties of the base structure are not inherited,
      :: the Mizar analyzer will give a error message.
      extend UnitStr inherits 1-sorted where
        field carrier inherits carrier;
      end
    end

    definition
      struct ZeroStr where
        field carrier -> set;
        property zero -> Element of the carrier;
      end

      extend ZeroStr inherits UnitStr where
        field carrier inherits carrier;
        property zero inherits unit;
      end
    end

    definition
      struct OneStr where
        field carrier -> set;
        property one -> Element of the carrier;
      end

      extend OneStr extends UnitStr where
        field carrier inherits carrier;
        property one inherits unit;
      end
    end

    definition
      struct ZeroOneStr where
        field carrier -> set;
        property zero -> Element of the carrier;
        property one -> Element of the carrier;
      end

      extend ZeroOneStr inherits ZeroStr where
        field carrier inherits carrier;
        property zero inherits zero;
      end

      extend ZeroOneStr inherits OneStr where
        field carrier inherits carrier;
        property one inherits one;
      end
    end

    definition
      struct 2-sorted where
        field carrier -> set;
        field carrier' -> set;
      end

      extend 2-sorted inherits 1-sorted where
        field carrier inherits carrier;
      end
    end
    ```

- magma.miz

    ```mizar
    import .sorted;

    definition
      struct Magma where
        field carrier -> set;
        field binop -> BinOp of the carrier;
      end

      extend Magma inherits 1-sorted where
        field carrier inherits carrier;
      end
    end

    definition
      struct AddMagma where
        field carrier -> set;
        field add -> BinOp of the carrier;
      end

      extend AddMagma inherits Magma where
        field carrier inherits carrier;
        field add inherits binop;
      end
    end

    definition
      struct MulMagma where
        field carrier -> set;
        field mul -> BinOp of the carrier;
      end

      extend MulMagma inherits Magma where
        field carrier inherits carrier;
        field mul inherits binop;
      end
    end

    ```

- loopstr.miz

    ```mizar
    import .magma;

    definition
      struct LoopStr where
        field carrier -> set;
        field binop -> BinOp of the carrier;
        property unit -> Element of the carrier;
      end

      extend LoopStr inherits Magma where
        field carrier inherits carrier;
        field binop inherits binop;
      end
      
      ::=
        The Mizar analyzer must check the consistency for diamond inheritance.
        It means the both following paths introduce the same fields or properties:
        Path 1: AddLoopStr.add -> LoopStr.binop -> Magma.binop
        Path 2: AddLoopStr.add -> AddMagma.add -> Magma.binop
      =::
      struct AddLoopStr where
        field carrier -> set;
        field add inherits binop -> BinOp of the carrier;
        property one inherits unit -> Element of the carrier;
      end

      extend AddLoopStr inherits LoopStr where
        field carrier inherits carrier;
        field add inherits binop;
        property one inherits unit;
      end

      extend AddLoopStr inherits AddMagma where
        field carrier inherits carrier;
        field add inherits add;
      end

      struct MulLoopStr where
        field carrier -> set;
        field mul -> BinOp of the carrier;
        property zero -> Element of the carrier;
      end

      extend MulLoopStr inherits LoopStr where
        field carrier inherits carrier;
        field mul inherits binop;
        property zero inherits unit;
      end

      extend MulLoopStr inherits MulMagma where
        field carrier inherits carrier;
        field mul inherits mul;
      end

      struct DoubleLoopStr where
        field carrier -> set;
        field add -> BinOp of the carrier;
        field mul -> BinOp of the carrier;
        property zero -> Element of the carrier;
        property one -> Element of the carrier;
      end;

      extend DoubleLoopStr inherits AddLoopStr where
        field carrier inherits carrier;
        field add inherits add;
        property zero inherits zero;
      end;

      extend DoubleLoopStr inherits MulLoopStr where
        field carrier inherits carrier;
        field mul inherits mul;
        property one inherits one;
      end;
    end

    notation
      let A be AddLoopStr;
      let x,y be Element of AddLoopStr;
      synonym x+y for A.binop(x,y);
    end;

    notation
      let M be MulLoopStr;
      let x,y be Element of MulLoopStr;
      synonym x*y for M.binop(x,y);
    end;

    ```

- group.miz

    ```mizar
    import .loopstr;

    definition
      let M be Magma;

      attr M is associative means
        for x,y,z be Element of M holds
        M.binop(M.binop(x,y),z) = M.binop(x,M.binop(y,z));

      attr M is unital means
        ex e be Element of M st
        for x be Element of M holds
        M.binop(x,e) = x & M.binop(e,x) = x;

      func id. M -> Element of unital M means
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

- ring.miz

    ```mizar
    import .group;

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

- field.miz
- module.miz
- vector.miz

## /mml/number/structure

- natural.miz
- integer.miz
- rational.miz
- real.miz
- complex.miz
