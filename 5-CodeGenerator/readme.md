# Code Generator

---

## 1.Introduction

In DJ compiler, implemented in C, the code generation phase is pivotal for translating DJ source code into DISM targe code, a simplified RISC-like assembly language. Using an Abstract Syntax Tree (AST), each element of the DJ code is processed to produce corresponding DISM instructions, and these instructions are then sequentially written to a text file, where each line represents a specific operation in the target assembly language. The objective is to ensure that the generated DISM code accurately reflects the meaning and functionality of the original DJ source code. 

---

## 2. Registers

DISM has a set of 8 registers, labeled from R[0] to R[7]. These registers are used to store data temporarily during the execution of instructions.

- R[0]: This register is designated to the value 0. It will not be modified and always returns 0 when read. This is often used for operations requiring a constant zero.

- R[1] to R[4]: These are general-purpose registers. They can be used for any temporary storage needed during computation, such as holding intermediate results of arithmetic operations or storing data temporarily during the execution of a program.

- R[5]: This register is designated as the Heap Pointer (HP). It is used to manage memory allocation on the heap, which is the area of memory used for dynamic memory allocation.

- R[6]: This register serves as the Stack Pointer (SP). It points to the top of the stack, which is a region of memory used for storing local variables and function call information.

- R[7]: This register is used as the Frame Pointer (FP). It points to the base of the current stack frame, providing a stable reference point within the stack for accessing function parameters and local variables.

---

## 3. DISM Memory Layout

- DISM has 65,535 memory addresses available, but the text section (where the code resides) is not accessible during program execution.
- Address 0 is reserved for the value 0.
- The heap grows upwards from lower memory addresses, while the stack grows downwards from higher memory addresses. This ensures that both sections expand towards each other.
- Each memory cell stores a single value and each cell is 4 bytes in size.
- All memory cell is initialized to 0 before program execution
- SP: Points to the next available memory cell in the stack. The stack pointer should always be greater than the heap pointer.
- HP: Points to the next available memory cell in the heap.
- FP: Points to the current frame in the stack. The frame pointer should always be greater than the stack pointer.
- Stack and Frame Management:
  - When a method is invoked, a new frame is pushed onto the stack.
  - When returning from a method, the frame is popped from the stack.
  - Each frame collects and stores data locally relevant to a method block or main block.

#### Example of Memory Layout

The following diagram represents the memory layout during the execution of the main frame, with n main local variables and m global variables (static variables from classes):

| Address | Content | Pointer|
|-|-|-|
|65535| main local 1 (if any) | ←FP |
|65534| main local 2 | |
|65533| main local 3 | |
|...| ... | |
|65535 - n| main local n | |
|65534 - n| expression result | |
|65533 - n| 0| ←SP ↓ | 
|...| ...|  |
|m + 1| 0| ←HP ↑ | 
|m| global m | | 
|...| ...| | 
|3| global 3 | |
|2| global 2 | | 
|1| global 1 (if any) | | 
|0| 0| | 

---

## 4. Expression List

**Invariant:** Every expression is translated into DISM code that leaves the expression's result on the top of the stack

In code generation for an expression list $e_1$, $e_2$, $e_3$, ..., $e_n$, the results of $e_1$ to $e_{n-1}$ are side effect of the execution. Only the result of $e_n$ is necessary.

| Before $e_1$| Pointer | After $e_1$ | Pointer | Before $e_2$| Pointer | After $e_2$ | Pointer | ... | Before $e_n$   | Pointer | After $e_n$ | Pointer |
|-            |-        |-            |-        |-            |-        |-            |-        |-    |-               |-        |-            |-        |
|...          |         |...          |         |...          |         |...          |         | ... | ...            |         | ...         |         |      
|stack        |         |stack        |         |stack        |         |stack        |         | ... |stack           |         |stack        |         |
|             | ←SP     | $e_1$ result|         |$e_1$ result |←SP      |$e_2$ result |         | ... |$e_{n-1}$ result|←SP      |$e_n$ result |         |
|             |         |             | ←SP     |             |         |             |←SP      | ... |                |         |             |←SP      |

---

## 5. Method Calling

#### 1. Before Setting Up for Method Prologue

| Content | Pointer |
|-|-|
| ... | |
| Caller's Frame Begins | ←FP |
| ... | |
| Caller's Frame Ends | |
| | ←SP |

