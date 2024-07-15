# Syntactic Analysis

---

## 1. Introduction

In DJ, bison is used to create the parser, which analyzes the tokenized source code and builds a syntax tree that represents the program's structure. This involves defining a set of grammar rules in the bison syntax, specifying how valid sequences of tokens should look. Bison then turns these rules into a C program that performs syntactic analysis, making sure the code follows the language's syntax rules and helping with the next stages of compilation or interpretation.

---

## 2. Operators' Precedence and Associativity

The following list defines the precedence and associativity of operators for the expression non-terminals, ensuring that expressions are parsed correctly according to the rules of the DJ language. Operators with higher precedence are evaluated before operators with lower precedence. Additionally, operators can be left-associative, right-associative, or non-associative.

<sub><sub>low precedence</sub></sub>
'='             : right
'&&'            : left
'=='            : non
'<'             : non
'+' '-'         : left
'*'             : left
'!'             : right
'instanceof'    : non
'.'             : right
<sub><sub>high precedence</sub></sub>

---

## 3. Context-Free Grammar 

Context-free grammar (CFG) is used to define the syntax rules. The first rule in this grammar is 'program', which serves as the entry point for parsing the source code. Terminals (CAP) are the basic symbols of the language, such as keywords, operators, and punctuation. Non-terminals (lower) are placeholders for patterns that can be expanded. 

**program** ::= class-decl-list MAIN LBRACE var-decl-list expr-list RBRACE ENDOFFILE 

**class-decl-list** ::= $\epsilon$ | class-decl-list class-decl 

**class-decl** ::= CLASS ast-id EXTENDS ast-id LBRACE static-var-decl-list var-decl-list method-block RBRACE 

**static-var-decl-list** ::= $\epsilon$ | static-var-decl-list static-var-decl

**static-var-decl** ::= STATIC type ast-id SEMICOLON 

**var-decl-list** ::= $\epsilon$ | var-decl-list var-decl

**method-block**  ::= $\epsilon$ | method-decl-list

**method-decl-list** ::= method-decl | method-decl-list method-decl

**method-decl** ::= type ast-id LPAREN type ast-id RPAREN LBRACE var-decl-list expr-list RBRACE 

**type** ::= NATTYPE | BOOLTYPE | ast-id

**ast-id** ::= ID

**expr-list** ::= expr SEMICOLON 

**expr** ::= paren-expr | dot-method-call-expr | method-call-expr | dot-id-expr | id-expr | dot-assign-expr | assign-expr | plus-expr | minus-expr | times-expr | equality-expr | less-than-expr | not-expr | and-expr | instanceof-expr | if-then-else-expr | for-expr | print-expr | read-expr | this-expr | new-expr | null-expr | nat-literal-expr | true-literal-expr | false-literal-expr

**paren-expr** ::= LPAREN expr RPAREN

**dot-method-call-expr** ::= expr DOT ast-id LPAREN expr RPAREN

**method-call-expr** ::= ast-id LPAREN expr RPAREN

**dot-id-expr** ::= expr DOT ast-id

**id-expr** ::= ast-id

**dot-assign-expr** ::= expr DOT ast-id ASSIGN expr

**assign-expr** ::= ast-id ASSIGN expr

**plus-expr** ::= expr PLUS expr

**minus-expr** ::= expr MINUS expr

**times-expr** ::= expr TIMES expr

**equality-expr** ::= expr EQUALITY expr

**less-than-expr** ::= expr LESS expr

**not-expr** ::= NOT expr

**and-expr** ::= expr AND expr

**instanceof-expr** ::= expr INSTANCEOF ast-id 

**if-then-else-expr** ::= IF LPAREN expr RPAREN LBRACE expr-list RBRACE ELSE LBRACE expr-list RBRACE

**for-expr** ::= FOR LPAREN expr SEMICOLON expr SEMICOLON expr RPAREN LBRACE expr-list RBRACE

**print-expr** ::= PRINTNAT LPAREN expr RPAREN

**read-expr** ::= READNAT LPAREN RPAREN

**this-expr** ::= THIS

**new-expr** ::= NEW ast-id LPAREN RPAREN

**null-expr** ::= NUL

**nat-literal-expr** ::= NATLITERAL

**true-literal-expr** ::= TRUELITERAL

**false-literal-expr** ::= FALSELITERAL

---

## 4. Abstract Syntax Tree

```
typedef struct ASTList ASTList;
typedef struct ASTree ASTree;
ASTree *newAST(...);
ASTree *appendToChildrenList(...);
```

An Abstract Syntax Tree (AST) is built using the parsing information obtained from the context-free grammar and tokenized source code. The AST represents the hierarchical structure of the program.

Two key data structures to build its AST: ASTList and ASTree. The ASTList is a simple linked list where each node points to an AST node and the next node in the list. This helps in organizing and accessing the children of any given AST node efficiently.

The ASTree structure represents individual nodes in the AST. Each node includes details about its type, the list of its children, the line number in the source code where the node ends, and various attributes used during type checking and code generation. These attributes help in identifying whether a node represents a natural numbers, an identifier, or a class member, and include specific information needed for typechecking the code.

To build the AST, the newAST function creates a new AST node with a specified type and initializes it with a single child node, along with attributes such as natural numbers or identifiers when needed. The appendToChildrenList function allows adding a new child node to a parent node's list of children, thus enabling the dynamic construction of the tree as the source code is parsed.

Together, these structures and functions help in systematically breaking down and representing the source code in a hierarchical, tree-like format, making it easier for further processing and compilation.

---

## 5. Examples

### 1. example1

#### code

```
main {0;}
```

#### output

