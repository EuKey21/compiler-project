# Diminished Java

---

## 1. Introduction

DJ (Diminished Java) is a simplified programming language closely resembling Java, designed with the dual objectives of being a complete object-oriented programming language while maintaining simplicity by including only core features. Developed to be Turing complete, DJ allows the expression of any algorithm and can be compiled straightforwardly. With features such as automatic initialization of variables, the absence of constructor methods, and a built-in new expression for object creation, DJ presents a challenge to an introductory compiler.

This compiler is a course project. Due to restrictions, the code cannot be published. However, I will describe how I created the compiler in this repository.

---

## 2. Acknowledgement

I would like to extend my gratitude to Dr. Jay Ligatti for his invaluable guidance in this compiler project. His insights and teachings have been instrumental in shaping my understanding and approach to this project. For more information on Dr. Ligatti and DJ, please visit his [webpage](https://cse.usf.edu/~ligatti/..)

---

## 3. Requirements

- Programming Language: C89
- Operating System: Red Hat Enterprise Linux Server release 7.9 (Maipo)
- Software:
  - gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44)
  - flex 2.5.37
  - bison (GNU Bison) 3.0.4

---

## 4. Compilation

> flex dj.l
> bison -v dj.y
> sed -i '/extern YYSTYPE yylval/d' dj.tab.c
> gcc dj.tab.c ast.c symtbl.c typecheck.c codegen.c -o djc

---

## 5. Source Code Structures

- DJ program must be in a single file.
- The file starts with class declarations and ends with a main block.
- Class declaration:
  - Starts with `class` keyword.
  - Followed by class name.
  - Followed by `extends` keyword and superclass name.
  - Enclosed in braces `{}`.
  - Contains:
    - Static variable declarations (with `static` keyword).
    - Regular variable declarations.
    - Method declarations.
- Static-variable declaration:
  - Starts with `static` keyword.
  - Followed by type (nat, bool, or class name).
  - Followed by variable name and a semicolon.
  - Example: `static nat i;`
- Regular variable declaration:
  - Same as static-variable declaration but without `static` keyword.
- Method declaration:
  - Starts with return type name.
  - Followed by method name.
  - Enclosed in parentheses `()` with parameter type and name.
  - Followed by variable-expression block.
- Variable-expression block:
  - Enclosed in braces `{}`.
  - Contains:
    - Regular variable declarations.
    - Nonempty sequence of expressions, each followed by a semicolon.
- Main block:
  - Starts with `main` keyword.
  - Followed by variable-expression block.

---

## 6. Features

- **Case Sensitivity**
  - Keywords and identifiers are case sensitive (e.g., "Class" is not the same as "class").
- **Identifiers**
  - Must begin with a letter.
  - Can contain only digits (0-9) and ASCII upper- and lower-case English letters.
- **Natural-number Literals**
  - All numbers have `nat` type and must be natural numbers (0, 1, 2, ...).
  - Leading zeroes are allowed (e.g., 00005 is valid).
- **The Object Class**
  - A class called `Object` always exists.
  - `Object` extends no other class and contains no members (neither fields nor methods).
- **Recursion**
  - Methods and classes may be mutually recursive.
  - Example: Class C1 may define a variable of type C2, and C2 may define a variable of type C1.
- **Data Initialization**
  - Natural-number variables and fields initialize to 0.
  - Boolean variables and fields initialize to `false`.
  - Object variables and fields initialize to `null`.
- **Static Fields**
  - Static fields are class variables, meaning one copy exists for the entire class.
  - Non-static fields have one copy per instance of the class.
  - Static variables act as globals, accessible throughout the program.
- **Inheritance**
  - Classes inherit all fields and methods from superclasses.
  - Subclasses may override methods but not variable fields.
  - Overridden methods must have identical parameter and return types.

---

## 7. Expression List

- `E`: expression
- `Es`: expression list
- `ID`: identifier

1. **new-expr** `new C()`
2. **this-expr** `this`
3. **read-expr** `readNat()`
4. **print-nat** `printNat(E)`
5. **for-expr** `for(E1;E2;E3) {Es}`
6. **if-then-else-expr** `if(E) {Es1} else {Es2}`
7. **instanceof-expr** `E instanceof ID`
8. **and-expr** `E1 && E2`
9. **not-expr** `!E`
10. **less-than-expr** `E1 < E2`
11. **equality-expr** `E1 == E2`
12. **times-expr** `E1 * E2`
13. **minus-expr** `E1 - E2`
14. **plus-expr** `E1 + E2`
15. **assign-expr** `ID = E`
16. **dot-assign-expr** `E1.ID = E2`
17. **id-expr** `ID`
18. **dot-id-expr** `E.ID` 
19. **method-call-expr** `ID(E)`
20. **dot-method-call-expr** `E1.ID(E2)`