#### 2. Before Method Prologue

| Content | Pointer |
|-|-|
| ... | |
| Caller's Frame Begins | ←FP |
| ... | |
| Caller's Frame Ends | |
| Return Address | |
| Dynamic Caller Object Address | |
| Static Class Number containing callee | |
| Static Method Number Within that class | |
| Dynamic Method Argument | |
| | ←SP |

#### 3. After Method Prologue

| Content | Pointer | Note |
|-|-|-|
| ... | |
| Caller's Frame Begins | |
| ... | |
| Caller's Frame Ends | |
| Return Address |  ←FP  | FP |
| Dynamic Caller Object Address | | FP - 1 |
| Static Class Number containing callee | | FP - 2 |
| Static Method Number Within that class | | FP - 3 |
| Dynamic Method Argument | | FP - 4 |
| Method local 1 (if any) | | FP - 5 |
| ... | | |
| Method local n | | FP - 5 - n + 1 |
| Caller's FP | | SP + 1 |
| | ←SP |

#### 4. Before Method Epilogue

| Content | Pointer | Note |
|-|-|-|
| ... | |
| Caller's Frame Begins | |
| ... | |
| Caller's Frame Ends | |
| Return Address |  ←FP  | FP |
| Dynamic Caller Object Address | | FP - 1 |
| Static Class Number containing callee | | FP - 2 |
| Static Method Number Within that class | | FP - 3 |
| Dynamic Method Argument | | FP - 4 |
| Method local 1 (if any) | | FP - 5 |
| ... | | |
| Method local n | | FP - 5 - n + 1 |
| Caller's FP | | SP + 2 |
| Method's Return Value | | SP + 1 |
| | ←SP |

#### 5. After Method Epilogue

| Content | Pointer | Note |
|-|-|-|
| ... | |
| Caller's Frame Begins | ←FP |
| ... | |
| Caller's Frame Ends | |
| Method's Return Value | | SP + 1 |
| | ←SP |

---

## 6. VTable

#### Build VTable Structure

1. **Iterate Over Static Class Methods**: The outer loop iterates over each method (`staticMethodNum`) defined in the static class (`staticClassNum`). The static class represents the type known at compile time.

2. **Initialize Search**: For each method in the static class, it initializes a search to find a corresponding method in the dynamic class (`type`) or its superclasses. The dynamic class represents the actual type of the object at runtime.

3. **Search for Method in Dynamic Type Hierarchy**: The `while` loop iterates through the type hierarchy starting from the dynamic class, moving up to its superclasses, looking for a method that matches the current static class method by name.

   - **Method Matching**: Within the `for` loop, it compares the method names of the static class and the current type (`type`) in the hierarchy. If a match is found, it means the dynamic type or one of its superclasses implements a method with the same name as the static class method.

   - **Append to VTable**: Upon finding a match, `appendToVTable` is called to add an entry to the VTable. This entry maps the static class method to the found method implementation in the dynamic class hierarchy. The parameters include the static class, the static method number, the dynamic caller, the dynamic class, and the dynamic method number.

4. **Method Not Found in Hierarchy**: If no matching method is found in the dynamic class or its superclasses (`found` remains false), the `else` block appends an entry to the VTable that maps the static class method to itself. This is a fallback mechanism, ensuring that there's always a method to call, even if it's just the implementation in the static class.

5. **Repeat**: The process repeats for each method in the static class, building a complete VTable that facilitates dynamic method dispatch based on the actual runtime type of objects, mimicking polymorphism in object-oriented programming.

#### Example
```
class C1 extends Object {
    nat f(C1 o) {o.g(3);}
    nat g(nat n) {n;}
}
class C2 extends C1 {
    nat g(nat n) {n+2;}
}
class C3 extends Object {
    nat g(nat n) {n+5;}
}
class C4 extends Object {
    nat g(nat n) {n+10;}
}
class C5 extends Object {}
class C6 extends C4 {}
class C7 extends C4 {
    nat f(nat i) {i;}
}
main {0;}
```

