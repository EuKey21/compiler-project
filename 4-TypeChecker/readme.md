# Semantic Analysis

---

## 1. Introduction

In DJ, the type checker is a component designed to ensure the semantic correctness of the source code by verifying that expressions and operations adhere to the rules of the type system. It operates on the abstract syntax tree (AST) generated from source code, systematically analyzing each node to determine and validate the types of expressions. The type checker supports a type system, including nat (natural numbers), bool (Boolean), null, and user-defined classes with inheritance hierarchies.

---

## 2. Type Checking Notes

- A list of expressions has the type of the final expression.
- DJ methods can redeclare class variable names locally or as parameters, but blocks cannot redeclare names within themselves.
- `null` has type "any object," a subtype of every class.
- `new C()` has type C.
- Equality (`E1 == E2`) is well-typed if E1 and E2 are either subtypes of each other and it results in type `bool`.
- An if-then-else expression is well-typed if the condition is `bool`, and both branches have the same type. The type of the expression matches the branches' type or the join of the branches' object types.
- A for-loop is well-typed if initialization, conclusion, and body are well-typed, and the test is `bool`.
- A method can accept parameter that are subtypes of the declared parameter type.
- A method can return a value that is a subtype of its declared return type, but the return type remains the declared type.
- Assignment (`ID=E`) is well-typed if ID is a valid variable and E's type is a subtype of ID's type.
- `instanceof` expressions are well-typed if E is an object type and ID is a valid class name.
- `this` must be used within its class declaration and has the type of that class.
- DJ class hierarchies cannot contain cycles or self-superclass.
- Overriding methods can have different parameter names from overridden methods.
- The main block must be well-typed but can have any type.
- All variables must be declared before use.

---

## 3. Validate Symbol Table

1. **Check Class Names**:
   - Ensure all class names are unique.

2. **Check Types Validity**:
   1. Validate types in `mainBlockST`:
      - Ensure all types are valid.
   2. Validate types in `classesST`:
      - Check for invalid superclasses.
      - Ensure a class does not have itself as a superclass.
      - Verify static variable types.
      - Verify non-static variable types.
      - Validate methods:
         1. Check for invalid parameter types.
         2. Check for invalid return types.
         3. Check for invalid local variable types.
3. **Check Class Hierarchy**:
      - Ensure there are no cycles in the class hierarchy.
      - A class is valid if it can be traced to the `Object` class within `numClasses` steps.
4. **Check Variable Uniqueness**:
      - Ensure all static and non-static variable names are unique within each class:
         1. Static variables must be unique within a class.
         2. Static variables must be unique compared to non-static variables.
         3. Non-static variables must be unique within a class.
         4. Non-static variables must be unique compared to static variables.
         5. Static and non-static variable names must be unique compared to those in superclasses.
5. **Check Declared Methods in Classes**:
      1. Ensure all method names are unique within a class.
      2. Verify that methods in superclasses with the same name have the same signature (parameter and return types).
      3. Ensure parameter and local variable names are unique within methods:
         1. Parameter names must be unique compared to local variable names.
         2. Local variable names must be unique within each other.
      4. Ensure the method's expression list is well-typed with a return type that is a subtype of the declared return type.
6. **Check Main Block**:
      - Ensure there are no duplicate variable names in the main block.
      - Ensure the main block's expression list is well-typed.

---

## 4. isSubtype

The `isSubtype` function determines if one type is a subtype of another type. The function returns 1 (true) if sub is a subtype of super, and 0 (false) otherwise.

1. **Reflexivity Check**:
   - If `sub` and `super` are the same, the function returns 1, as a type is always a subtype of itself.

2. **NAT Type Check**:
   - If either `sub` or `super` is -1 (representing NAT), the function returns 0, except when they are the same type.

3. **BOOL Type Check**:
   - If either `sub` or `super` is -2 (representing BOOL), the function returns 0, except when they are the same type.

4. **NULL Type Check**:
   - If `sub` is -3 (representing NULL), and `super` is a non-negative integer (object type), the function returns 1, indicating NULL is a subtype of all object types. Otherwise, it returns 0.

5. **Object Type Check**:
   - For non-negative integer types (object types), the function checks if `sub` is a subtype of `super` through inheritance. It does this by traversing the inheritance chain, starting from `sub` and following the `superclass` field in the `classesST` array until `sub` matches `super` or until the traversal exceeds `numClasses`.

6. **Default Case**:
   - If none of the conditions are met, the function returns 0, indicating `sub` is not a subtype of `super`.

---

## 5. Join

The `join` function a common type between two given types, `type1` and `type2`.

1. **Check if `type1` is a Subtype of `type2`**:
   - It first checks if `type1` is a subtype of `type2` by calling `isSubtype(type1, type2)`.
   - If `type1` is a subtype of `type2`, it returns `type2` as the common type.

2. **Check if `type2` is a Subtype of `type1`**:
   - If `type1` is not a subtype of `type2`, the function checks if `type2` is a subtype of `type1` by calling `isSubtype(type2, type1)`.
   - If `type2` is a subtype of `type1`, it returns `type1` as the common type.

