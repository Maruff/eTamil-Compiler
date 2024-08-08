
---

## Chapter 3: Syntax Analysis

### Overview

Syntax analysis, also known as parsing, is the second phase of the compilation process. This phase checks the sequence of tokens produced by lexical analysis to ensure they form a valid syntactic structure according to the language's grammar. The primary output of syntax analysis is typically an abstract syntax tree (AST), which represents the hierarchical structure of the source code. This chapter delves into the principles of syntax analysis, parsing techniques, and provides a practical implementation using Rust for the eTamil language.

### Role of Syntax Analysis

The main functions of syntax analysis include:

- **Verification**: Ensuring that the token sequence adheres to the grammatical rules of the programming language.
- **Structure**: Building a structured representation of the source code, often in the form of an abstract syntax tree (AST).
- **Error Reporting**: Detecting and reporting syntactic errors in the source code.

### Parsing Techniques

There are two primary parsing techniques used in syntax analysis:

1. **Top-Down Parsing**: This method starts from the highest-level rule and works its way down the parse tree, expanding non-terminals until the entire input is consumed. Examples include recursive descent parsing and LL parsers.
2. **Bottom-Up Parsing**: This method starts from the input tokens and works its way up to the start symbol of the grammar, reducing sequences of tokens to non-terminals. Examples include LR parsers, SLR parsers, and LALR parsers.

### Context-Free Grammars

Context-free grammars (CFGs) are used to define the syntax of programming languages. A CFG consists of a set of production rules that describe how non-terminal symbols can be expanded into terminal symbols (tokens) and other non-terminal symbols. A grammar is considered context-free if the left-hand side of each production rule consists of a single non-terminal symbol.

#### Example CFG for a Simple Expression Language

```
E  -> E + T | E - T | T
T  -> T * F | T / F | F
F  -> ( E ) | id
```

### Parse Trees and Abstract Syntax Trees (ASTs)

- **Parse Trees**: A parse tree is a tree representation of the syntactic structure of the source code, where each node corresponds to a grammar rule.
- **Abstract Syntax Trees (ASTs)**: An AST is a simplified version of the parse tree, where unnecessary nodes (such as punctuation) are removed, and the hierarchical structure is more focused on the meaningful constructs of the language.

### Tools and Techniques

Several tools and techniques are used to implement parsers:

- **Parser Generators**: Tools like Yacc, Bison, and ANTLR automate the generation of parsers from grammar specifications.
- **Manual Implementation**: Writing custom parsers using programming languages.

### Rust Implementation

In this section, we will implement a simple recursive descent parser for eTamil using Rust. The eTamil language has unique syntactic requirements, including support for Tamil characters and specific constructs.

#### Defining the Grammar

First, we define the grammar for a subset of the eTamil language. This example focuses on basic arithmetic expressions and variable assignments.

```
Program       -> StmtList
StmtList      -> Stmt StmtList | ε
Stmt          -> Assignment | Expression
Assignment    -> Identifier '=' Expression
Expression    -> Term (( '+' | '-' ) Term)*
Term          -> Factor (( '*' | '/' ) Factor)*
Factor        -> '(' Expression ')' | Number | Identifier
```

#### Defining AST Nodes

Next, we define the AST nodes for the eTamil language.

```rust
#[derive(Debug, PartialEq)]
pub enum ASTNode {
    Program(Vec<ASTNode>),
    Assignment { identifier: String, expression: Box<ASTNode> },
    Expression { left: Box<ASTNode>, operator: char, right: Box<ASTNode> },
    Term { left: Box<ASTNode>, operator: char, right: Box<ASTNode> },
    Factor(Box<ASTNode>),
    Identifier(String),
    Number(i64),
}
```

#### Implementing the Parser

Next, we implement the parser that constructs the AST from the tokens produced by the lexer.