| Static Class Num | Static Method Num | Dynamic Caller | → | Dynamic Class Num | Dynamic Method Num |
|-|-|-|-|-|-|
| 1                | 0                 | 1              | | 1                 | 0                  |
| 1                | 0                 | 2              | | 1                 | 0                  |
| 1                | 1                 | 1              | | 1                 | 1                  |
| 1                | 1                 | 2              | | 2                 | 0                  |
| 2                | 0                 | 2              | | 2                 | 0                  |
| 3                | 0                 | 3              | | 3                 | 0                  |
| 4                | 0                 | 4              | | 4                 | 0                  |
| 4                | 0                 | 6              | | 4                 | 0                  |
| 4                | 0                 | 7              | | 4                 | 0                  |
| 7                | 0                 | 7              | | 7                 | 0                  |

---

## 7. ITable

#### Example
```
class C1 extends Object {
    nat f;
}
class C2 extends C1 {
    nat g;
}
class C3 extends C2 {
    nat h;
}
main {
    C1 o;
    o = new C3(); // statically C1, dynamically C3
    if (o instanceof C3) {printNat(10);}
    else {printNat(0);};
}
```

| Dynamic Class Num | Static Class Num | → | Resulted Class Num |
|-|-|-|-|
| 1 | 1 | | 1 |
| 1 | 2 | | 0 |
| 1 | 3 | | 0 |
| 2 | 1 | | 1 |
| 2 | 2 | | 1 |
| 2 | 3 | | 0 |
| 3 | 1 | | 1 |
| 3 | 2 | | 1 |
| 3 | 3 | | 1 |

---

## 8. Code Generation (Expression - Write to File)

The code generation function takes an abstract syntax tree (AST) node of a given expression, and the class and/or method where the expression is located. It writes the DISM code on a text file.

#### 0. Aux Function
- **DecSP**
   1. R[1] ← 1
   2. SP ← SP - R[1]
   3. branch to L if HP < SP 
   4. print error msg
   5. halt
   6. L:
- **IncSP**
   1. R[1] ← 1
   2. SP ← SP + R[1] 
- **CheckNullDereference**
   1. R[1] ← M[SP+1]
   2. branch to L if R[0] < R[1]
   3. print error msg
   4. halt
   5. L:
- **AllocateHeapVars**
   1. R[1] ← 1
   2. R[1] ← HP + R[1]
   3. R[2] ← numHeapVars
   4. R[1] ← R[1] + R[2] 
   5. R[2] ← MAX_DISM_ADDR
   6. branch to L1 if R[1] < R[2]
   7. print error msg
   8. halt
   9. L1: 
   10. R[1] ← 1 
   11. R[2] ← 0
   12. R[3] ← numHeapVars
   13. L2:
   14. branch to L3 if R[2] == R[3]
   15. M[HP] ← R[0]
   16. HP ← HP + R[1]
   17. R[2] ← R[2] + R[1]
   18. jmp L2
   19. L3:
- **AllocateLocalVars**
   1. R[1] ← 1
   2. R[2] ← 0
   3. R[3] ← numLocals
   4. L1:
   5. branch to L2 if R[2] == R[3]
   6. M[SP] ← R[0]
   7. R[2] ← R[2] + R[1]
   8. jmp L1
   9. L2:

#### 1. Nat Type
- If the expression type is **nat-literal-expr**:
  1. R[1] ← %d
  2. M[SP] ← R[1]
  3. DecSP

#### 2. Bool Type - True
- If the expression type is **true-literal-expr**:
  1. R[1] ← 1
  2. M[SP] ← R[1]
  3. DecSP

#### 3. Bool Type - True
- If the expression type is **false-literal-expr**:
  1. M[SP] ← R[0]
  2. DecSP

#### 4. Null
- If the expression type is **null-expr**:
  1. M[SP] ← R[0]
  2. DecSP

#### 5. `new C()`
- If the expression type is **new-expr**:
  1. AllocateHeapVars
  2. R[2] ← staticClassNum
  3. M[HP] ← staticClassNum
  4. M[SP] ← HP
  5. R[1] ← 1
  6. HP ← HP + R[1]
  7. DecSP

#### 6. `this` in Class `C`
- If the expression type is **this-expr**:
  1. R[1] ← M[FP-1]
  2. M[SP] ← R[1]
  3. DecSP

#### 7. `readNat()`
- If the expression type is **read-expr**:
  1. read into R[1]
  2. M[SP] ← R[1]

#### 8. `printNat(E)`
- If the expression type is **print-nat**:
  1. Call codeGen on E
  2. R[1] ← M[SP+1]
  3. print from R[1]