3. **Recursive Call with Superclass of `type1`**:
   - If neither type is a subtype of the other, the function attempts to find a common type by moving up the inheritance hierarchy of `type1`.
   - It does this by making a recursive call to itself with `type1.superclass` and `type2`.

---

## 6. Type checker

The type check function takes an abstract syntax tree (AST) node of a given expression, and the class and/or method where the expression is located. It determines and returns the type information of the expression.

#### 1. Nat Type
- If the expression type is **nat-literal-expr**, return -1.

#### 2. Bool Type
- If the expression type is **true-literal-expr** or **false-literal-expr**, return -2.

#### 3. Null Type
- If the expression type is **null-expr**, return -3.

#### 4. `new C()` is Type `C`
- If the expression type is **new-expr**:
  - Get the class name from the AST node.
  - Find the type number in `classesST` by comparing the class name to those in the symbol table.
  - Return the type number.

#### 5. `this` in Class `C` is Type `C`
- If the expression type is **this-expr**:
  - Return the class number where the expression is located.

#### 6. `readNat()` is Type `nat`
- If the expression type is **read-expr**, return -1.

#### 7. `printNat(E)` is Type `nat`
- If the expression type is **print-nat**:
  - Call type check on expression `E`.
  - If `E` is -1, return -1.

#### 8. `for(E1;E2;E3) {Es}` is Type `nat`
- If the expression type is **for-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - Call type check on expression `E3`
  - Call type check on expression list`Es`
  - If `E2` is -2, return -1.

#### 9. `if(E) {Es1} else {Es2}` is Type of `join(Es1, Es2)`
- If the expression type is **if-then-else-expr**:
  - Call type check on expression `E`
  - Call type check on expression list `Es1`
  - Call type check on expression list`Es2`
  - If `E` is -2, call `join` on the types of `Es1` and `Es2`, and return the resulting type.

#### 10. `E instanceof ID` is Type `bool`
- If the expression type is **instanceof-expr**:
  - Call type check on expression `E`.
  - If `E` > 0:
    - Get the name of `ID`.
    - Compare the name to the user defined classes.
    - If a match is found, return type -2.

#### 11. `E1 && E2` is Type `bool`
- If the expression type is **and-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - If both `E1` and `E2` are -2, return -2.

#### 12. `!E` is Type `bool`
- If the expression type is **not-expr**:
  - Call type check on expression `E`.
  - If `E` is -2, return -2.

#### 13. `E1 < E2` is Type `bool`
- If the expression type is **less-than-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - If both `E1` and `E2` are -1, return -2.

#### 14. `E1 == E2` is Type `bool`
- If the expression type is **equality-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - If `E1` is a subtype of `E2` or vice versa, return -2.

#### 15. `E1 * E2` is Type `nat`
- If the expression type is **times-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - If both `E1` and `E2` are -1, return -1.

#### 16. `E1 - E2` is Type `nat`
- If the expression type is **minus-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - If both `E1` and `E2` are -1, return -1.

#### 17. `E1 + E2` is Type `nat`
- If the expression type is **plus-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - If both `E1` and `E2` are -1, return -1.

#### 18. `ID = E` is Type of `E`
- If the expression type is **assign-expr**:
  - Get the name of `ID`.
  - Call type check on expression `E`.
  - Find the location of `ID` in the symbol table.
    - Check main block, method parameter, method locals, class (static and non-static fields) and its superclass (until reaching `Object`) for `ID` -- in this strict order.
    - If `ID` is found and `E` is a subtype of `ID` type, return `E` type.

#### 19. `E1.ID = E2` is Type of `E2`
- If the expression type is **dot-assign-expr**:
  - Get the name of `ID`.
  - Call type check on expressions `E1` and `E2`.
  - If `E1` > 0, check class `E1` (static and non-static fields) and its superclass (until reaching `Object`) for `ID`.
  - If `ID` is found and `E2` is a subtype of `ID` type, return `E2` type.

#### 20. `ID` is Type of `ID`
- If the expression type is **id-expr**:
  - Get the name of `ID`.
  - Find the location of `ID` in the symbol table 
    - Check main block, method parameter, method locals, class (static and non-static fields) and its superclass (until reaching `Object`) for `ID` -- in this strict order.
  - If `ID` is found, return `ID` type.

#### 21. `E.ID` is Type of `ID`
- If the expression type is **dot-id-expr**:
  - Get the name of `ID`.
  - Call type check on expression `E`.
  - If `E` > 0, check class `E` (static and non-static fields) and its superclass (until reaching `Object`) for `ID`.
  - If `ID` is found, return `ID` type.

#### 22. `ID(E)` is Type of Method Return Value
- If the expression type is **method-call-expr**:
  - Get the name of `ID`.
  - Call type check on expression `E`.
  - Check `ID(E)` is not in main.
  - Check `ID` as a method first in the current class and then its superclasses (until reaching `Object`).
  - If found, return the type of the method return value.

#### 23. `E1.ID(E2)` is Type of Method Return Value
- If the expression type is **dot-method-call-expr**:
  - Call type check on expression `E1`
  - Call type check on expression `E2`
  - Get the name of `ID`.
  - If `E1` > 0, check class `E1` for the method.
  - Check `ID` as a method first in the `E1` and then its superclasses (until reaching `Object`).
  - If found, return the type of the method return value.




