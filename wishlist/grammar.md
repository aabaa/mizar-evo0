<!-- markdownlint-disable MD024 -->

# Mizar Wishlist - Grammar

## Environment part

### 1. Directive Structure

#### Issues

Ten directives are available, which users can configure in detail according to the objects they wish to import from external files. Historically, this specification is a remnant of the days when memory was precious. However, the bunch of types of different directives makes it difficult to understand which articles should be described in which directives, and also increases the time and effort required by the user.

```text
environ
 vocabularies NUMBERS, ZFMISC_1, ...
 notations TARSKI, XBOOLE_0, ...
 constructors FINSUB_1, NAT_1, ...
 registrations XBOOLE_0, SUBSET_1, ...
 requirements BOOLE, SUBSET, ...
 definitions TARSKI, XBOOLE_0, ...
 equalities FINSEQ_1, LATTICE3, ...
 expansions TARSKI, XBOOLE_0, ...
 theorems RSKI, XBOOLE_0, ...
 schemes XBOOLE_0, NAT_1, ...
```

#### Suggestions

Simplify the directive to only a single *import* keyword. The disadvantages are increased memory usage and verification time, which should be taken care of by the language processor.

```text
import <filepath>
```

#### Remark

[Adam Naumowicz (ISAT2017)](https://link.springer.com/chapter/10.1007/978-3-319-67229-8_15) proposed to aggregate the directives into three types: *vocabularies*, *requirements*, and *imports*, and showed that it is feasible, although there is a performance penalty.

### 2. Context Sensitive Grammar

#### Issues

The Mizar language has a context-sensitive grammar for the following two reasons:

1. The valid symbols change depending on the article written in the vocabularies directive. The behavior of the lexical analyzer changes depending on the articles written in the vocabularies directive.
2. Multiple definitions of *functor/mode/attribute/predicate* are allowed. Currently, if there is ambiguity in the multiple definitions, the last loaded definition in the notation directive is applied. Therefore, overload resolution depends on the import order of articles written in the directive section.

#### Suggestions

1. Although it is unavoidable that the valid symbols are determined by the vocabularies directive, the vocabularies directive should be abolished and directives should be unified with the *import* keyword.
2. As in the C++ language, determine precedence rules for overload resolution and introduce a syntax that allows explicit type specification. Namespace function should also be provided as a means of avoiding symbol collisions due to multiple definitions. If the disambiguation cannot be removed, an error should be generated. In addition, tools such as definition jumps should be developed so that users can quickly check which definitions have been applied.

### 3. VOC File

#### Issues

Symbol names must be added to the .voc file when new symbols are defined. This results in maintaining the same definition in multiple files, which is not appropriate for a modern language specification.

#### Suggestions

Eliminate .voc files; automatically recognize a symbol definition (+ precedence if it is infix operator) as a new token when it is found in the lexical analyzer.

### 4. Namespace

#### Issues

Because .miz files are managed in a flat structure, it is difficult to grasp what is where.

#### Suggestions

Introduce hierarchical structure and namespaces; use directory structure as namespace, as in Python.

### 5. Wildcard Import and Re-export Function

#### Issues

The ability to include multiple miz files at once does not exist. In addition, the lack of a re-export function makes it necessary to write many articles in the directive section.

#### Suggestions

Allow wildcard imports like ```import algebra.*```. In addition, import and re-export are realized together by the following grammar: ```reexport algebra.group```

### 6. Article name

#### Issues

There is a limit of 8 characters for article names in the MS-DOS era.

#### Suggestions

Remove length limitation on article names.

## Main part