```
0:PROGRAM   (ends on line 2)
1:   CLASS_DECL_LIST   (ends on line 1)
1:   VAR_DECL_LIST   (ends on line 1)
1:   EXPR_LIST   (ends on line 1)
2:      NAT_LITERAL_EXPR(0)   (ends on line 1)
```

### 2. example2

#### code

```
main {
  nat x; 
  x=0; 
  x=x+1;
  x=x+1; 
  x=x+1;
  x=x+1; 
  printNat(x);
}
```

#### output

```
0:PROGRAM   (ends on line 12)
1:   CLASS_DECL_LIST   (ends on line 1)
1:   VAR_DECL_LIST   (ends on line 2)
2:      VAR_DECL   (ends on line 3)
3:         NAT_TYPE   (ends on line 3)
3:         AST_ID(x)   (ends on line 3)
1:   EXPR_LIST   (ends on line 4)
2:      ASSIGN_EXPR   (ends on line 4)
3:         AST_ID(x)   (ends on line 4)
3:         NAT_LITERAL_EXPR(0)   (ends on line 4)
2:      ASSIGN_EXPR   (ends on line 5)
3:         AST_ID(x)   (ends on line 5)
3:         PLUS_EXPR   (ends on line 5)
4:            ID_EXPR   (ends on line 5)
5:               AST_ID(x)   (ends on line 5)
4:            NAT_LITERAL_EXPR(1)   (ends on line 5)
2:      ASSIGN_EXPR   (ends on line 6)
3:         AST_ID(x)   (ends on line 6)
3:         PLUS_EXPR   (ends on line 6)
4:            ID_EXPR   (ends on line 6)
5:               AST_ID(x)   (ends on line 6)
4:            NAT_LITERAL_EXPR(1)   (ends on line 6)
2:      ASSIGN_EXPR   (ends on line 7)
3:         AST_ID(x)   (ends on line 7)
3:         PLUS_EXPR   (ends on line 7)
4:            ID_EXPR   (ends on line 7)
5:               AST_ID(x)   (ends on line 7)
4:            NAT_LITERAL_EXPR(1)   (ends on line 7)
2:      ASSIGN_EXPR   (ends on line 8)
3:         AST_ID(x)   (ends on line 8)
3:         PLUS_EXPR   (ends on line 8)
4:            ID_EXPR   (ends on line 8)
5:               AST_ID(x)   (ends on line 8)
4:            NAT_LITERAL_EXPR(1)   (ends on line 8)
2:      PRINT_EXPR   (ends on line 9)
3:         ID_EXPR   (ends on line 9)
4:            AST_ID(x)   (ends on line 9)
```

### 3. example3

#### code

```
class Factorial extends Object {  
  nat factorial(nat n) {
    if(1<n) {n*factorial(n-1);}
    else {1;};
  }
}

main {
  Factorial f;
  f = new Factorial();
  printNat(f.factorial(readNat()));
}
```

#### output

```
0:PROGRAM   (ends on line 20)
1:   CLASS_DECL_LIST   (ends on line 1)
2:      CLASS_DECL   (ends on line 11)
3:         AST_ID(Factorial)   (ends on line 5)
3:         AST_ID(Object)   (ends on line 5)
3:         STATIC_VAR_DECL_LIST   (ends on line 5)
3:         VAR_DECL_LIST   (ends on line 7)
3:         METHOD_DECL_LIST   (ends on line 10)
4:            METHOD_DECL   (ends on line 10)
5:               NAT_TYPE   (ends on line 7)
5:               AST_ID(factorial)   (ends on line 7)
5:               NAT_TYPE   (ends on line 7)
5:               AST_ID(n)   (ends on line 7)
5:               VAR_DECL_LIST   (ends on line 7)
5:               EXPR_LIST   (ends on line 9)
6:                  IF_THEN_ELSE_EXPR   (ends on line 9)
7:                     LESS_THAN_EXPR   (ends on line 8)
8:                        NAT_LITERAL_EXPR(1)   (ends on line 8)
8:                        ID_EXPR   (ends on line 8)
9:                           AST_ID(n)   (ends on line 8)
7:                     EXPR_LIST   (ends on line 8)
8:                        TIMES_EXPR   (ends on line 8)
9:                           ID_EXPR   (ends on line 8)
10:                              AST_ID(n)   (ends on line 8)
9:                           METHOD_CALL_EXPR   (ends on line 8)
10:                              AST_ID(factorial)   (ends on line 8)
10:                              MINUS_EXPR   (ends on line 8)
11:                                 ID_EXPR   (ends on line 8)
12:                                    AST_ID(n)   (ends on line 8)
11:                                 NAT_LITERAL_EXPR(1)   (ends on line 8)
7:                     EXPR_LIST   (ends on line 9)
8:                        NAT_LITERAL_EXPR(1)   (ends on line 9)
1:   VAR_DECL_LIST   (ends on line 13)
2:      VAR_DECL   (ends on line 14)
3:         AST_ID(Factorial)   (ends on line 14)
3:         AST_ID(f)   (ends on line 14)
1:   EXPR_LIST   (ends on line 15)
2:      ASSIGN_EXPR   (ends on line 15)
3:         AST_ID(f)   (ends on line 15)
3:         NEW_EXPR   (ends on line 15)
4:            AST_ID(Factorial)   (ends on line 15)
2:      PRINT_EXPR   (ends on line 16)
3:         DOT_METHOD_CALL_EXPR   (ends on line 16)
4:            ID_EXPR   (ends on line 16)
5:               AST_ID(f)   (ends on line 16)
4:            AST_ID(factorial)   (ends on line 16)
4:            READ_EXPR   (ends on line 16)
```





