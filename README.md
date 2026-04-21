# 📚 Compiler Design – Chapter 5: Syntax Directed Translation
### Complete Exam-Oriented Notes | UE23CS341B

---

## 🗂️ TABLE OF CONTENTS
1. [Introduction to Semantics](#1-introduction-to-semantics)
2. [Semantic Analysis](#2-semantic-analysis)
3. [Syntax-Directed Translation (SDT) – Overview](#3-syntax-directed-translation-sdt--overview)
4. [Attribute Grammars & Types of Attributes](#4-attribute-grammars--types-of-attributes)
5. [Syntax-Directed Definitions (SDD)](#5-syntax-directed-definitions-sdd)
6. [S-Attributed SDD & Examples](#6-s-attributed-sdd--examples)
7. [Evaluating SDDs – Parse Tree Method](#7-evaluating-sdds--parse-tree-method)
8. [Dependency Graphs & Evaluation Order](#8-dependency-graphs--evaluation-order)
9. [When Are Inherited Attributes Useful?](#9-when-are-inherited-attributes-useful)
10. [L-Attributed SDD & Examples](#10-l-attributed-sdd--examples)
11. [Construction of Syntax Trees (ASTs)](#11-construction-of-syntax-trees-asts)
12. [SDT Schemes – Postfix, Parser-Stack, Actions Inside Productions](#12-sdt-schemes)
13. [Eliminating Left Recursion from SDTs](#13-eliminating-left-recursion-from-sdts)
14. [Intermediate Code Generation (TAC)](#14-intermediate-code-generation-tac)
15. [Bottom-Up Parsing of L-Attributed SDDs](#15-bottom-up-parsing-of-l-attributed-sdds)
16. [Important Keywords & Quick Revision Summary](#16-important-keywords--quick-revision-summary)
17. [Question Bank](#17-question-bank)

---

## 1. Introduction to Semantics

### 🔍 What is Semantics?
**Semantics** = the *meaning* of a program's constructs.

After the parser builds a parse tree (during syntax analysis), that tree tells us the **structure** of the program — but nothing about **what it means**. Semantics adds the meaning.

**Simple Analogy:** Grammar tells you "this sentence is correctly formed." Semantics tells you "does this sentence make sense?" For example, "Colorless green ideas sleep furiously" is grammatically correct English but semantically nonsensical.

### The Problem with Plain Parse Trees
Consider the grammar:
```
E → E + T | T
T → T * F
F → id
```
This grammar says how expressions are structured, but gives **no rule for evaluating** them. There's no way to know what `3 + 5 * 2` equals just from the parse tree alone.

**We need to associate something extra with each grammar symbol and each production.** That "something extra" is called an **attribute**.

---

## 2. Semantic Analysis

### What Does Semantic Analysis Do?
Once the syntactic structure is known, semantic analysis:
- Computes additional meaning-related information
- Adds data to the **symbol table**
- Performs **type checking**
- Judges whether syntax constructs derive any real meaning

### Classic Example of a Semantic Error:
```c
int a = "value";   // Lexically OK ✓, Syntactically OK ✓, Semantically WRONG ✗
```
- Lexical analysis: sees tokens `int`, `a`, `=`, `"value"` — all valid tokens ✓
- Syntax analysis: matches the pattern `type id = literal` — valid structure ✓
- Semantic analysis: **catches the type mismatch!** `int` variable cannot hold a string.

### Types of Semantic Errors:
| Semantic Error | Example |
|---|---|
| Type mismatch | `int x = "hello"` |
| Undeclared variable | using `y` without declaring it |
| Reserved identifier misuse | naming a variable `if` or `while` |
| Multiple declarations in same scope | declaring `int x; int x;` twice |
| Out-of-scope variable access | using a local variable outside its function |
| Formal/actual parameter mismatch | `void f(int x)` called as `f("hello")` |

### What Does Semantic Analysis Produce?
- **Object code** (in single-pass compilers, code generation happens during parsing)
- **Intermediate Representation (IR)**: most commonly an **Abstract Syntax Tree (AST)** or **quadruples (TAC)**

### For Semantic Analysis We Need Two Things:
1. **Syntax-Directed Definitions (SDD)** = CFG + attributes + semantic rules → *Representation Formalism*
2. **Syntax-Directed Translation (SDT)** = CFG + attributes + semantic actions → *Implementation Mechanism*

> 💡 **Memory Trick:** SDD = Specification (what to compute). SDT = Do it (how to execute).

---

## 3. Syntax-Directed Translation (SDT) – Overview

### Definition
**Syntax-Directed Translation** is a method where the translation of the source language is **completely driven by the parser**. The parse tree directs both semantic analysis and translation.

### What It Does:
- Semantic Analysis
- Intermediate code generation (TAC, SSA)
- Infix → Postfix conversion
- Expression evaluation
- Building syntax trees (ASTs)
- etc.

### How It Works:
Our normal CFG is **augmented** (extended) with:
- **Attributes** attached to grammar symbols
- **Semantic rules / actions** associated with productions

Such augmented grammars are called **attribute grammars**.

### SDT vs SDD:
| | SDD | SDT |
|---|---|---|
| Full form | Syntax-Directed Definition | Syntax-Directed Translation |
| What it is | CFG + attributes + semantic **rules** | CFG + program fragments (**actions**) embedded in productions |
| Purpose | Specification (easier to read) | Implementation (more efficient) |
| Order of evaluation | Not specified | Order of execution matters |

**Example – Infix to Postfix:**
- SDD rule: `E → E1 + T` → `E.code = E1.code || T.code || '+'`
- SDT action: `E → E1 + T { print('+'); }`

---

## 4. Attribute Grammars & Types of Attributes

### What is an Attribute?
An **attribute** is a variable attached to a grammar symbol. It holds information (like a value, type, code string, etc.) that is computed during translation.

**Examples of attribute values:** integers, strings, types, pointers to symbol-table entries, code fragments.

### Two Types of Attributes:

---

### 🔼 Synthesized Attributes (Bottom-Up)

**Definition:** A synthesized attribute at node N is defined in terms of:
- Attribute values at **children** of N, and/or
- Attribute values at **N itself**

**Direction of flow: ⬆️ UPWARD** (child → parent)

**Simple Analogy:** Like a sum calculated by adding up children's values and passing the total to the parent.

**Example:**
```
Production: E → E1 + T
Rule:       E.val = E1.val + T.val
```
Here, `E.val` is synthesized from its children `E1.val` and `T.val`.

**Key Properties:**
- Can be associated with both terminals AND non-terminals
- Terminals usually have synthesized attributes supplied by the lexical analyzer (e.g., `digit.lexval`)
- SDDs with ONLY synthesized attributes are called **S-attributed SDDs**
- Can be evaluated during a **single bottom-up traversal** (postorder)

---

### 🔽 Inherited Attributes (Top-Down / Sideways)

**Definition:** An inherited attribute at node N is defined in terms of:
- Attribute values at N's **parent**, and/or
- Attribute values at N's **siblings** (usually left siblings), and/or
- Attribute values at **N itself**

**Direction of flow: ⬇️ DOWNWARD or ↔️ SIDEWAYS** (parent/left-sibling → current node)

**Simple Analogy:** Like a type being passed down to all variables in a declaration list: `int x, y, z` — the type `int` is inherited by `x`, `y`, and `z`.

**Key Properties:**
- Only **non-terminals** can have inherited attributes (terminals CANNOT)
- Used exclusively in **L-attributed SDTs**
- Allow information to flow **downward or sideways**
- Cannot be evaluated by simple preorder traversal (exception: if not dependent on right siblings)

**Critical Difference Table:**

| Property | Synthesized | Inherited |
|---|---|---|
| Flow direction | ⬆️ Bottom-up | ⬇️ Top-down / Sideways |
| Defined in terms of | Children and self | Parent, left siblings, self |
| Terminals can have it? | ✅ Yes | ❌ No |
| Evaluation order | Postorder (easy) | More complex |
| Used in | S-attributed & L-attributed | L-attributed only |

---

## 5. Syntax-Directed Definitions (SDD)

### Definition
An **SDD** is a CFG where:
- **Attributes** are associated with grammar symbols
- **Semantic rules** are associated with productions

### SDD = CFG + Attributes + Semantic Rules

### Example – Type Checking SDD:
```
Production:              Semantic Rule:
Expr → Expr1 + Expr2    Expr.type = if (Expr1.type == Expr2.type)
                                     then Expr1.type
                                     else error("Type mismatch")
Expr → Num              Expr.type = "int"
```

### Important Note:
> Terminal symbols are assumed to have **synthesized attributes supplied by the lexical analyzer**. For example, `digit.lexval` holds the integer value of a digit token.

---

## 6. S-Attributed SDD & Examples

### Definition
An SDD that involves **only synthesized attributes** is called **S-attributed**.

- Each rule computes an attribute for the **head** (left-hand side) of a production from attributes in the **body** (right-hand side).
- Can be evaluated naturally during **bottom-up (LR) parsing**.
- Semantic actions go at the **rightmost end** of production bodies.

### 📊 The Desk Calculator SDD (Figure 5.1 – EXAM FAVORITE!):

```
Production          Semantic Rule
L → E n             L.val = E.val
E → E1 + T          E.val = E1.val + T.val
E → T               E.val = T.val
T → T1 * F          T.val = T1.val * F.val
T → F               T.val = F.val
F → (E)             F.val = E.val
F → digit           F.val = digit.lexval
```
**All attributes here (`val`, `lexval`) are SYNTHESIZED → S-attributed SDD.**

**Corresponding S-attributed SDT (postfix form):**
```
L → E n             { L.val = E.val }
E → E1 + T          { E.val = E1.val + T.val }
E → T               { E.val = T.val }
T → T1 * F          { T.val = T1.val * F.val }
T → F               { T.val = F.val }
F → (E)             { F.val = E.val }
F → digit           { F.val = digit.lexval }
```

---

### 🔢 S-Attributed SDD Examples (All EXAM CRITICAL)

#### 1. Count number of 1's in a Binary Number:
```
Productions    Semantic Rules
L → L1 B      L.count = L1.count + B.count
L → B         L.count = B.count
B → 0         B.count = 0
B → 1         B.count = 1
```
**Trace for 1011:**
```
L → LB → L1 → LB1 → L11 → LB11 → L011 → B011 → 1011
Result: L.count = 3
```

#### 2. Count number of 0's in Binary Number:
```
B → 0         B.count = 1    (flip: 0 contributes 1 to count)
B → 1         B.count = 0    (flip: 1 contributes 0 to count)
```

#### 3. Count number of bits (total):
```
B → 0         B.count = 1
B → 1         B.count = 1
```

#### 4. Binary to Decimal Conversion:
```
Productions    Semantic Rules
L → L1 B      L.val = 2 * L1.val + B.val
L → B         L.val = B.val
B → 0         B.val = 0
B → 1         B.val = 1
```
**Trace for 1100 = 12:**
```
Derivation: L → LB → L0 → LB0 → L00 → LB00 → L100 → B100 → 1100
Step by step:
  B.val=1 → L.val=1 (from L→B)
  B.val=1 → L.val = 2*1+1 = 3
  B.val=0 → L.val = 2*3+0 = 6
  B.val=0 → L.val = 2*6+0 = 12  ✓
```

#### 5. Binary Fraction to Decimal (e.g., 11.11 = 3.75):
```
Productions    Semantic Rules
S → L1.L2     S.val = L1.val + L2.val / 2^(L2.count)
L → L1 B      L.val = 2*L1.val + B.val
              L.count = L1.count + B.count
L → B         L.val = B.val
              L.count = B.count
B → 0         B.val = 0; B.count = 1
B → 1         B.val = 1; B.count = 1
```
**For 11.11:**
- L1 (integer part "11"): val = 3, count = 2
- L2 (fractional part "11"): val = 3, count = 2
- S.val = 3 + 3/2² = 3 + 0.75 = **3.75** ✓

#### 6. Count Balanced Nested Parentheses:
```
S → (S1)      S.count = S1.count + 1
S → x         S.count = 0
```
**For (((x))): count = 3** ✓

#### 7. Infix to Postfix Conversion:
```
E → E + T     { print("+") }
E → E - T     { print("-") }
E → T
T → T * F     { print("*") }
T → T / F     { print("/") }
T → F
F → num       { print(num.lexval) }
```
**For input 4+5*6, output: 456*+**

**How it works (bottom-up):**
```
Rightmost derivation reverse:
E → E+T → E+T*F → E+T*num(6) → ... → num(4)+num(5)*num(6)
Actions fire when production is reduced:
  F→num(4): print(4)
  F→num(5): print(5)
  F→num(6): print(6)
  T→T*F:    print(*)
  E→E+T:    print(+)
Output: 4 5 6 * +  ✓
```

#### 8. Determine Type (int/float) of Expression:
```
E → E1 + T    if(E1.type == float || T.type == float) E.type = float
              else E.type = integer
E → T         E.type = T.type
T → num.num   T.type = float
T → num       T.type = integer
```

#### 9. Determine Sign (Positive/Negative):
```
S → E         if E.sign==POS: print("Result is Positive"); else print("Result is Negative")
E → E1 * E2   E.sign = POS if E1.sign == E2.sign, else NEG
E → + E1      E.sign = E1.sign
E → - E1      E.sign = NEG if E1.sign==POS, else POS
E → num       E.sign = POS
```
> `E.sign` is a **synthesized** attribute (computed from children).

#### 10. Count Executed Statements (for loop analysis):
```
PROGRAM → procedure STMT_LIST   PROGRAM.cnt = STMT_LIST.cnt
STMT_LIST → STMT STMT_LIST1     STMT_LIST.cnt = STMT.cnt + STMT_LIST1.cnt
STMT_LIST → STMT                STMT_LIST.cnt = STMT.cnt
STMT → do var=num1 to num2
        begin STMT_LIST end      STMT.cnt = (num2.val – num1.val + 1) * STMT_LIST.cnt
STMT → expr_stmt                STMT.cnt = 1
```

---

## 7. Evaluating SDDs – Parse Tree Method

### Steps to Evaluate an SDD:
1. **Construct the Parse Tree** for the given input
2. **Construct the Dependency Graph** (which attribute depends on which)
3. **Topologically sort** the dependency graph nodes
4. **Produce the Annotated Parse Tree** (parse tree with attribute values)

### Annotated Parse Tree
A parse tree showing the **values** of its attributes at every node.

**Example for 3*5+4n (using Desk Calculator SDD):**
```
          L.val=19
          |
          E.val=19
         / \
       E.val=15  +   T.val=4
       / \              |
    T.val=15         F.val=4
    / | \               |
T.val=3 * F.val=5     digit.lexval=4
   |         |
F.val=3  digit.lexval=5
   |
digit.lexval=3
```
Evaluation (bottom-up):
- `T.val = 3 * 5 = 15`
- `T.val = 4`
- `E.val = 15 + 4 = 19`
- `L.val = 19`

### Circular Dependency Problem ⚠️
```
Production    Semantic Rules
A → B         A.s = B.i
              B.i = A.s + 1     ← CIRCULAR! Cannot evaluate either.
```
> **Key Rule:** An SDD **cannot be evaluated** when there is a circular dependency in the attribute dependency graph.

---

## 8. Dependency Graphs & Evaluation Order

### Dependency Graph
A graph that shows **which attribute depends on which other attributes**.

- **Node** = one attribute of one grammar symbol at one parse-tree node
- **Edge from X.c → A.b** means: the value of X.c is needed to compute A.b

### Difference:
| Annotated Parse Tree | Dependency Graph |
|---|---|
| Shows **values** of attributes | Shows **order** in which to compute |

### Topological Sort
A valid evaluation order is a **topological sort** of the dependency graph:
- For every directed edge from node i to node j, node i appears **before** j in the ordering.
- Topological sort is only possible if the graph has **NO cycles** (is a DAG).

**Example:** For the dependency graph of `3*5+4`:
- Topological sort 1: `1,2,3,4,5,6,7,8,9`
- Topological sort 2: `1,3,5,2,4,6,7,8,9`
Both are valid because they respect all dependency edges.

> 💡 **Key:** If the dependency graph has **cycles**, the SDD is **not well-defined** and cannot be evaluated.

---

## 9. When Are Inherited Attributes Useful?

### Classic Example – Variable Type Declaration:
**Grammar:**
```
D → T L
T → int | float
L → L, id | id
```
**Problem:** How do we record the type of each `id` in the symbol table?
The type comes from `T`, but the identifiers are in `L`. Information must **flow down** from `T` to all the `id` nodes in `L`.

**Solution – L-Attributed SDD (Fig. 5.8):**
```
Productions    Semantic Rules
D → T L        L.inh = T.type
T → int        T.type = integer
T → float      T.type = float
L → L1, id     L1.inh = L.inh
               addType(id.entry, L.inh)
L → id         addType(id.entry, L.inh)
```
**L.inh is an INHERITED attribute** — it passes the declared type **downward** through the list.
`addType(id.entry, L.inh)` enters each identifier's type into the symbol table.

**Corresponding L-Attributed SDT:**
```
D → T { L.inh = T.type } L
T → int { T.type = integer }
T → float { T.type = float }
L → { L1.inh = L.inh } L1, id { addType(id.entry, L.inh) }
L → id { addType(id.entry, L.inh) }
```

### Why Inherited Attributes Cannot Be Used in Bottom-Up Parsing Directly:
- Bottom-up parsing moves from leaves to root.
- Inherited attributes require info from **parent or left sibling**, which may not yet be on the stack.
- For inherited attributes: use **top-down parsing** strategy.
- Special techniques (marker nonterminals) can make it work bottom-up.

---

## 10. L-Attributed SDD & Examples

### Definition
An SDD is **L-attributed** if in every production `A → X1 X2 … Xn`, for each inherited attribute `Xi.a`:
1. It may use **inherited attributes of the head A**
2. It may use **inherited or synthesized attributes of left siblings X1, …, Xi-1**
3. It may use **inherited or synthesized attributes of Xi itself** (no cycles)

**Key rule: Information flows Left to Right — inherited attributes NEVER flow from right siblings or children.**

### Important Relationship:
> Every **S-attributed** SDD is also **L-attributed**, but NOT vice versa.

```
          L-attributed SDT
        ┌─────────────────────┐
        │   S-attributed SDT  │
        └─────────────────────┘
```

### Example – Is it S-attributed or L-attributed?
```
P1: S → MN { S.val = M.val + N.val }   → S-attributed (and also L-attributed)
P2: M → PQ { M.val = P.val * Q.val; P.val = Q.val }   
    → P.val depends on Q (which is to the RIGHT of P) → NOT L-attributed!
```
**Answer: P1 is L-attributed, P2 is not L-attributed. (Option C)**

### L-Attributed SDD – Desk Calculator:
```
Productions    Semantic Rules
E → TE'        E'.inh = T.val;    E.val = E'.syn
E' → +TE'1     E'1.inh = E'.inh + T.val;  E'.syn = E'1.syn
E' → ε         E'.syn = E'.inh
T → FT'        T'.inh = F.val;   T.val = T'.syn
T' → *FT'1     T'1.inh = T'.inh * F.val;  T'.syn = T'1.syn
T' → ε         T'.syn = T'.inh
F → digit      F.val = digit.lexval
```
Here `E'.inh`, `T'.inh` are **inherited** (passed down from parent/left), while `.syn` and `.val` are **synthesized** (passed up).

### L-Attributed SDD – Array Type & Width Calculation:
**Grammar:**
```
T → BC
B → int | float
C → [num]C1 | ε
```
**Semantic Rules:**
```
Productions     Semantic Rules
T → BC          C.inhType = B.type;  C.inhWidth = B.width;
                T.type = C.type;     T.width = C.width
B → int         B.type = int;        B.width = 4
B → float       B.type = float;      B.width = 8
C → [num]C1     C1.inhType = C.inhType;  C1.inhWidth = C.inhWidth
                C.type = array(num.lexval, C1.type)
                C.width = num.lexval * C1.width
C → ε           C.type = C.inhType;  C.width = C.inhWidth
```

**For input `int[2][3]`:**
- B.type = int, B.width = 4
- Inner C (for `[3]`): C.type = array(3, int), C.width = 3×4 = 12
- Outer C (for `[2]`): C.type = array(2, array(3, int)), C.width = 2×12 = 24
- T.type = array(2, array(3, int)), T.width = 24 ✓

### L-Attributed Exercise – Is It L-attributed?
For production `A → BCD`:
```
a) A.s = B.i + C.s        → NOT L-attributed (inherited attr B.i cannot flow upward to synthesized A.s)
b) A.s = B.s + C.s and D.i = A.i + B.s  → YES, L-attributed (D inherits from A and left sibling B)
c) A.s = B.s + D.s        → YES, S-attributed (all synthesized)
d) D.i = C.i, C.i = B.i, B.i = D.i + A.i  → CYCLIC, no valid evaluation order
```

---

## 11. Construction of Syntax Trees (ASTs)

### Parse Tree vs Syntax Tree:
| Parse Tree | Syntax Tree (AST) |
|---|---|
| Interior nodes = nonterminals, leaves = terminals | Interior nodes = operators, leaves = operands |
| Concrete syntax (full grammar) | Abstract syntax (semantics only) |
| Verbose / redundant | Compact / meaningful |
| Parentheses appear as nodes | Grouping implicit in tree structure |

**For input `a + b * c`:**
```
Parse Tree:               Syntax Tree (AST):
    E                           +
   / \                         / \
  E   T                       a   *
  |  / \                         / \
  T  T   F                       b   c
  |  |   |
  F  F  id(c)
  |  |
 id  id(b)
  a
```

### Node Construction Functions:
- `Leaf(op, val)` — creates a leaf node (for terminals: id, num, etc.)
- `Node(op, c1, c2, ..., ck)` — creates an interior node with operator `op` and children `c1..ck`

### Example – SDD to Build AST for Expressions (Bottom-Up):
```
E → E1 + T    { E.node = new Node('+', E1.node, T.node) }
E → E1 - T    { E.node = new Node('-', E1.node, T.node) }
E → T         { E.node = T.node }
T → (E)       { T.node = E.node }
T → id        { T.node = new Leaf(id, id.entry) }
T → num       { T.node = new Leaf(num, num.val) }
```

**Building AST for `a - 4 + c` (steps):**
```
1) T.node = Leaf(id, entry-a)      → leaf node for 'a'
2) E.node = T.node                 → E points to 'a' leaf
3) T.node = Leaf(num, 4)           → leaf node for 4
4) E.node = Node('-', E.node, T.node)  → subtree for a-4
5) T.node = Leaf(id, entry-c)      → leaf node for 'c'
6) E.node = Node('+', E.node, T.node)  → full tree: (a-4)+c
```

### SDD to Build AST for Statements:
```
Stmt → S Stmt1      Stmt.node = new Node(Seq, S.node, Stmt1.node)
S → if(Cond){Stmt}  S.node = new Node(if, Cond.node, Stmt.node)
S → while(Cond){Stmt} S.node = new Node(while, Cond.node, Stmt.node)
S → AssignExpr      S.node = AssignExpr.node
AssignExpr → id=E;  AssignExpr.node = new Node('=', new Leaf(id, id.entry), E.node)
```

---

## 12. SDT Schemes

### Postfix SDTs (S-Attributed)
- All semantic actions placed at the **rightmost end** of production bodies.
- Executed during **reduction** in LR parsing.
- Only works when grammar is LR and SDD is S-attributed.

**Postfix SDT for Desk Calculator:**
```
L → E n      { print(E.val) }
E → E1 + T   { E.val = E1.val + T.val }
...
F → digit    { F.val = digit.lexval }
```

### Parser-Stack Implementation of Postfix SDTs
In LR parsing, semantic actions can directly manipulate the **parser stack**:
```
E → E1 + T   { stack[top-2].val = stack[top-2].val + stack[top].val; top = top-2; }
T → T1 * F   { stack[top-2].val = stack[top-2].val * stack[top].val; top = top-2; }
```
Each grammar symbol on the stack has a **record** holding its attributes.

### Actions Inside Productions
An action at position `p` in `B → X {a} Y` is executed:
- In **bottom-up parsing**: as soon as X is on top of the parser stack
- In **top-down parsing**: just before expanding Y (or matching Y if terminal)

**Problematic SDT (Cannot be parsed directly):**
```
E → {print('+');} E1 + T   ← cannot print '+' before knowing if '+' exists!
```
**Solution for problematic SDTs:**
1. Build parse tree (ignoring actions)
2. Insert actions as extra children at correct positions
3. Perform **preorder traversal** and execute actions when visited

### SDT Exercise Examples (Exam Favorites!):

**Exercise 1 – Bottom-up, input `aab`:**
```
S → aA { print('1') }
S → a  { print('2') }
A → Sb { print('3') }
```
```
S → aA → aSb → aab
Actions: print(2), print(3), print(1) → Output: 231
```

**Exercise 5 – Bottom-up, input `4+5*6`:**
```
S → E {print("$")}
E → E+T {print("+")}
T → T*F {print("*")}
F → num {print(num.val)}
```
**Output: `456*+$`**

---

## 13. Eliminating Left Recursion from SDTs

### Why Needed?
Left-recursive grammars cannot be used with top-down (LL) parsers.
When removing left recursion, treat **semantic actions as terminal symbols** to preserve their order.

### Pattern:
```
A → Aα | β    becomes:    A → βA'
                           A' → αA' | ε
```

### Example – Infix to Postfix (converting for top-down parsing):
```
Bottom-up (LR) grammar:          Top-down (LL) grammar after eliminating LR:
E → E + T {print('+')}           E → TE'
E → E - T {print('-')}           E' → +T {print('+')} E'
E → T                            E' → -T {print('-')} E'
T → F                            E' → ε
F → digit {print(digit.lexval)}  T → F
                                 F → digit {print(digit.lexval)}
```
**Both print the same output for input `6+4-3`: `64+3-`** (postfix)

---

## 14. Intermediate Code Generation (TAC)

### Three-Address Code (TAC) Concepts:
- Each instruction has at most **one operator** and **three addresses** (2 operands + 1 result)
- Temporary variables like `t1, t2, ...` hold intermediate results

### SDD for Arithmetic Expression `a = b + -c`:
```
S → id = E;      S.code = E.code || gen(id.lexval '=' E.addr)
E → E1 + T       E.addr = newTemp()
                 E.code = E1.code || T.code || gen(E.addr '=' E1.addr '+' T.addr)
E → -E1          E.addr = newTemp()
                 E.code = E1.code || gen(E.addr '=' 'minus' E1.addr)
E → id           E.addr = id.lexval;  E.code = ''
```
**For `a = b + -c`:**
```
TAC output:
t1 = minus c
t2 = b + t1
a = t2
```

### Intermediate Code for While Statement:
```
Production:  S → while (C) S1
Rules:
  L1 = new()           // label for start of while
  L2 = new()           // label for body of while
  S1.next = L1         // after body, loop back
  C.false = S.next     // if condition false, exit
  C.true = L2          // if condition true, enter body
  S.code = label||L1||C.code||label||L2||S1.code
```
**Generated Code Structure:**
```
L1:
    [code for C]         ; evaluate condition
    if C.true goto L2
    goto S.next
L2:
    [code for S1]        ; loop body
    goto L1              ; back to condition
```

**L-Attributed SDT for While:**
```
S → while( {L1=new(); L2=new(); C.false=S.next; C.true=L2;} C )
           {S1.next=L1;} S1
           {S.code = label||L1||C.code||label||L2||S1.code;}
```

### Intermediate Code for Do-While:
```
S → do S1 while(C)
L1 = new();  L2 = new()
S1.next = L2;   C.false = S.next;   C.true = L1
S.code = label||L1||S1.code||label||L2||C.code
```
**Structure:**
```
L1:
    [S1.code]   ; execute body first
L2:
    [C.code]    ; then check condition
    if C.true goto L1
    goto S.next
```

### Intermediate Code for If-Else:
```
S → if(C) S1 else S2
L1 = new();  L2 = new()
C.true = L1;  C.false = L2
S1.next = S.next;  S2.next = S.next
S.code = C.code||label||L1||S1.code||label||L2||S2.code
```
**Structure:**
```
[C.code]     ; evaluate condition
    if C.true goto L1
    goto L2
L1:
    [S1.code]  ; if branch
    goto S.next
L2:
    [S2.code]  ; else branch
```

### Intermediate Code for For Loop:
```
S → for(S1; C; S3) S4
L1 = new(); L2 = new(); L3 = new()
S1.next = L1;  C.true = L2;  C.false = S.next
S4.next = L3;  S3.next = L1
S.code = S1.code||label||L1||C.code||label||L2||S4.code||label||L3||S3.code
```
**Structure:**
```
[S1.code]     ; initialization (e.g., i=0)
L1:
    [C.code]  ; condition check
    if C.true goto L2
    goto S.next
L2:
    [S4.code] ; loop body
L3:
    [S3.code] ; increment
    goto L1
```

### SDD for Boolean Expressions:
```
B → B1 || B2:
  L1 = new()
  B1.true = B.true    // short-circuit: if B1 is true, whole OR is true
  B1.false = L1       // if B1 is false, try B2
  B2.true = B.true
  B2.false = B.false
  B.code = B1.code||label||L1||B2.code

B → B1 && B2:
  L1 = new()
  B1.true = L1        // if B1 is true, check B2
  B1.false = B.false  // if B1 is false, whole AND is false
  B2.true = B.true
  B2.false = B.false
  B.code = B1.code||label||L1||B2.code

B → !B1:
  B1.true = B.false   // just swap true/false — no extra code needed!
  B1.false = B.true
  B.code = B1.code

B → E1 rel E2:
  B.code = E1.code||E2.code
           ||gen('if' E1.addr rel.op E2.addr 'goto' B.true)
           ||gen('goto' B.false)

B → true:   B.code = gen('goto' B.true)
B → false:  B.code = gen('goto' B.false)
```

---

## 15. Bottom-Up Parsing of L-Attributed SDDs

### The Challenge
L-attributed SDDs have **embedded actions** in the middle of productions. LR parsers normally execute actions only at the **end** (during reduction). So how do we handle them bottom-up?

### The "Trick" – Marker Nonterminals
**Three-step process:**
1. Start with the SDT that has embedded actions (from L-attributed SDD)
2. Replace each embedded action with a **marker nonterminal M** with production `M → ε`
3. Modify the action associated with M to use attributes from the stack (at known offsets)

**Pattern:**
```
A → {B.i = f(A.i);} B C
becomes:
A → M B C
M → ε { M.i = A.i; M.s = f(M.i); }
```
(M.s becomes effectively B.i — found below B when B is reduced)

### Example – While Statement:
**L-attributed SDT:**
```
S → while({L1=new(); L2=new(); C.false=S.next; C.true=L2;} C)
           {S1.next=L1;} S1
           {S.code = label||L1||C.code||label||L2||S1.code;}
```
**Becomes (suitable for LR parsing):**
```
S → while(M C) N S1 {S.code = label||L1||C.code||label||L2||S1.code;}
M → ε  {L1=new(); L2=new(); C.false=S.next; C.true=L2;}
N → ε  {S1.next=L1;}
```
**Stack access:**
- `S.next` is below `while` on stack
- `C.true/C.false` come from M's record (just below C)
- `S1.next` comes from N's record (just below S1)

### Example – Do-While:
```
S → do M S1 while(N C) {S.code=label||L1||S1.code||label||L2||C.code}
M → ε {L2=new(); S1.next=L2;}
N → ε {L1=new(); C.false=S.next; C.true=L1;}
```

### Example – If-Else:
```
S → if(M C) N S1 else P S2 {S.code=C.code||label||L1||S1.code||label||L2||S2.code}
M → ε {L1=new(); L2=new(); C.false=L2; C.true=L1;}
N → ε {S1.next=S.next;}
P → ε {S2.next=S.next;}
```

---

## 16. Important Keywords & Quick Revision Summary

### 📋 Keyword Glossary

| Term | Definition |
|---|---|
| **Semantics** | The meaning of program constructs |
| **Semantic Analysis** | Phase that checks meaning, performs type checking, fills symbol table |
| **Attribute** | A variable attached to a grammar symbol holding computed information |
| **Synthesized Attribute** | Computed from children nodes; flows upward |
| **Inherited Attribute** | Computed from parent/left siblings; flows downward/sideways |
| **SDD** | CFG + attributes + semantic rules; specification formalism |
| **SDT** | CFG + embedded program fragments (semantic actions); implementation |
| **S-Attributed SDD** | SDD using only synthesized attributes; evaluated bottom-up |
| **L-Attributed SDD** | SDD where inherited attributes depend only on left siblings and parent |
| **Annotated Parse Tree** | Parse tree with computed attribute values at all nodes |
| **Dependency Graph** | Graph showing which attributes depend on which others |
| **Topological Sort** | Linear ordering that respects all dependencies in a DAG |
| **Attribute Grammar** | CFG augmented with attributes and semantic rules |
| **Postfix SDT** | SDT with all actions at the end of production bodies |
| **Postorder Traversal** | Visit children first, then parent; used for S-attributed SDDs |
| **Marker Nonterminal** | Dummy nonterminal `M → ε` used to handle embedded actions in LR parsing |
| **AST** | Abstract Syntax Tree; compact tree where operators are interior nodes |
| **TAC** | Three-Address Code; intermediate code with ≤1 operator and ≤3 addresses |
| **Symbol Table** | Data structure storing info about all identifiers (name, type, scope) |
| **Type Checking** | Semantic check ensuring type compatibility in operations |
| `id.entry` | Lexical value pointing to a symbol-table entry for an identifier |
| `newTemp()` | Function creating a new temporary variable for TAC |
| `new()` | Function creating a new label for intermediate code |
| `gen()` | Function that emits/generates a TAC instruction |
| `addType()` | Function that records type of an identifier in the symbol table |

### ⚡ Quick Revision Summary

#### Core Concepts:
- **SDD** = Specification; **SDT** = Implementation
- **Synthesized** = bottom-up (children → parent); **Inherited** = top-down (parent/left sibling → node)
- **S-attributed** ⊆ **L-attributed** (every S-attributed is also L-attributed)
- **S-attributed** → LR/bottom-up parsing; **L-attributed** → LL/top-down parsing
- **Circular dependency** → SDD cannot be evaluated

#### Attribute Flow Cheat Sheet:
```
Synthesized:   children ──→ parent         (↑ up)
Inherited:     parent   ──→ child          (↓ down)
               left sibling → right sibling (→ sideways)
```

#### For Intermediate Code Generation:
```
While loop:   label L1; C.code; if true goto L2; goto S.next; label L2; S1.code; goto L1
Do-While:     label L1; S1.code; label L2; C.code; if true goto L1; goto S.next
If-Else:      C.code; if true goto L1; goto L2; label L1; S1.code; goto S.next; label L2; S2.code
For loop:     S1.code; label L1; C.code; if true goto L2; goto S.next; label L2; S4.code; label L3; S3.code; goto L1
```

---

## 17. Question Bank

### ✏️ 1-Mark Questions

**Q1.** What is a synthesized attribute?
**A.** An attribute whose value is computed from the attribute values of its children nodes in the parse tree; information flows upward.

**Q2.** What is an inherited attribute?
**A.** An attribute whose value is computed from the attribute values of its parent node, left siblings, or itself; information flows downward or sideways.

**Q3.** What is an S-attributed SDD?
**A.** An SDD that uses only synthesized attributes.

**Q4.** What is an L-attributed SDD?
**A.** An SDD where each inherited attribute of a symbol Xi in a production can only depend on attributes of the head symbol A, attributes of symbols to the left of Xi (X1...Xi-1), and attributes of Xi itself.

**Q5.** Can terminals have inherited attributes?
**A.** No. Only non-terminals can have inherited attributes.

**Q6.** What is an annotated parse tree?
**A.** A parse tree that shows the computed values of attributes at each of its nodes.

**Q7.** What is a dependency graph?
**A.** A directed graph showing the dependencies between attribute instances; an edge from X.c to A.b means X.c must be computed before A.b.

**Q8.** What is the condition for evaluating an SDD using topological sort?
**A.** The dependency graph must have no cycles (must be a DAG).

**Q9.** What is TAC?
**A.** Three-Address Code — an intermediate code form where each instruction has at most one operator and three addresses (two operands and one result).

**Q10.** What is a marker nonterminal?
**A.** A dummy nonterminal M with production `M → ε` introduced to replace embedded actions, enabling L-attributed SDTs to be evaluated during LR/bottom-up parsing.

---

### ✏️ 2-Mark Questions

**Q1.** Differentiate between SDD and SDT.
**A.** 
- **SDD** = CFG + attributes + semantic rules; it is a *specification* formalism that is easier to read and understand.
- **SDT** = CFG + program fragments (semantic actions) embedded in production bodies; it is an *implementation* mechanism that specifies the exact order of execution and is more efficient.

**Q2.** Differentiate between synthesized and inherited attributes.
**A.** 
- **Synthesized**: Computed from children; flows upward; applicable to both terminals and non-terminals; used in S-attributed and L-attributed SDDs.
- **Inherited**: Computed from parent/left siblings; flows downward or sideways; only for non-terminals; used only in L-attributed SDDs.

**Q3.** What is the relationship between S-attributed and L-attributed SDDs?
**A.** Every S-attributed SDD is also L-attributed, but not vice versa. S-attributed is a strict subset — it only uses synthesized attributes, while L-attributed allows inherited attributes with the restriction that they flow from left to right only.

**Q4.** Why can't inherited attributes be easily evaluated in bottom-up parsing?
**A.** Bottom-up parsing constructs the parse tree from leaves to root. Inherited attributes require values from parent or left siblings, which may not yet be known. Special techniques like marker nonterminals must be used to handle them during LR parsing.

**Q5.** Give the SDD for binary-to-decimal conversion.
**A.**
```
L → L1 B    L.val = 2 * L1.val + B.val
L → B       L.val = B.val
B → 0       B.val = 0
B → 1       B.val = 1
```

**Q6.** What is the difference between a parse tree and a syntax tree (AST)?
**A.** A parse tree has non-terminals as interior nodes and terminals as leaves — it represents concrete syntax. A syntax tree (AST) has operators as interior nodes and operands as leaves — it represents abstract syntax, is more compact, and discards redundant structural information like parentheses.

**Q7.** What are semantic errors? Give two examples.
**A.** Semantic errors are errors that are syntactically correct but semantically invalid. Examples: (1) Type mismatch: `int x = "hello"` — assigning a string to an integer variable. (2) Undeclared variable: using a variable `y` that was never declared.

---

### ✏️ 5-Mark / 10-Mark Questions

---

**Q1. (5 marks)** Explain S-attributed SDDs with the desk calculator example. Show the annotated parse tree for 3*5+4.

**A.**
An **S-attributed SDD** uses only synthesized attributes. Each semantic rule computes the attribute of the head (LHS) of a production from the attributes of the body (RHS).

**Desk Calculator SDD:**
```
L → E n       L.val = E.val
E → E1 + T    E.val = E1.val + T.val
E → T         E.val = T.val
T → T1 * F    T.val = T1.val * F.val
T → F         T.val = F.val
F → (E)       F.val = E.val
F → digit     F.val = digit.lexval
```

**Annotated Parse Tree for 3*5+4n:**
```
         L.val=19
             |
         E.val=19
       /    |     \
   E.val=15  +   T.val=4
    /  |  \          |
T.val=15      F.val=4
  / | \           |
T=3 * F=5    digit.lexval=4
  |      |
F=3  digit(5)
  |
digit(3)
```
Bottom-up evaluation:
- F.val=3 (digit.lexval=3)
- T.val=3 (from T→F)
- F.val=5 (digit.lexval=5)
- T.val=15 (3*5=15)
- F.val=4 (digit.lexval=4)
- T.val=4 (from T→F)
- E.val=15 (from E→T)
- E.val=19 (15+4)
- L.val=19

---

**Q2. (5 marks)** What are inherited attributes? Explain with the type declaration example `float x, y`.

**A.**
An **inherited attribute** at a parse-tree node N is computed from N's parent, left siblings, or N itself. Information flows **downward** or **sideways** in the parse tree.

**Why needed for `float x, y`:**
The type `float` comes from the T nonterminal, but the identifiers x, y are in the L subtree. We need to pass this type **down** to each identifier — hence inherited attributes.

**L-attributed SDD:**
```
D → TL        L.inh = T.type
T → int       T.type = integer
T → float     T.type = float
L → L1, id    L1.inh = L.inh;  addType(id.entry, L.inh)
L → id        addType(id.entry, L.inh)
```

**Parse tree for `float x, y`:**
```
          D
         / \
        T   L
      float / | \
           L  ,  id(y)
           |
          id(x)
```
- T.type = float
- L.inh (at root L) = T.type = float  ← inherited!
- L.inh (at child L) = L.inh = float  ← passed sideways
- addType(id.entry for x, float) → x:float in symbol table
- addType(id.entry for y, float) → y:float in symbol table

---

**Q3. (10 marks)** Explain SDTs for generating intermediate code (TAC) for while, do-while, and if-else statements with SDD, SDT, and code structure.

**A.**

### While Statement: `while(C) S1`
```
Visual code structure:
L1:   [C.code]      ← evaluate condition
      if C goto L2  ← if true, go to body
      goto S.next   ← if false, exit
L2:   [S1.code]     ← body
      goto L1       ← loop back
```
**SDD:**
```
L1=new(); L2=new()
S1.next = L1;  C.false = S.next;  C.true = L2
S.code = label||L1||C.code||label||L2||S1.code
```
**SDT (L-attributed):**
```
S → while({L1=new(); L2=new(); C.false=S.next; C.true=L2;} C)
         {S1.next=L1;} S1
         {S.code = label||L1||C.code||label||L2||S1.code;}
```
**SDT (for LR parsing, using marker nonterminals):**
```
S → while(M C) N S1 {S.code=label||L1||C.code||label||L2||S1.code;}
M → ε  {L1=new(); L2=new(); C.false=S.next; C.true=L2;}
N → ε  {S1.next=L1;}
```

---

### Do-While Statement: `do S1 while(C)`
```
Visual code structure:
L1:   [S1.code]     ← execute body first
L2:   [C.code]      ← then check condition
      if C goto L1  ← if true, loop back
      goto S.next   ← if false, exit
```
**SDD:**
```
L1=new(); L2=new()
S1.next=L2;  C.false=S.next;  C.true=L1
S.code = label||L1||S1.code||label||L2||C.code
```
**SDT (for LR parsing):**
```
S → do M S1 while(N C) {S.code=label||L1||S1.code||label||L2||C.code}
M → ε  {L2=new(); S1.next=L2;}
N → ε  {L1=new(); C.false=S.next; C.true=L1;}
```

---

### If-Else Statement: `if(C) S1 else S2`
```
Visual code structure:
      [C.code]      ← evaluate condition
      if C goto L1  ← if true, go to S1
      goto L2       ← else go to S2
L1:   [S1.code]     ← true branch
      goto S.next   ← exit if
L2:   [S2.code]     ← false branch
```
**SDD:**
```
L1=new(); L2=new()
C.true=L1;  C.false=L2
S1.next=S.next;  S2.next=S.next
S.code = C.code||label||L1||S1.code||label||L2||S2.code
```
**SDT (for LR parsing):**
```
S → if(M C) N S1 else P S2 {S.code=C.code||label||L1||S1.code||label||L2||S2.code}
M → ε {L1=new(); L2=new(); C.false=L2; C.true=L1;}
N → ε {S1.next=S.next;}
P → ε {S2.next=S.next;}
```

---

**Q4. (5 marks)** Explain dependency graphs and topological ordering. Give an example.

**A.**
A **dependency graph** for a parse tree depicts the flow of information among attribute instances. 

**Rules for constructing dependency graph:**
1. For each parse-tree node, create one dependency-graph node per attribute.
2. If a semantic rule defines synthesized attribute A.b from X.c, draw edge X.c → A.b.
3. If a semantic rule defines inherited attribute B.c from X.a, draw edge X.a → B.c.

**Example:** For production `E → E1 + T` with rule `E.val = E1.val + T.val`:
- Draw edges: `E1.val → E.val` and `T.val → E.val`
- This means E1.val and T.val must both be computed before E.val.

**Topological Sort:** A linear ordering of nodes such that for every edge u → v, u appears before v. This gives us a valid evaluation order.

Example ordering: if nodes are {1,2,3,4,5,6,7,8,9} and edges only go from lower to higher numbers, one valid topological sort is 1,2,3,4,5,6,7,8,9.

> ⚠️ If the dependency graph has a cycle, the SDD is not well-defined and CANNOT be evaluated.

---

**Q5. (10 marks)** Describe the method for implementing L-attributed SDDs during bottom-up parsing using marker nonterminals. Explain with the while-statement example, showing the parser stack at various stages.

**A.**
*(Full detailed answer with parser stack stages)*

**The method** consists of:
1. Take the L-attributed SDT with embedded actions
2. Replace each embedded action with marker nonterminal M (with `M → ε`)
3. Move embedded action into M's semantic rule
4. Implement using parser stack, accessing attributes at known stack offsets

**For while-statement:**
```
Original L-attributed SDT:
S → while({L1=new(); L2=new(); C.false=S.next; C.true=L2;} C)
         {S1.next=L1;} S1 {S.code=label||L1||C.code||label||L2||S1.code;}

After transformation:
S → while(M C) N S1 {S.code=label||L1||C.code||label||L2||S1.code;}
M → ε  {L1=new(); L2=new(); C.false=stack[top-3].next; C.true=L2;}
N → ε  {S1.next=stack[top-3].L1;}
```

**Parser Stack Evolution:**

Step 1: Stack has `[..., S.next record]`, input starts with `while`
→ Shift `while` and `(`

Step 2: Stack: `[..., S.next, while, (]`
→ Reduce ε to M, execute: L1=new(); L2=new(); C.false=S.next; C.true=L2
→ Stack: `[..., S.next, while, (, M]` (M record holds L1, L2)

Step 3: Parse and reduce condition to C
→ Stack: `[..., S.next, while, (, M, C]` (C record holds C.code)

Step 4: Shift `)`
→ Stack: `[..., S.next, while, (, M, C, )]`
→ Reduce ε to N, execute: S1.next = stack[top-3].L1
→ Stack: `[..., S.next, while, (, M, C, ), N]`

Step 5: Parse and reduce loop body to S1
→ Stack: `[..., S.next, while, (, M, C, ), N, S1]` (S1.code in S1 record)

Step 6: Reduce `while(M C) N S1` to S
→ Execute: `tempCode = label||stack[top-4].L1||stack[top-3].code||label||stack[top-4].L2||stack[top].code`
→ `top = top - 6`; `stack[top].code = tempCode`
→ S appears on stack with correct S.code ✓

This demonstrates how inherited attributes (like C.true, C.false, S1.next) are managed using the stack offsets during LR parsing.

---

### 🎯 Final Memory Tricks

1. **S-attributed → S for Simple (Synthesized only) → Bottom-up**
2. **L-attributed → L for Left-to-right → Can also go Top-down**
3. **Synthesized flows UP like SAP rising in a tree**
4. **Inherited flows DOWN like water trickling from parent to child**
5. **SDD = Describe (specification) | SDT = Do (implementation)**
6. **Marker nonterminal = Placeholder for actions that must happen "before" something during LR parsing**
7. **For While TAC: L1 is for condition, L2 is for body**
8. **For Do-While TAC: L1 is for body, L2 is for condition (reversed!)**

---
*Notes compiled from UE23CS341B – Compiler Design, Chapter 5 | Prof. Prakash C O, PES University*

