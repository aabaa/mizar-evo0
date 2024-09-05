<!-- markdownlint-disable MD022 MD024 MD031 MD032 MD040 -->
# Mizar Specification - Grammar
## Import Directive
1. Namespace and Directory Structure:  
    The default namespace aligns with the directory structure. For example, the file ```/math/algebra/group.miz``` belongs to the ```math.algebra``` namespace. Therefore, if ```Group``` is defined in ```/math/algebra/group.miz```, then it can be specified as ```math.algebra.Group```.
2. Basic Import Syntax:  
    ```
    import math.algebra.group;
    ```
    This syntax imports all elements in ```/math/algebra/group.miz``` and add them to ```math.algebra``` namespace. To import multiple files, the following syntax can be used:
    ```
    import math.algebra.{group, ring};
    ```
3. Importing with Aliases:  
    ```
    import math.algebra.group into alg;
    ```
    This syntax imports elements in ```/math/algebra/group.miz``` and add them to the ```alg``` namespace. The string after ```into``` may include a period to specify nested namespaces:
    ```
    import math.algebra.group into math.alg;
    ```
    Then ```math.algebra.Group``` element can be specified as ```math.alg.Group```.
4. ```from ... import``` Syntax:  
    ```
    from math.algebra import group, ring;
    ```
    This syntax imports elements from ```/math/algebra/group.miz``` and ```/math/algebra/ring.miz```, and adds them to the root namespace. This syntax can be used with ```into``` keyword:
    ```
    from math.algebra import group, ring into alg;
    ```
    This syntax imports elements from ```/math/algebra/group.miz``` and ```/math/algebra/ring.miz```, and adds them to the ```alg``` namespace.
5. Importing All Files in a Directory:  
    ```
    from math.algebra import *;
    ```
    or
    ```
    import math.algebra.*;
    ```
    These syntaxes import all .miz files from the ```/math/algebra/``` directory.
6. Recursive Import:  
    ```
    from math.algebra import **;
    ```
    or
    ```
    import math.algebra.**;
    ```
    These syntaxes import all .miz files under the ```/math/algebra/``` directory recursively.
7. Re-exporting:  
    ```
    :: in field.miz
    from math.algebra export group,ring;
    ```
    This syntax re-exports elements imported from other .miz files through the current file. For example, if ```Group``` element is defined in ```/math/algebra/group.miz```. Then, if ```Group``` is re-exported in ```field.miz``` as above, importing  ```field.miz``` (e.g., ```import math.algebra.field;``` ) will also add ```Group``` to the ```math.algebra``` namespace without explicitly importing ```/math/algebra/group.miz```.
8. Privacy Control:  
    Prefixing a filename with an underscore creates a private file. For example, ```_internal_helpers.miz``` becomes a private file that can only be imported by files in the same directory.
