
---

## Chapter 4: Semantic Analysis

### Overview

Semantic analysis is the third phase of the compilation process. This phase checks the abstract syntax tree (AST) produced by syntax analysis to ensure that it adheres to the semantic rules of the programming language. These rules include type checking, variable declaration, and scope resolution. This chapter explores the principles of semantic analysis, error detection, and provides a practical implementation using Rust for the eTamil language.

### Role of Semantic Analysis

The main functions of semantic analysis include:

- **Type Checking**: Ensuring that operations are applied to compatible types and that expressions have the correct types.
- **Scope Resolution**: Verifying that variables and functions are declared before they are used and resolving their scopes.
- **Error Detection**: Identifying and reporting semantic errors such as type mismatches, undeclared variables, and invalid operations.

### Semantic Rules and Type Checking

Semantic rules define the correct usage of language constructs. Type checking is a crucial part of semantic analysis, ensuring that operations are applied to compatible types and that expressions evaluate to the correct types.

#### Example Semantic Rules

- **Type Compatibility**: Arithmetic operations should be performed on numeric types, and logical operations on boolean types.
- **Variable Declaration**: Variables must be declared before they are used.
- **Function Calls**: The number and types of arguments in a function call must match the function's definition.

### Symbol Tables

Symbol tables are essential data structures used in semantic analysis to store information about variables, functions, and their attributes (e.g., types, scope, memory location).

#### Example Symbol Table Entry

```rust
struct Symbol {
    name: String,
    symbol_type: SymbolType,
    scope_level: usize,
}

enum SymbolType {
    Variable(DataType),
    Function(DataType, Vec<DataType>), // Return type, Parameter types
}

enum DataType {
    Int,
    Float,
    Bool,
    // Add other types as needed
}
```

### Error Detection and Reporting

Semantic analysis involves detecting and reporting various types of errors, such as:

- **Type Errors**: Applying an operation to incompatible types.
- **Undeclared Variables**: Using a variable that has not been declared.
- **Scope Errors**: Referencing variables or functions outside their declared scope.
- **Function Errors**: Mismatched arguments in function calls.

### Rust Implementation

In this section, we will implement a simple semantic analyzer for eTamil using Rust. This analyzer will perform type checking and scope resolution.

#### Defining the Symbol Table

First, we define the symbol table and its entries.

```rust
use std::collections::HashMap;

#[derive(Debug, PartialEq, Clone)]
pub enum DataType {
    Int,
    Float,
    Bool,
}

#[derive(Debug)]
pub struct Symbol {
    name: String,
    symbol_type: SymbolType,
    scope_level: usize,
}

#[derive(Debug)]
pub enum SymbolType {
    Variable(DataType),
    Function(DataType, Vec<DataType>), // Return type, Parameter types
}

pub struct SymbolTable {
    symbols: HashMap<String, Symbol>,
    scope_level: usize,
}

impl SymbolTable {
    pub fn new() -> Self {
        SymbolTable {
            symbols: HashMap::new(),
            scope_level: 0,
        }
    }

    pub fn enter_scope(&mut self) {
        self.scope_level += 1;
    }

    pub fn exit_scope(&mut self) {
        self.scope_level -= 1;
        self.symbols.retain(|_, symbol| symbol.scope_level <= self.scope_level);
    }

    pub fn add_symbol(&mut self, name: String, symbol_type: SymbolType) {
        let symbol = Symbol {
            name: name.clone(),
            symbol_type,
            scope_level: self.scope_level,
        };
        self.symbols.insert(name, symbol);
    }

    pub fn get_symbol(&self, name: &str) -> Option<&Symbol> {
        self.symbols.get(name)
    }
}
```

#### Implementing the Semantic Analyzer

Next, we implement the semantic analyzer that checks the AST for semantic correctness.