#### 9. `for(E1;E2;E3) {Es}`
- If the expression type is **for-expr**:
  1. Call codeGen on E1
  2. IncSP
  3. LOOP:
  4. Call codeGen on E2 
  5. R[1] ← M[SP+1]
  6. branch to END if R[1] == R[0]
  7. IncSP
  8. Call codeGen on Es
  9. IncSP
  10. Call codeGen on E3
  11. IncSP
  12. jmp LOOP
  13. END:

#### 10. `if(E) {Es1} else {Es2}`
- If the expression type is **if-then-else-expr**:
  1. Call codeGen on E
  2. R[1] ← M[SP+1]
  3. branch to ELSE if R[1] == R[0]
  4. IncSP
  5. Call codeGen on Es1
  6. jmp END
  7. ELSE:
  8. IncSP
  9. Call codeGen on Es2
  10. END:

#### 11. `E instanceof ID` 
- If the expression type is **instanceof-expr**:
  1. R[1] ← retAddr
  2. M[SP] ← R[1]
  3. DecSP
  4. R[1] ← staticClassNum
  5. M[SP] ← R[1]
  6. DecSP
  7. Call codeGen on E
  8. R[1] ← M[SP+1] # dynamic class address
  9. branch to NULL if R[1] == R[0]
  10. R[1] ← M[R[1]] # dynamic class number
  11. M[SP+1] ← R[1] 
  12. jmp ITABLE
  13. NULL:
  14. M[SP+3] ← R[0]
  15. retAddr: 

#### 12. `E1 && E2`
- If the expression type is **and-expr**:
  1. Call codeGen on E1
  2. R[1] ← M[SP+1]
  3. branch to FALSE if R[1] == R[0]
  4. IncSP
  5. Call codeGen on E2
  6. R[1] ← M[SP+1]
  7. branch to FALSE if R[1] == R[0] 
  8. R[1] ← 1
  9. M[SP+1] ← R[1]
  10. jmp END
  11. FALSE:
  12. M[SP+1] ← R[0]
  13. END:

#### 13. `!E`
- If the expression type is **not-expr**:
  1. Call codeGen on E
  2. R[1] ← M[SP+1]
  3. branch to FALSE if R[1] == R[0]
  4. M[SP+1] ← R[0]
  5. jmp END
  6. FALSE:
  7. R[1] ← 1
  8. M[SP+1] ← R[1]
  9. END:

#### 14. `E1 < E2`
- If the expression type is **less-than-expr**:
  1. Call codeGen on E1
  2. Call codeGen on E2
  3. R[1] ← M[SP+2]
  4. R[2] ← M[SP+1]
  5. branch to TRUE if R[1] < R[2]
  6. M[SP+2] ← R[0]
  7. jmp END
  8. TRUE:
  9. R[1] ← 1
  10. M[SP+2] ← R[1]
  11. END:
  12. IncSP

#### 15. `E1 == E2`
- If the expression type is **equality-expr**:
  1. Call codeGen on E1
  2. Call codeGen on E2
  3. R[1] ← M[SP+2]
  4. R[2] ← M[SP+1]
  5. branch to TRUE if R[1] == R[2]
  6. M[SP+2] ← R[0]
  7. jmp END
  8. TRUE:
  9. R[1] ← 1
  10. M[SP+2] ← R[1]
  11. END:
  12. IncSP

#### 16. `E1 * E2`
- If the expression type is **times-expr**:
  1. Call codeGen on E1
  2. Call codeGen on E2
  3. R[1] ← M[SP+2]
  4. R[2] ← M[SP+1]
  5. R[1] ← R[1] * R[2]
  6. M[SP+2] ← R[1]
  7. IncSP

#### 17. `E1 - E2`
- If the expression type is **minus-expr**:
  1. Call codeGen on E1
  2. Call codeGen on E2
  3. R[1] ← M[SP+2]
  4. R[2] ← M[SP+1]
  5. R[1] ← R[1] - R[2]
  6. M[SP+2] ← R[1]
  7. IncSP 

#### 18. `E1 + E2`
- If the expression type is **plus-expr**:
  1. Call codeGen on E1
  2. Call codeGen on E2
  3. R[1] ← M[SP+2]
  4. R[2] ← M[SP+1]
  5. R[1] ← R[1] + R[2]
  6. M[SP+2] ← R[1]
  7. IncSP  

