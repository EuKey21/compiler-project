# Lexical Analysis

---

## 1. Introduction

In DJ, flex is used to tokenize the source code, breaking it down into meaningful symbols and keywords that the compiler can understand. This process involves defining a set of rules and patterns in the flex syntax, which flex then converts into a C program capable of performing lexical analysis. 

---

## 2. Token List

CLASS EXTENDS MAIN IF ELSE FOR STATIC NEW THIS
NATTYPE BOOLTYPE
NATLITERAL TRUELITERAL FALSELITERAL NUL ID
PRINTNAT READNAT
AND NOT PLUS MINUS TIMES EQUALITY LESS ASSIGN DOT INSTANCEOF
SEMICOLON LBRACE RBRACE LPAREN RPAREN
ENDOFFILE

---

## 3. Rules/Regular Expression

<sup> *Token = { rule }* </sup>

**keywords**
CLASS = { 'class' }
EXTENDS = { 'extends' }
MAIN = { 'main' }
IF = { 'if' }
ELSE = { 'else' }
FOR = { 'for' }
STATIC = { 'static' }
NEW = { 'new' }
THIS = { 'this' }

**data types**
NATTYPE = { 'nat' }
BOOLTYPE = { 'bool' }

**r-values**
NATLITERAL = { '[0-9]'+ }
TRUELITERAL = { 'true' }
FALSELITERAL = { 'false' }
NUL = { 'null' }
ID = { '[a-zA-Z]''[a-zA-Z0-9]'* }

**built-in functions**
PRINTNAT = { 'printNat' }
READNAT = { 'readNat' }

**operators**
AND = { '&&' }
NOT = { '!' }
PLUS = { '+' }
MINUS = { '-' }
TIMES = { '*' }
EQUALITY = { '==' }
LESS = { '<' }
ASSIGN = { '=' }
DOT = { '.' }
INSTANCEOF = { 'instanceof' }

**delimiters**
SEMICOLON = { ';' }
LBRACE = { '{' }
RBRACE = { '}' }
LPAREN = { '(' }
RPAREN = { ')' }

**special**
ENDOFFILE = { 'EOF' }

---

## 4. Examples

### 1. example1

#### code
```
main { 0; }
```

#### output

```
MAIN LBRACE NATLITERAL(0) SEMICOLON RBRACE ENDOFFILE
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
MAIN LBRACE NATTYPE ID(x) SEMICOLON ID(x) ASSIGN NATLITERAL(0) SEMICOLON ID(x) ASSIGN ID(x) PLUS NATLITERAL(1) SEMICOLON ID(x) ASSIGN ID(x) PLUS NATLITERAL(1) SEMICOLON ID(x) ASSIGN ID(x) PLUS NATLITERAL(1) SEMICOLON ID(x) ASSIGN ID(x) PLUS NATLITERAL(1) SEMICOLON PRINTNAT LPAREN ID(x) RPAREN SEMICOLON RBRACE ENDOFFILE
```

### example3

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
CLASS ID(Factorial) EXTENDS ID(Object) LBRACE NATTYPE ID(factorial) LPAREN NATTYPE ID(n) RPAREN LBRACE IF LPAREN NATLITERAL(1) LESS ID(n) RPAREN LBRACE ID(n) TIMES ID(factorial) LPAREN ID(n) MINUS NATLITERAL(1) RPAREN SEMICOLON RBRACE ELSE LBRACE NATLITERAL(1) SEMICOLON RBRACE SEMICOLON RBRACE RBRACE MAIN LBRACE ID(Factorial) ID(f) SEMICOLON ID(f) ASSIGN NEW ID(Factorial) LPAREN RPAREN SEMICOLON PRINTNAT LPAREN ID(f) DOT ID(factorial) LPAREN READNAT LPAREN RPAREN RPAREN RPAREN SEMICOLON RBRACE ENDOFFILE
```