```rust
pub struct SemanticAnalyzer {
    symbol_table: SymbolTable,
}

impl SemanticAnalyzer {
    pub fn new() -> Self {
        SemanticAnalyzer {
            symbol_table: SymbolTable::new(),
        }
    }

    pub fn analyze(&mut self, node: &ASTNode) -> Result<DataType, String> {
        match node {
            ASTNode::Program(statements) => {
                for statement in statements {
                    self.analyze(statement)?;
                }
                Ok(DataType::Int)
            }
            ASTNode::Assignment { identifier, expression } => {
                let expr_type = self.analyze(expression)?;
                self.symbol_table.add_symbol(identifier.clone(), SymbolType::Variable(expr_type.clone()));
                Ok(expr_type)
            }
            ASTNode::Expression { left, operator, right } => {
                let left_type = self.analyze(left)?;
                let right_type = self.analyze(right)?;

                if left_type != right_type {
                    return Err(format!("Type mismatch: {:?} {} {:?}", left_type, operator, right_type));
                }

                match operator {
                    '+' | '-' | '*' | '/' => {
                        if left_type != DataType::Int {
                            return Err(format!("Invalid operand types for {}: {:?}", operator, left_type));
                        }
                        Ok(DataType::Int)
                    }
                    _ => Err(format!("Unknown operator: {}", operator)),
                }
            }
            ASTNode::Term { left, operator, right } => self.analyze_expression(left, operator, right),
            ASTNode::Factor(expr) => self.analyze(expr),
            ASTNode::Identifier(name) => {
                if let Some(symbol) = self.symbol_table.get_symbol(name) {
                    match &symbol.symbol_type {
                        SymbolType::Variable(data_type) => Ok(data_type.clone()),
                        _ => Err(format!("Identifier {} is not a variable", name)),
                    }
                } else {
                    Err(format!("Undeclared variable: {}", name))
                }
            }
            ASTNode::Number(_) => Ok(DataType::Int),
            _ => Err("Unknown AST node".to_string()),
        }
    }

    fn analyze_expression(
        &mut self,
        left: &ASTNode,
        operator: &char,
        right: &ASTNode,
    ) -> Result<DataType, String> {
        let left_type = self.analyze(left)?;
        let right_type = self.analyze(right)?;

        if left_type != right_type {
            return Err(format!(
                "Type mismatch: {:?} {} {:?}",
                left_type, operator, right_type
            ));
        }

        match operator {
            '+' | '-' | '*' | '/' => {
                if left_type != DataType::Int {
                    return Err(format!(
                        "Invalid operand types for {}: {:?}",
                        operator, left_type
                    ));
                }
                Ok(DataType::Int)
            }
            _ => Err(format!("Unknown operator: {}", operator)),
        }
    }
}
```

### Case Study: Semantic Analysis for eTamil

For the eTamil language, semantic analysis involves type checking and scope resolution based on the semantic rules defined for the language. Here’s an example eTamil program and how it is analyzed semantically.

#### Example eTamil Program

```tamil
எண் a = 5 + 3 * (2 - 1);
```

#### Analyzing the AST

Using the lexer, parser, and semantic analyzer implemented in Rust, the above eTamil program would be analyzed as follows:

```rust
fn main() {
    let source_code = "எண் a = 5 + 3 * (2 - 1);";
    let mut lexer = Lexer::new(source_code.to_string());
    let tokens: Vec<Token> = lexer.tokenize();

    let mut parser = Parser::new(tokens);
    let ast = parser.parse_program();

    let mut semantic_analyzer = SemanticAnalyzer::new();
    match semantic_analyzer.analyze(&ast) {
        Ok(_) => println!("Semantic analysis successful"),
        Err(e) => println!("Semantic analysis error: {}", e),
    }
}
```

The semantic analyzer checks for type correctness, variable declarations, and scope resolution, ensuring that the program adheres to the semantic rules of eTamil.

### Conclusion

In this chapter, we explored the principles of semantic analysis, focusing on type checking, scope resolution, and error detection. We also demonstrated how to implement a semantic analyzer in Rust for the eTamil language. The semantic analyzer ensures that the AST is semantically correct, preparing it for the subsequent phases of intermediate code generation and optimization.

---