#### 19. `ID = E`
- If the expression type is **assign-expr**:
  - Call codeGen on E
  - If ID is main local
    1. Get expression address on Main
    2. R[1] ← M[SP+1]
    3. R[2] ← mainLocalAddress
    4. M[2] ← R[1]
  - If ID is global variable / static variable
    1. Get static field address
    2. R[1] ← M[SP+1] 
    3. R[2] ← staticFieldAddress
    4. M[2] ← R[1]
  - If ID is method local or parameter
    1. Get method field offset (0: parameter; >0: local)
    2. R[1] ← FP
    3. R[2] ← 4
    4. R[1] ← R[1] - R[2] # address of parameter  
    5. R[2] ← offset
    6. R[1] ← R[1] - R[2] # address of target
    7. R[2] ← M[SP+1]
  - If ID is field of this object
    1. Get object field offset
    2. R[1] ← M[FP-1] # load this
    3. R[2] ← M[SP+1]
    4. R[3] ← offset
    5. R[1] ← R[1] - R[3]
    6. M[R[1]] ← R[2]

#### 20. `E1.ID = E2`
- If the expression type is **dot-assign-expr**:
  - Call codeGen on E2
  - If ID is global variable / static variable
    1. Get static field address
    2. R[1] ← M[SP+1] 
    3. R[2] ← staticFieldAddress
    4. M[2] ← R[1]
  - If ID is field of this object
    1. Get object field offset
    2. Call codeGen on E1
    3. CheckNullDereference
    4. R[1] ← M[SP+1] # load this 
    5. R[2] ← M[SP+2] # load result
    6. R[3] ← offset
    7. R[1] ← R[1] - R[3]
    8. M[R[1]] ← R[2]
    9. IncSP

#### 21. `ID`
- If the expression type is **id-expr**:
  - If ID is main local
    1. Get expression address on Main
    2. R[1] ← M[mainLocalAddress]
    3. M[SP] ← R[1]
    4. DecSP
  - If ID is global variable / static variable
    1. Get static field address
    2. R[1] ← M[staticFieldAddres]
    3. M[SP] ← R[1]
    4. DecSP
  - If ID is method local or parameter
    1. Get method field offset (0: parameter; >0: local)
    2. R[1] ← FP
    3. R[2] ← 4
    4. R[1] ← R[1] - R[2] # address of parameter  
    5. R[2] ← offset
    6. R[1] ← R[1] - R[2] # address of target
    7. R[1] ← M[R[1]] # load local variable
    8. M[SP] ← R[1]
    9. DecSP
  - If ID is field of this object
    1. Get object field offset
    2. R[1] ← M[FP-1] # load this
    3. R[2] ← offset
    4. R[1] ← R[1] - R[2]
    5. R[1] ← M[R[1]] # load field
    6. M[SP] ← R[1]
    7. DecSP

#### 22. `E.ID`
- If the expression type is **dot-id-expr**:
  - If ID is global variable / static variable
    1. Get static field address
    2. R[1] ← M[staticFieldAddress]
    3. M[SP] ← R[1]
    4. DecSP
  - If ID is field of this object
    1. Get object field offset
    2. Call codeGen on E
    3. CheckNullDereference
    4. R[1] ← M[SP+1] # load this
    5. R[3] ← offset
    6. R[1] ← R[1] - R[3]
    7. R[1] ← M[R[1]] # load field
    8. M[SP+1] ← R[1]

#### 22. `ID(E)`
- If the expression type is **method-call-expr**:
  1. R[1] ← retAddr
  2. M[SP] ← R[1]
  3. DecSP 
  4. R[1] ← M[FP-1] # load this
  5. M[SP] ← R[1]
  6. DecSP
  7. R[1] ← staticClassNum
  8. M[SP] ← R[1]
  6. DecSP
  7. R[1] ← staticMethodNum
  8. M[SP] ← R[1]
  6. DecSP  
  7. Call codeGen on E
  8. jmp VTABLE
  9. retAddr:

#### 23. `E1.ID(E2)`
- If the expression type is **dot-method-call-expr**:
  1. R[1] ← retAddr
  2. M[SP] ← R[1]
  3. DecSP 
  4. Call codeGen on E1
  5. CheckNullDereference
  6. R[1] ← staticClassNum
  7. M[SP] ← R[1]
  8. DecSP
  9.  R[1] ← staticMethodNum
  10. M[SP] ← R[1]
  11. DecSP  
  12. Call codeGen on E2
  13. jmp VTABLE
  14. retAddr: 

