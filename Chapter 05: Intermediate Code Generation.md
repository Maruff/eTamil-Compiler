
---

## Chapter 5: Intermediate Code Generation

### Overview

Intermediate code generation is the fourth phase of the compilation process. This phase translates the abstract syntax tree (AST) into an intermediate representation (IR) that is easier to manipulate and optimize than the high-level source code and more abstract than machine code. This chapter explores the principles of intermediate code generation, various intermediate representations, and provides a practical implementation using Rust for the eTamil language.

### Purpose of Intermediate Code

Intermediate code serves as a bridge between the high-level source code and the target machine code. The primary purposes of intermediate code include:

- **Portability**: Intermediate code can be used to generate machine code for different architectures.
- **Optimization**: It allows for more effective optimization techniques to be applied before generating machine code.
- **Simplification**: It simplifies the code generation process by providing a more uniform representation of the program.

### Intermediate Representations

Several types of intermediate representations are used in compiler design. The choice of IR depends on the specific requirements of the compiler and the optimizations to be applied. Common intermediate representations include:

- **Three-Address Code (TAC)**: A simple and widely used IR where each instruction contains at most three addresses (operands).
- **Abstract Syntax Trees (ASTs)**: A hierarchical representation of the program structure, often used as the initial IR.
- **Control Flow Graphs (CFGs)**: A graphical representation of the program's control flow, useful for certain optimizations.
- **Static Single Assignment (SSA) Form**: An IR where each variable is assigned exactly once, simplifying data flow analysis and optimization.

### Three-Address Code (TAC)

Three-address code is a popular choice for intermediate representation due to its simplicity and ease of manipulation. Each instruction in TAC typically takes the form of `x = y op z`, where `x`, `y`, and `z` are variables or constants, and `op` is an operator.

#### Example TAC Instructions

```plaintext
t1 = 5
t2 = 3
t3 = 2
t4 = t3 - 1
t5 = t2 * t4
t6 = t1 + t5
a = t6
```

### Rust Implementation

In this section, we will implement intermediate code generation for the eTamil language using Rust. The implementation will translate the AST into three-address code (TAC).

#### Defining TAC Instructions

First, we define the structure for TAC instructions.

```rust
#[derive(Debug)]
pub enum TACInstruction {
    Assign { dest: String, src: String },
    BinaryOp { dest: String, op: char, left: String, right: String },
    LoadConst { dest: String, value: i64 },
}

#[derive(Debug)]
pub struct TAC {
    instructions: Vec<TACInstruction>,
    temp_count: usize,
}

impl TAC {
    pub fn new() -> Self {
        TAC {
            instructions: Vec::new(),
            temp_count: 0,
        }
    }

    pub fn new_temp(&mut self) -> String {
        self.temp_count += 1;
        format!("t{}", self.temp_count)
    }

    pub fn add_instruction(&mut self, instruction: TACInstruction) {
        self.instructions.push(instruction);
    }
}
```

#### Implementing Intermediate Code Generation

Next, we implement the intermediate code generator that translates the AST into TAC.

```rust
pub struct CodeGenerator {
    tac: TAC,
}

impl CodeGenerator {
    pub fn new() -> Self {
        CodeGenerator {
            tac: TAC::new(),
        }
    }

    pub fn generate(&mut self, node: &ASTNode) -> Result<String, String> {
        match node {
            ASTNode::Program(statements) => {
                for statement in statements {
                    self.generate(statement)?;
                }
                Ok(String::new())
            }
            ASTNode::Assignment { identifier, expression } => {
                let expr_result = self.generate(expression)?;
                self.tac.add_instruction(TACInstruction::Assign {
                    dest: identifier.clone(),
                    src: expr_result,
                });
                Ok(identifier.clone())
            }
            ASTNode::Expression { left, operator, right } => {
                let left_result = self.generate(left)?;
                let right_result = self.generate(right)?;
                let temp = self.tac.new_temp();
                self.tac.add_instruction(TACInstruction::BinaryOp {
                    dest: temp.clone(),
                    op: *operator,
                    left: left_result,
                    right: right_result,
                });
                Ok(temp)
            }
            ASTNode::Term { left, operator, right } => self.generate_expression(left, operator, right),
            ASTNode::Factor(expr) => self.generate(expr),
            ASTNode::Identifier(name) => Ok(name.clone()),
            ASTNode::Number(value) => {
                let temp = self.tac.new_temp();
                self.tac.add_instruction(TACInstruction::LoadConst {
                    dest: temp.clone(),
                    value: *value,
                });
                Ok(temp)
            }
            _ => Err("Unknown AST node".to_string()),
        }
    }

    fn generate_expression(
        &mut self,
        left: &ASTNode,
        operator: &char,
        right: &ASTNode,
    ) -> Result<String, String> {
        let left_result = self.generate(left)?;
        let right_result = self.generate(right)?;
        let temp = self.tac.new_temp();
        self.tac.add_instruction(TACInstruction::BinaryOp {
            dest: temp.clone(),
            op: *operator,
            left: left_result,
            right: right_result,
        });
        Ok(temp)
    }
}
```

### Case Study: Intermediate Code Generation for eTamil

For the eTamil language, intermediate code generation involves translating the AST into three-address code. Here’s an example eTamil program and how it is translated into TAC.

#### Example eTamil Program

```tamil
எண் a = 5 + 3 * (2 - 1);
```

#### Generating TAC

Using the lexer, parser, semantic analyzer, and intermediate code generator implemented in Rust, the above eTamil program would be translated into TAC as follows:

```rust
fn main() {
    let source_code = "எண் a = 5 + 3 * (2 - 1);";
    let mut lexer = Lexer::new(source_code.to_string());
    let tokens: Vec<Token> = lexer.tokenize();

    let mut parser = Parser::new(tokens);
    let ast = parser.parse_program();

    let mut semantic_analyzer = SemanticAnalyzer::new();
    match semantic_analyzer.analyze(&ast) {
        Ok(_) => {
            let mut code_generator = CodeGenerator::new();
            match code_generator.generate(&ast) {
                Ok(_) => println!("{:?}", code_generator.tac.instructions),
                Err(e) => println!("Code generation error: {}", e),
            }
        }
        Err(e) => println!("Semantic analysis error: {}", e),
    }
}
```

The output TAC instructions might look like this:

```plaintext
LoadConst { dest: "t1", value: 5 }
LoadConst { dest: "t2", value: 3 }
LoadConst { dest: "t3", value: 2 }
LoadConst { dest: "t4", value: 1 }
BinaryOp { dest: "t5", op: '-', left: "t3", right: "t4" }
BinaryOp { dest: "t6", op: '*', left: "t2", right: "t5" }
BinaryOp { dest: "t7", op: '+', left: "t1", right: "t6" }
Assign { dest: "a", src: "t7" }
```

### Conclusion

In this chapter, we explored the principles of intermediate code generation, focusing on the purpose of intermediate code, various intermediate representations, and the generation of three-address code (TAC). We also demonstrated how to implement an intermediate code generator in Rust for the eTamil language. The intermediate code serves as a bridge between the high-level source code and the target machine code, enabling effective optimization and simplification of the code generation process.

---
