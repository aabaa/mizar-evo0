# Grammar
## Environment part
### 1. Directives
**Current**: 
Ten directives are available, which users can configure in detail according to the objects they wish to import from external files. Historically, this specification is a remnant of the days when memory was precious. However, the bunch of types of different directives makes it difficult to understand which articles should be described in which directives, and also increases the time and effort required by the user. 

**Example**:
```
environ

```

**Proposal**: 
Simplify the directive to only a single *import* keyword.

**Usage**:
```
import <filepath>
```

### 2. Context sensitive grammar
**Current**:
Mizar is a context-sensitive grammar for two reasons. The first one is that the valid symbols change depending on the article written in the vocabularies directive. The other is caused by multiple definition functor/mode enabled by the notation directive.

**Example**:
```
```

**Proposal**:
Although it is unavoidable that the valid symbols are determined by the vocabularies directive, the vocabularies directive should be abolished and directives should be unified with the import keyword. Namespaces should be provided as a means of avoiding symbol collisions due to multiple definitions.

**Usage**:
```
```

### 3. .voc file
**Current**:
Symbol names must be added to the .voc file to define new symbols.

**Proposal**:
Eliminate .voc files; automatically recognize a symbol definition as a new token when it is found in Scanner.

### 4. Import order sensitivity
**Current**:
Directives such as notations, expansions, etc. depend on the import order of articles.

**Proposal**:
As in the C++ language, the priority of multiple definition resolution is defined, and if it cannot be resolved, an error is generated.

### 5. Namespace
**Current**:
Because .miz files are managed in a flat structure, it is difficult to grasp what is where.

**Proposal**:
Introduce hierarchical structure and namespaces; use directory structure as namespace, as in Python.

### 6. Chained import / wildcard import
**Current**:
The ability to include multiple miz files at once does not exist.

**Proposal**:
Allow wildcard imports like ```import "algebra/*"```.

### 7. Article name
**Current**:
There is a limit of 8 characters for article names in the MSDOS era.

**Proposal**:
Remove length limitation on article names.

## Main part