---

## 9. Code Generation (C code)

#### 1. Get expression address on Main
- input
  - AST node: containing the expression
1. idName = getName(node)
2. for i in [0, numMainBlockLocals)
   1. if idName == mainBlockST[i].varName
      1. return MAX_DISM_ADDR - i

#### 2. Get static field address
- input 
  - staticClassNum: Class number in classesST
  - staticMemberNum: Member number in the class
1. staticFieldAddress = 0
2. for i in [1, staticClassNum)
   1. staticFieldAddress += classesST[i].numStaticVars
3. staticFieldAddress += staticMemberNum
4. return staticFieldAddress + 1

#### 3. Get method field offset (-1: parameter; >-1: local) 
- input
  - AST node: containing the expression
  - staticClassNum: Class number in classesST
  - staticMemberNum: Member number in the class
1. idName = getName(node)
2. if idName == classesST[staticClassNum].methodList[staticMemberNum].paramName
   1. return 0
3. for i in [0, classesST[staticClassNum].methodList[staticMemberNum].numLocals)
   1. if idName == classesST[staticClassNum].methodList[staticMemberNum].localST[i].varName
      1. return i + 1

#### 4. Get object field offset
- input
  - staticClassNum: Class number in classesST
  - staticMemberNum: Member number in the class
1. numField = 0
2. while staticClassNum != 0
   1. numFields += classesST[staticClassNum].numVar
   2. staticClassNum = classesST[staticClassNum].superclass 
3. return numField + staticMemberNum + 1 

---

## 10. Code Generation (Body - Write to Files)

#### 0. Output Structure
1. GenPrologueMain
2. GenBodyMain
3. GenEpilogueMain
4. GenMethods
5. GenITable
6. GenVTable 

#### 1. GenPrologueMain
1. initialize R[0] to 0 
2. initialize FP to MAX_DISM_ADDR
3. initialize SP to MAX_DISM_ADDR
4. initialize HP to 1
5. AllocateLocalVars
6. AllocateHeapVars

#### 2. GenBodyMain
1. call codeGen on MainExprs

#### 3. GenEpilogueMain
1. halt

#### 4. GenMethods
1. for i in [1, numClasses)
   1. for j in [0, classesST[i].numMethods)
      1. GenPrologue(i, j)
      2. GenBody(i, j)
      3. GenEpilogue(i, j)
2. GenPrologue
   - input:
     - staticClassNum
     - staticMethodNum
   1. M[HP] ← SP
   2. AllocateLocalVars
   3. M[SP] ← FP 
   4. DecSP
   5. R[1] ← M[HP] # load old SP
   6. R[2] ← 5
   7. FP ← R[1] + R[2] # point FP to return address
3. GenBody
   - input:
     - staticClassNum
     - staticMethodNum
   1. call codeGen on classesST[staticClassNum].methodList[staticMethodNum].bodyExpr
4. GenEpilogue 
   - input:
     - staticClassNum
     - staticMethodNum
   1. R[1] ← M[SP+1] # load return value
   2. R[2] ← M[SP+2] # load old FP
   3. R[3] ← M[FP] # load the return address
   4. SP ← FP # pop frame
   5. M[SP] ← R[1]
   6. FP ← R[2]
   7. DecSP
   8. jmp to R[3]

#### 5. GenITable
1. ITABLE:
2. R[1] <- M[SP+1] # load the dynamic class number 
3. R[2] <- M[SP+2] # load the static class number
4. R[3] <- M[SP+3] # load the return address
5. It iterates over all classes to emit instructions that check if the dynamic class number matches any defined class, branching accordingly. It then verifies if the static class number matches any class in the hierarchy, setting the return value based on whether a match is found.

#### 6. GenVTable
1. VTABLE:
2. R[1] <- M[SP+3] # static class number
3. R[2] <- M[SP+2] # static method number
4. R[3] <- M[SP+4] # dynamic class address
5. R[3] <- M[R[3]] # load dynamic class number
6. It first iterates over all classes to compare and branch based on static class numbers. It includes error handling for invalid method calls by setting an error code and halting the program. Subsequently, it iterates over classes and their methods to dispatch methods based on the method number and dynamic caller, marking sections with specific labels and handling errors similarly.