```rust
pub struct Parser {
    tokens: Vec<Token>,
    position: usize,
}

impl Parser {
    pub fn new(tokens: Vec<Token>) -> Self {
        Parser { tokens, position: 0 }
    }

    pub fn parse_program(&mut self) -> ASTNode {
        let mut statements = Vec::new();
        while self.position < self.tokens.len() {
            statements.push(self.parse_statement());
        }
        ASTNode::Program(statements)
    }

    fn parse_statement(&mut self) -> ASTNode {
        match self.current_token() {
            Some(Token::Identifier(_)) => self.parse_assignment(),
            Some(Token::Number(_)) => self.parse_expression(),
            _ => panic!("Unexpected token"),
        }
    }

    fn parse_assignment(&mut self) -> ASTNode {
        if let Some(Token::Identifier(identifier)) = self.consume_token() {
            self.expect_token(Token::Operator('='));
            let expression = self.parse_expression();
            ASTNode::Assignment {
                identifier,
                expression: Box::new(expression),
            }
        } else {
            panic!("Expected identifier")
        }
    }

    fn parse_expression(&mut self) -> ASTNode {
        let mut node = self.parse_term();
        while let Some(Token::Operator(op)) = self.current_token() {
            if op == '+' || op == '-' {
                self.consume_token();
                let right = self.parse_term();
                node = ASTNode::Expression {
                    left: Box::new(node),
                    operator: op,
                    right: Box::new(right),
                };
            } else {
                break;
            }
        }
        node
    }

    fn parse_term(&mut self) -> ASTNode {
        let mut node = self.parse_factor();
        while let Some(Token::Operator(op)) = self.current_token() {
            if op == '*' || op == '/' {
                self.consume_token();
                let right = self.parse_factor();
                node = ASTNode::Term {
                    left: Box::new(node),
                    operator: op,
                    right: Box::new(right),
                };
            } else {
                break;
            }
        }
        node
    }

    fn parse_factor(&mut self) -> ASTNode {
        match self.consume_token() {
            Some(Token::Number(value)) => ASTNode::Number(value),
            Some(Token::Identifier(identifier)) => ASTNode::Identifier(identifier),
            Some(Token::Punctuation('(')) => {
                let expr = self.parse_expression();
                self.expect_token(Token::Punctuation(')'));
                ASTNode::Factor(Box::new(expr))
            }
            _ => panic!("Unexpected token"),
        }
    }

    fn current_token(&self) -> Option<&Token> {
        self.tokens.get(self.position)
    }

    fn consume_token(&mut self) -> Option<Token> {
        if self.position < self.tokens.len() {
            self.position += 1;
            self.tokens.get(self.position - 1).cloned()
        } else {
            None
        }
    }

    fn expect_token(&mut self, expected: Token) {
        if let Some(token) = self.consume_token() {
            if token != expected {
                panic!("Expected token {:?}, but got {:?}", expected, token);
            }
        } else {
            panic!("Unexpected end of input");
        }
    }
}
```

### Case Study: Syntax Analysis for eTamil

For the eTamil language, syntax analysis involves parsing tokens generated by the lexer into an AST that represents the structure of the program. Here’s an example eTamil program and how it is parsed into an AST.

#### Example eTamil Program

```tamil
எண் a = 5 + 3 * (2 - 1);
```

#### Parsing to AST

Using the lexer and parser implemented in Rust, the above eTamil program would be parsed into an AST as follows:

```rust
fn main() {
    let source_code = "எண் a = 5 + 3 * (2 - 1);";
    let mut lexer = Lexer::new(source_code.to_string());
    let tokens: Vec<Token> = lexer.tokenize();

    let mut parser = Parser::new(tokens);
    let ast = parser.parse_program();

    println!("{:?}", ast);
}
```

The output AST might look like this:

```
Program(
    [
        Assignment {
            identifier: "a",
            expression: Expression {
                left: Box::new(Number(5)),
                operator: '+',
                right: Box::new(Term {
                    left: Box::new(Number(3)),
                    operator: '*',
                    right: Box::new(Factor(
                        Box::new(Expression {
                            left: Box::new(Number(2)),
                            operator: '-',
                            right: Box::new(Number(1)),
                        })
                    )),
                }),
            },
        }
    ]
)
```

### Conclusion

In this chapter, we explored the principles of syntax analysis, focusing on parsing techniques, context-free grammars, and the construction of parse trees and abstract syntax trees (ASTs). We also demonstrated how to implement a recursive descent parser in Rust for the eTamil language. The parser builds the AST, which serves as the structured representation of the source code, ready for semantic analysis in the next phase.

---
